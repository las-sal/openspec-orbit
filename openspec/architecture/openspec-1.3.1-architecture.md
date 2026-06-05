# OpenSpec 1.3.1 — Architecture Deep-Dive & Walkthrough

> Grounding document for forking openspec@1.3.1 and folding orbit into it ("orbify").
> Written from a read of the actual source at tag `v1.3.1` (cloned from
> `github.com/Fission-AI/OpenSpec`; the Homebrew/npm install ships only compiled `dist/`).
> Every file path and construct below was verified against source.

---

## 0. TL;DR mental model

OpenSpec is **two cooperating systems sharing one repo**:

1. **A spec-lifecycle engine** — a CLI that manages a `openspec/` directory of *specs* (source of
   truth) and *changes* (proposed modifications, as deltas), and folds changes back into specs on
   archive. This is plain data-on-disk + parsers + a merge engine. *Orbit pins this and mostly
   leaves it alone.*

2. **A prompt-authoring/distribution system** — a TypeScript-defined library of **workflow
   instruction prompts** (explore, propose, apply, archive, …) that get *generated* into 26 different
   AI tools' on-disk formats (`.claude/`, `.cursor/`, `.codex/`, …) as **skills** and **slash
   commands**. *This is exactly the layer orbit overlays today, and the layer the fork must take
   ownership of at the source level.*

The bridge between them is the **schema/artifact-graph engine**: a YAML-defined DAG of artifacts
(proposal → specs/design → tasks → apply) that the CLI turns into *enriched instructions* an agent
consumes. The prompts in system (2) tell the agent to call the CLI in system (3) to get those
instructions. So the three layers form a loop:

```
   prompts (skills/commands)  ──tell agent to run──▶  openspec CLI (instructions/status/validate)
            ▲                                                  │
            │                                                  │ reads/writes
            └──────────────  agent edits files  ◀─────────────┘
                                   in openspec/specs , openspec/changes
```

---

## 1. Source layout

~22,400 LOC TypeScript source / ~23,100 LOC tests. ESM, Node ≥20.19, built with **plain `tsc`**
(`build.js` cleans `dist/` then compiles — no bundler). Deps are deliberately small:
`commander` (CLI), `zod` (validation), `yaml`, `fast-glob`, `@inquirer/*` (prompts),
`ora`/`chalk` (terminal UI), `posthog-node` (telemetry).

```
src/
├── cli/index.ts                      ← single Commander entrypoint (~510 LOC); wires every command
├── commands/                         ← command implementations (the "verbs")
│   ├── change.ts spec.ts             ← deprecated noun-first commands
│   ├── validate.ts show.ts           ← top-level verb-first commands
│   ├── config.ts schema.ts           ← config + schema management (largest: schema.ts ~30KB)
│   ├── completion.ts feedback.ts
│   └── workflow/                     ← the schema-driven workflow commands
│       ├── instructions.ts           ← emits enriched instructions for an artifact / for apply
│       ├── status.ts                 ← artifact completion status for a change
│       ├── new-change.ts schemas.ts templates.ts shared.ts
├── core/
│   ├── init.ts (779)  update.ts (702)  ← setup/refresh orchestrators (profiles, delivery, versioning)
│   ├── archive.ts (12.6KB)            ← THE merge engine: folds change deltas into main specs
│   ├── list.ts view.ts               ← read/dashboard
│   ├── config.ts config-schema.ts    ← AI_TOOLS table; global config zod schema
│   ├── global-config.ts profiles.ts project-config.ts  ← config resolution
│   ├── migration.ts legacy-cleanup.ts profile-sync-drift.ts  ← upgrade/cleanup machinery
│   ├── artifact-graph/               ← ★ the workflow "brain" (schema DAG runtime)
│   │   ├── types.ts                  ←   zod schemas: Artifact, SchemaYaml, ApplyPhase, ChangeMetadata
│   │   ├── schema.ts resolver.ts     ←   load/parse/resolve schema YAML (built-in + project-local)
│   │   ├── graph.ts                  ←   ArtifactGraph: Kahn topo-sort, getNextArtifacts, getBlocked
│   │   ├── state.ts outputs.ts       ←   completion detection by file existence (+ glob)
│   │   └── instruction-loader.ts     ←   assembles enriched instructions from schema + change state
│   ├── templates/                    ← ★ the prompt SOURCE OF TRUTH
│   │   ├── workflows/<wf>.ts (×12)   ←   one module per workflow; exports skill + command templates
│   │   ├── skill-templates.ts        ←   re-export facade
│   │   └── types.ts                  ←   SkillTemplate / CommandTemplate interfaces
│   ├── command-generation/           ← ★ prompt → per-tool file formatter
│   │   ├── types.ts generator.ts registry.ts
│   │   └── adapters/<tool>.ts (×26)  ←   claude.ts, cursor.ts, codex.ts, … (path + frontmatter rules)
│   ├── shared/skill-generation.ts    ← ★ registration tables (skills+commands ↔ workflow IDs)
│   ├── parsers/                      ← markdown/spec/change parsers
│   │   ├── requirement-blocks.ts     ←   the delta format parser (### Requirement / #### Scenario)
│   │   ├── spec-structure.ts change-parser.ts markdown-parser.ts
│   ├── validation/                   ← validator.ts + constants + types (zod-backed)
│   ├── converters/json-converter.ts  ← markdown ↔ JSON for --json output
│   ├── completions/                  ← shell completion generators/installers (bash/zsh/fish)
│   └── schemas/                      ← *.schema.ts zod schemas for change/spec JSON shapes
├── telemetry/                        ← posthog wrapper + first-run notice + trackCommand
├── prompts/searchable-multi-select.ts← custom inquirer tool-picker
├── ui/ utils/                        ← welcome screen; fs/interactive/match/task-progress helpers
└── prompts/

schemas/spec-driven/                  ← the bundled default workflow (shipped as a data file, not code)
├── schema.yaml                       ←   the artifact DAG definition
└── templates/{proposal,spec,design,tasks}.md   ← the artifact body templates
docs/                                 ← user docs (concepts, workflows, opsx, customization, …)
```

---

## 2. The data model: specs vs changes (system 1)

This is the substrate everything else manipulates. From `docs/concepts.md`, the philosophy is
*fluid / iterative / easy / brownfield-first*, and the structure is two worlds under `openspec/`:

```
openspec/
├── specs/<capability>/spec.md      ← SOURCE OF TRUTH: how the system behaves now
├── changes/<name>/                 ← one in-flight change = one folder
│   ├── proposal.md  design.md  tasks.md
│   ├── specs/<capability>/spec.md  ← DELTA specs (ADDED/MODIFIED/REMOVED/RENAMED requirements)
│   └── .openspec.yaml              ← per-change metadata (which schema; created date)
├── changes/archive/<YYYY-MM-DD>-<name>/   ← completed changes, dated
└── config.yaml                     ← project config (currently mostly: schema selection)
```

### Spec & delta file format

Specs are **structured markdown** with a strict grammar the parsers depend on
(`src/core/parsers/requirement-blocks.ts`, `spec-structure.ts`):

```
## Requirements
### Requirement: <name>          ← exactly "### Requirement:"
The system SHALL …               ← normative prose (SHALL/MUST)
#### Scenario: <name>            ← EXACTLY 4 hashtags; 3 fails silently
- **WHEN** …
- **THEN** …
```

A change's delta spec (`changes/<name>/specs/<cap>/spec.md`) wraps requirements in delta operations:
`## ADDED Requirements`, `## MODIFIED Requirements` (must contain the *full* updated block),
`## REMOVED Requirements` (needs **Reason** + **Migration**), `## RENAMED Requirements` (FROM:/TO:).

> **Fork-relevant gotcha:** the "4 hashtags or it fails silently" and "MODIFIED must carry the whole
> block" rules are load-bearing and parser-enforced, not just convention. Orbit's review/audit layers
> exist partly to catch exactly these. Any fork edits to the parser ripple into every spec.

---

## 3. The schema / artifact-graph engine (system bridging 1↔3)

This is the conceptual heart and the least obvious part. A **schema** is a YAML file describing a DAG
of **artifacts**. The only one shipped is `schemas/spec-driven/schema.yaml`:

```yaml
name: spec-driven
version: 1
artifacts:
  - id: proposal   generates: proposal.md      template: proposal.md   requires: []
  - id: specs      generates: "specs/**/*.md"  template: spec.md       requires: [proposal]
  - id: design     generates: design.md        template: design.md     requires: [proposal]
  - id: tasks      generates: tasks.md         template: tasks.md      requires: [specs, design]
apply:
  requires: [tasks]
  tracks: tasks.md     # checkbox progress source
  instruction: |  Read context files, work through pending tasks…
```

Each artifact carries an `instruction` (long-form guidance the agent gets) and a `template`
(the skeleton file to fill in, under `schemas/spec-driven/templates/`).

### The runtime pieces (`src/core/artifact-graph/`)

- **`types.ts`** — zod schemas (`ArtifactSchema`, `SchemaYamlSchema`, `ApplyPhaseSchema`,
  `ChangeMetadataSchema`). Schema validity is enforced at load.
- **`schema.ts` / `resolver.ts`** — load + validate YAML. `resolver.ts` resolves a schema *name* to a
  file, searching **project-local schemas first, then the package's built-in `schemas/` dir** (so
  projects can define custom workflows). Also handles a global data dir.
- **`graph.ts`** — `ArtifactGraph`. Pure graph logic:
  - `getBuildOrder()` — Kahn's algorithm topological sort (deterministic: roots & newly-ready sets
    are sorted).
  - `getNextArtifacts(completed)` — artifacts whose `requires` are all satisfied.
  - `getBlocked(completed)` — artifacts with unmet deps → which deps.
  - `isComplete(completed)`.
- **`state.ts` + `outputs.ts`** — completion is detected purely by **file existence**:
  `resolveArtifactOutputs()` resolves an artifact's `generates` (literal path or glob via
  `fast-glob`) to concrete files; if any exist, the artifact is "done". No database, no state file.
- **`instruction-loader.ts`** — `loadChangeContext()` (auto-detects schema from the change's
  `.openspec.yaml`) and `generateInstructions()` assemble the enriched instruction payload.

> **Why this matters for the fork:** the workflow's "what comes next" intelligence is *data*
> (schema.yaml) interpreted by ~1,150 LOC of generic graph code — **not** hardcoded. Orbit's added
> workflows (review, address-reviews, audit-drift) are *not* artifacts in this graph; they're
> additional *prompts*. If you ever want orbit's review to be a first-class graph phase (a gate
> between tasks and archive), this engine is where it'd live — but today it's orthogonal.

---

## 4. The instructions protocol — how the CLI talks to agents (system 3)

This is the contract between the prompts and the engine. When a skill says "run
`openspec instructions <artifact> --change <name>`", here's what comes back
(`src/commands/workflow/instructions.ts`):

The CLI prints a structured, **XML-tagged prompt** for the agent:

```
<artifact id="specs" change="add-foo" schema="spec-driven">
  <warning> …unmet dependencies… </warning>          (only if blocked)
  <task> Create the specs artifact for change "add-foo". <description> </task>
  <project_context> …background; do NOT echo… </project_context>
  <rules> …constraints; do NOT echo… </rules>
  <dependencies> read proposal.md (status=done) … </dependencies>
  <output> Write to: openspec/changes/add-foo/specs/**/*.md </output>
  <instruction> …the schema artifact.instruction… </instruction>
  <template> …the schema template body to fill in… </template>
  <success_criteria/> <unlocks> tasks </unlocks>
</artifact>
```

`openspec instructions apply` is special-cased (`applyInstructionsCommand`): it reads the schema's
`apply` block, checks required artifacts exist, parses `tasks.md` checkboxes
(`/^[-*]\s*\[([ xX])\]\s*(.+)$/`), computes progress, and returns a `state` of
`blocked | ready | all_done` with tailored guidance.

`openspec status` (`workflow/status.ts`) gives the artifact completion grid; both support `--json`
for programmatic/agent use.

> **Design insight:** the CLI is the *deterministic* half (graph math, file detection, validation),
> the agent is the *generative* half. The prompts orchestrate the handoff. This is the seam orbit
> leverages — orbit's value is almost entirely in the *prompt* half, which is why it can be a pure
> `.claude/` overlay today.

---

## 5. ★ The prompt template → generation → adapter pipeline (system 2)

**This is the single most important section for the fork.** Orbit's hand-maintained
`.claude/skills/openspec-*/SKILL.md` and `.claude/commands/opsx/*.md` are *generated output* of this
pipeline. To "orbify the fork" is to move orbit's content from the output position (overlay) up to
the source position (these TS modules).

### 5.1 The source: one module per workflow

`src/core/templates/workflows/<wf>.ts` — each exports **two** functions:

```ts
export function getExploreSkillTemplate(): SkillTemplate { name, description, instructions }
export function getOpsxExploreCommandTemplate(): CommandTemplate { name, description, category, tags, content }
```

`SkillTemplate` and `CommandTemplate` (`templates/types.ts`):

```ts
interface SkillTemplate  { name; description; instructions; license?; compatibility?; metadata? }
interface CommandTemplate{ name; description; category; tags; content }
```

The 12 modules: `explore, new-change, continue-change, apply-change, ff-change, sync-specs,
archive-change, bulk-archive-change, verify-change, onboard, propose` (+ `feedback`).
`skill-templates.ts` is just a re-export facade.

### 5.2 The registration tables

`src/core/shared/skill-generation.ts` is the **central registry** — two arrays map workflow IDs to
their templates and on-disk skill-dir names:

```ts
getSkillTemplates()  → [{ template, dirName:'openspec-explore', workflowId:'explore' }, … ]  // 11
getCommandTemplates()→ [{ template, id:'explore' }, … ]                                       // 11
getCommandContents() → maps CommandTemplate → CommandContent (adds body=content)
generateSkillContent(template, version, transform?) → SKILL.md with YAML frontmatter
```

> **The entire registration surface for a new workflow is: add a `templates/workflows/<x>.ts`,
> export it from `skill-templates.ts`, and add one line to each of the two arrays here.** That's it.
> This is where orbit's 4 net-new workflows (review, review-external, address-reviews, audit-drift)
> would slot in.

### 5.3 The formatter: tool adapters

`src/core/command-generation/`:

- `types.ts` — `CommandContent` (tool-agnostic: id/name/description/category/tags/body) and
  `ToolCommandAdapter` (`getFilePath(id)` + `formatFile(content)`).
- `registry.ts` — `CommandAdapterRegistry` with a static block registering **26 adapters**
  (claude, cursor, codex, gemini, github-copilot, windsurf, cline, roocode, kilocode, …).
- `adapters/claude.ts` — Claude is just one adapter:
  ```ts
  getFilePath(id) → ".claude/commands/opsx/<id>.md"
  formatFile()    → "---\nname:…\ndescription:…\ncategory:…\ntags:[…]\n---\n\n" + body
  ```
- `generator.ts` — trivial: `generateCommand(content, adapter) = { path, fileContent }`.

**Skills vs commands** are two different outputs from the same templates:
- **Skills** → `<toolDir>/skills/<dirName>/SKILL.md` via `generateSkillContent()` (richer
  frontmatter: license, compatibility, metadata.author/version, **`generatedBy: <openspec version>`**).
  Skill paths are uniform across tools (`.claude/skills/…`, `.cursor/skills/…`).
- **Commands** → per-adapter path & frontmatter (e.g. `.claude/commands/opsx/<id>.md`).

A `transformInstructions` hook exists for tools where filename == command name (opencode, pi) to
rewrite `/opsx:x` references to hyphen form — relevant if you keep multi-tool support.

```
templates/workflows/explore.ts
   getExploreSkillTemplate() ─┐         getOpsxExploreCommandTemplate() ─┐
                              ▼                                          ▼
   shared/skill-generation.ts: getSkillTemplates() / getCommandContents()
                              │                                          │
        generateSkillContent()│                generateCommands(contents, adapter)
                              ▼                                          ▼
   <tool>/skills/openspec-explore/SKILL.md      <tool>/commands/opsx/explore.md
                              ▲
              written to disk by init.ts / update.ts
```

---

## 6. Setup & refresh lifecycle — `init` / `update` (`src/core/init.ts`, `update.ts`)

These orchestrate the pipeline in §5 onto disk. Both are ~700–780 LOC and share a lot of shape.

### `openspec init [path] --tools <…> --force --profile <…>`

1. `validate()` — detect **extend mode** (does `openspec/` already exist?) + write perms.
2. **Legacy cleanup** (`legacy-cleanup.ts`) — detect & remove old-format artifacts (interactive
   confirm, or auto under `--force`/non-interactive).
3. **Tool detection** (`getAvailableTools`) — scans for tool dirs (`.claude/`, `.cursor/`, …) to
   pre-select.
4. **Migration** (`migration.ts`) — bring existing projects onto the profile system.
5. **Tool selection** — `--tools all|none|<csv>` non-interactive, or a searchable multi-select.
6. Create `openspec/{specs,changes,changes/archive}`.
7. **`generateSkillsAndCommands()`** — the core: read **global config** for `profile` + `delivery`,
   compute the **workflow set**, then for each tool write skills and/or commands (§5).
8. Write `openspec/config.yaml` (schema selection) if missing.
9. Success report + "restart your IDE".

### Profiles & delivery (the knobs that decide *what* gets generated)

From `global-config.ts` / `profiles.ts` / `config-schema.ts` (global config lives in a user-level
data dir, zod-validated, `passthrough()` for forward-compat):

- **profile**: `core` (a curated subset of workflows) vs `custom` (explicit `workflows: [...]` list).
  `getProfileWorkflows(profile, workflows)` resolves the active set. `CORE_WORKFLOWS` / `ALL_WORKFLOWS`
  are the constants. *(This is why a default `init` yields ~10 skills+commands, not all 11–12.)*
- **delivery**: `both` (default) | `skills` | `commands`. Controls whether skills, commands, or both
  are emitted; the off mode triggers removal of the other.

### `openspec update` — smart refresh

Each generated skill embeds `generatedBy: <version>`. `update`:
- migrates + handles legacy, reads profile/delivery,
- per tool computes **version drift** (`getToolVersionStatus`) and **profile/config drift**
  (`profile-sync-drift.ts`: tools needing the workflow set re-synced),
- regenerates only tools that need it (or all with `--force`),
- **prunes** skills/commands for workflows no longer in the profile, and prunes the other delivery
  mode's files,
- warns about *extra* installed workflows and *newly detected* tool dirs.

> **Fork implication:** `generatedBy` version stamping + drift detection is the machinery that makes
> "overlay gets overwritten by upstream" inevitable today. Once orbit's content *is* the source,
> this machinery becomes your friend: `openspec update` legitimately refreshes orbit content and the
> version stamp becomes orbit's version. You inherit a working update/migration system for free.

---

## 7. The archive / merge engine (`src/core/archive.ts`) — the riskiest inherited code

`openspec archive [name] -y --skip-specs --no-validate`. `ArchiveCommand.execute()`:

1. Resolve the change (interactive picker if omitted).
2. (Unless `--no-validate`) validate the change.
3. For each delta spec in the change, **fold it into `openspec/specs/<cap>/spec.md`**: parse the
   main spec into requirement blocks (`parsers/requirement-blocks.ts`), apply ADDED/MODIFIED/
   REMOVED/RENAMED operations, write back. (`--skip-specs` bypasses this for tooling/doc-only changes.)
4. **Move** the change dir to `changes/archive/<YYYY-MM-DD>-<name>/` (date prefix added here — this is
   why orbit's `.orbit-runs/` audit trail picks up its date on archive).

> ⚠️ **Known correctness bug, documented in the repo's own `openspec-parallel-merge-plan.md`:** the
> merge is **replace-only requirement-block substitution** with no base-version fingerprint and no
> scenario-level granularity. Two parallel changes that touch the same requirement → the second
> archive silently overwrites the first's scenarios. No warning, no conflict markers. The plan doc
> proposes base fingerprints + scenario-level deltas + a conflict-resolution UX.
>
> **This is the one piece of upstream you should treat as a liability you now own.** It's
> independent of orbit, but a fork is the natural place to actually fix it (orbit's audit-drift /
> review layers already exist to *detect* drift; fixing the merge engine would address the root cause).

---

## 8. Supporting subsystems (skim)

- **validation/** (`validator.ts` + constants/types) — zod-backed validation of changes and specs;
  `--strict` mode; drives `openspec validate` and pre-archive checks. `core/schemas/*.schema.ts`
  define the JSON shapes; `converters/json-converter.ts` does markdown↔JSON for `--json`.
- **parsers/** — `change-parser.ts`, `spec-structure.ts`, `markdown-parser.ts`,
  `requirement-blocks.ts`. The grammar police; any spec-format change touches these.
- **config** — global (user data dir, `global-config.ts`) vs project (`openspec/config.yaml`,
  `project-config.ts` — schema selection + optional `validateConfigRules`). `commands/config.ts` and
  `commands/schema.ts` are the management CLIs (schema.ts is the biggest single command, ~30KB:
  add/list/validate project-local schemas).
- **completions/** — bash/zsh/fish completion generation + install/uninstall; `__complete` hidden
  command feeds machine-readable data.
- **telemetry/** — `posthog-node`, first-run notice (`maybeShowTelemetryNotice`), `trackCommand` in a
  Commander `preAction` hook, flushed in `postAction`. **A fork almost certainly rips this out or
  makes it opt-in.**
- **migration.ts / legacy-cleanup.ts / profile-sync-drift.ts** — the upgrade/cleanup/drift machinery
  init & update lean on. ~55KB combined; mostly defensive.
- **ui/welcome-screen.ts, prompts/searchable-multi-select.ts** — interactive niceties.

---

## 9. End-to-end walkthrough: one change, all the way through

Tracing which code runs (★ = orbit overlays/extends this step today):

```
1. /opsx:propose "add data export"      ← skill body from templates/workflows/propose.ts ★(orbit-modified)
   └─ agent: `openspec new change add-data-export`  → workflow/new-change.ts
              writes changes/add-data-export/.openspec.yaml (schema: spec-driven)
   └─ agent: `openspec instructions proposal --change add-data-export`
              → instruction-loader assembles <artifact> prompt from schema.yaml + templates/proposal.md
   └─ agent writes proposal.md

2. /opsx:continue                         ← templates/workflows/continue-change.ts
   └─ `openspec status`  → graph.getNextArtifacts(detectCompleted(...))  → "specs, design ready"
   └─ `openspec instructions specs …`  → agent writes delta specs
   └─ repeat for design, tasks

   (orbit-only) /opsx:review               ← orbit skill; NOT an upstream artifact. Editorial passes.
                /opsx:review-external       ← packages a prompt for a second AI
                /opsx:address-reviews       ← resolves @review: markers; writes .orbit-runs/

3. /opsx:apply                            ← templates/workflows/apply-change.ts ★
   └─ `openspec instructions apply`  → parses tasks.md checkboxes, state machine, guidance
   └─ agent implements, checks boxes

4. /opsx:verify                           ← templates/workflows/verify-change.ts
   (orbit-only) /opsx:audit-drift          ← orbit project-wide drift scan

5. /opsx:archive                          ← templates/workflows/archive-change.ts ★(orbit-modified)
   └─ `openspec archive add-data-export`  → archive.ts: fold deltas into specs/, move to archive/<date>-…
```

The CLI never generates prose; the agent never computes graph state. Orbit inserts review/audit
*prompts* between the upstream phases and modifies a few upstream prompts — all in layer (2), none in
the engine.

---

## 10. How orbit maps onto this architecture

| Orbit surface (today: `.claude/` overlay) | Upstream home (after fork) | Notes |
|---|---|---|
| 11 modified `SKILL.md` / `opsx/*.md` (explore, propose, apply, archive, onboard, …) | edit the 11 `templates/workflows/*.ts` | direct source edits replace overlay |
| `review`, `review-external`, `address-reviews`, `audit-drift` (4 net-new) | new `templates/workflows/*.ts` + 2 lines each in `shared/skill-generation.ts` + facade export | becomes first-class, multi-tool, profile-aware |
| `orbit-conventions` (read-before-ref / change-completeness / pushback) | shared instruction snippet injected into every template, OR an always-installed skill | a cross-cutting decision |
| `orbit-run-summary-emit` (`.orbit-runs/` audit trail) | convention referenced by review/archive prompts | data-on-disk; no engine change needed |
| `openspec/lenses/*` (perspectives, critical-paths) | orbit-specific persistence; lives in the consuming repo's `openspec/`, not the tool | no upstream code change |
| version peg @1.3.1 | becomes the fork's own version line | `generatedBy` stamp becomes orbit's |

**Net:** orbit = 11 modified upstream workflows + 4 new workflows + 2 cross-cutting disciplines.
All of it lives in layer (2) (templates + registration) plus on-disk conventions. The engine
(layers 1 & 3) is untouched by orbit's *features* — though the fork is the right place to fix the
archive merge bug (§7) if you choose to.

---

## 11. Fork & orbify: what you're taking on

**Cleanly forkable** because the architecture cooperates:
- Prompt content is already factored into per-workflow modules with a tiny central registry.
- Adding workflows is a 3-touch operation (module + facade + two array lines).
- `init`/`update`/migration/version-stamping/profiles all key off those same registries, so orbit
  workflows automatically get distribution, refresh, drift-detection, and profile management.
- 26-tool adapter matrix means orbit reaches every tool for free (if you keep it).
- ~23k LOC of tests give you a safety net for engine edits.

**Decisions to make before/while forking:**

1. **Tool breadth — all 26 adapters, or Claude-first?** Keeping all is nearly free (templates flow
   to every adapter) but widens the test/support surface and ties orbit prompts to a lowest-common-
   denominator phrasing. Claude-only simplifies but discards upstream's main breadth advantage.
2. **Telemetry** — remove `posthog-node` / `telemetry/` or make it opt-in. (Easy; isolated to the
   Commander hooks + one module.)
3. **Cross-cutting conventions delivery** — inject `orbit-conventions` into every workflow template
   (DRY but duplicated at generation time, matching today's "intentional duplication for reliability"
   stance) vs. ship as a standalone always-on skill.
4. **Are orbit's review/audit phases prompts or graph artifacts?** Today they're orthogonal prompts.
   You *could* promote them into `schema.yaml` as real phases (e.g. a `review` gate requiring `tasks`,
   an `audit` gate before `apply` to archive) so `status`/`instructions`/`getBlocked` understand them.
   Bigger change; engine-level; optional.
5. **Upstream rebase strategy** — once you edit templates in place, pulling future upstream fixes
   (esp. the archive merge fix) is a manual merge. Consistent with orbit's existing "no auto-ingest,
   deliberate upgrades only" stance; the fork hardens it into repo structure.
6. **Archive merge bug (§7)** — fix now (engine work, but root-cause) or keep detecting via
   audit-drift/review (status quo)?

**Lowest-risk first move:** fork, rip telemetry, get a green build/test run, then migrate ONE
workflow (e.g. `explore`) from overlay → template edit and confirm `openspec init --tools claude`
regenerates orbit's exact content. That proves the whole loop before doing the other 10 + 4 new ones.

---

## Appendix: quick command ↔ code map

| CLI command | Entry | Core logic |
|---|---|---|
| `init` | `cli/index.ts` → `core/init.ts` | InitCommand.execute |
| `update` | `core/update.ts` | UpdateCommand.execute |
| `new change` | `commands/workflow/new-change.ts` | writes `.openspec.yaml` |
| `status` | `commands/workflow/status.ts` | ArtifactGraph + detectCompleted |
| `instructions [artifact|apply]` | `commands/workflow/instructions.ts` | instruction-loader |
| `schemas` / `templates` | `workflow/schemas.ts`, `templates.ts` | resolver |
| `validate` | `commands/validate.ts` | `core/validation/validator.ts` |
| `show` / `list` / `view` | `commands/show.ts`, `core/list.ts`, `core/view.ts` | converters/parsers |
| `archive` | `core/archive.ts` | requirement-block merge + move |
| `config` / `schema` | `commands/config.ts`, `commands/schema.ts` | global/project config, custom schemas |
| `completion` | `commands/completion.ts` | `core/completions/*` |
| `feedback` | `commands/feedback.ts` | posthog |

| Term | Meaning |
|---|---|
| **artifact** | a node in the schema DAG (proposal/specs/design/tasks); detected done by file existence |
| **schema** | YAML defining the artifact DAG + apply phase (`schemas/<name>/schema.yaml`) |
| **delta spec** | a change's spec file using ADDED/MODIFIED/REMOVED/RENAMED operations |
| **skill** | `<tool>/skills/<dir>/SKILL.md` — auto-loaded capability prompt |
| **command** | `<tool>/commands/opsx/<id>.md` — explicit `/opsx:<id>` slash command |
| **adapter** | per-tool formatter mapping a CommandContent to that tool's path + frontmatter |
| **profile** | curated workflow subset (`core`) vs explicit list (`custom`) |
| **delivery** | `both` / `skills` / `commands` — which artifact types `init`/`update` emit |
| **generatedBy** | openspec version stamped into each SKILL.md; drives `update` drift detection |
```
