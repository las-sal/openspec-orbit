# openspec-orbit

An opinionated `.claude/` overlay for [@fission-ai/openspec](https://github.com/Fission-AI/OpenSpec) that bakes a tested review-and-capture workflow into spec-driven development.

> **Status**: v1 design exploration. Command/skill sketches complete; implementation pending. Repo currently contains design records under `openspec/changes/bootstrap-openspec-orbit/`.

---

## Table of contents

- [openspec-orbit](#openspec-orbit)
  - [Table of contents](#table-of-contents)
  - [What this is](#what-this-is)
  - [The orbit workflow](#the-orbit-workflow)
  - [Guiding principles](#guiding-principles)
  - [Command reference](#command-reference)
    - [`/opsx:explore [<name>]`](#opsxexplore-name)
    - [`/opsx:propose <name>`](#opsxpropose-name)
    - [`/opsx:review <name> [--as proposal|system]`](#opsxreview-name---as-proposalsystem)
    - [`/opsx:audit-drift`](#opsxaudit-drift)
    - [`/opsx:address-reviews [<scope>]`](#opsxaddress-reviews-scope)
    - [`/opsx:review-external <change-name> [--as proposal|system]`](#opsxreview-external-change-name---as-proposalsystem)
    - [`/opsx:archive <change-name>`](#opsxarchive-change-name)
  - [The external review cycle](#the-external-review-cycle)
    - [The full cycle, concretely](#the-full-cycle-concretely)
    - [Why the format is rigid](#why-the-format-is-rigid)
    - [Iteration tracking across runs](#iteration-tracking-across-runs)
    - [Pushback discipline matters most here](#pushback-discipline-matters-most-here)
  - [Project-level structures orbit relies on](#project-level-structures-orbit-relies-on)
    - [`openspec/explore/<name>/`](#openspecexplorename)
    - [`openspec/lenses/`](#openspeclenses)
    - [`openspec/changes/<name>/.orbit-runs/` (per-change)](#openspecchangesnameorbit-runs-per-change)
  - [Marker convention: `@review:`](#marker-convention-review)
  - [Repo layout](#repo-layout)
  - [Design records](#design-records)
  - [v2 / deferred work](#v2--deferred-work)
  - [Installation](#installation)
  - [License](#license)

---

## What this is

orbit is **not** a fork of the OpenSpec CLI. It's a downstream overlay that sits next to your project's existing `.claude/` directory after `openspec init` has run. It overrides/extends specific opsx skills and adds new ones — the CLI binary is unchanged.

The overlay adds four new opsx commands and modifies three existing ones to bake in disciplines that have been working ad-hoc across real projects:

- **Editorial review passes** that complement upstream's structural `verify-change`
- **A drift audit** that catches the gaps `sync-specs` doesn't (the lesson: delta-driven sync is necessary but not sufficient)
- **A formal marker convention** (`@review:`) for inline review notes
- **External-AI handoff packaging** so the cross-AI review cycle has no copy-paste friction
- **An exploration capture layer** so think-mode work becomes durable design records

orbit is developed with the discipline that *if* it were ever upstreamed, the diff should read like a contributor's PR — not like two products mashed together. See [`openspec/changes/bootstrap-openspec-orbit/explore.md`](./openspec/changes/bootstrap-openspec-orbit/explore.md) for the full set of guiding principles and decisions.

---

## The orbit workflow

End-to-end loop, from idea to archived change:

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   1. THINK                                                          │
│   /opsx:explore [<name>]                                            │
│      ↓                                                              │
│   As conversation produces capture-worthy content, /opsx:explore    │
│   offers (doesn't auto-write) to capture each to its right home:    │
│                                                                     │
│   Trigger pattern              → Target file                        │
│   ───────────────────────────────────────────────────────────────   │
│   "we decided X over Y"        → explore.md  Decisions              │
│                                  (proactively captured w/ ack)      │
│   "X calls our Y"              → openspec/lenses/perspectives.md    │
│   "the typical user flow is…"  → openspec/lenses/critical-paths.md  │
│   "we always do X" / "name as" → <topic>_convention.md (root)       │
│   "see file/transcript X"      → explore.md  References             │
│                                                                     │
│   User accepts or declines each offer; declines aren't re-offered   │
│   within the same conversation. Three invocation modes: bare        │
│   (think-only, no file), named (creates/resumes explore.md),        │
│   crystallized (bare → name prompt after 2+ decisions emerge).      │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   2. PROPOSE                                                        │
│   /opsx:propose <name>                                              │
│      ↓                                                              │
│   Reads explore.md; generates proposal.md / design.md / specs/ /    │
│   tasks.md; MOVES openspec/explore/<name>/ → openspec/changes/<     │
│   name>/. explore.md persists as historical record.                 │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   3. REVIEW (proposal side, internal)                               │
│   /opsx:review <name> --as proposal                                      │
│      ↓                                                              │
│   9 passes → 3-dimension scorecard (Completeness/Correctness/       │
│   Coherence). Findings reported with file:line + recommendation.    │
│                                                                  │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   4. REVIEW (proposal side, external)                               │
│   /opsx:review-external <name> --as proposal                        │
│      ↓                                                              │
│   Prompt file written to .orbit-runs/external-prompt-proposal-      │
│   <TS>.md (committed). Tiny invocation snippet emitted to chat →    │
│   user pushes + pastes snippet into codex → codex pulls + reads     │
│   prompt → writes findings to .orbit-runs/external-proposal-<TS>.md │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   5. RESOLVE                                                        │
│   /opsx:address-reviews [--from-file <path>]                        │
│      ↓                                                              │
│   Walks @review: markers OR ingests external-review file. Pushback  │
│   discipline: verify against current state before fixing. Removes   │
│   markers on resolution.                                            │
│                                                                     │
│                  (cycle 3-5 until clean)                            │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   6. APPLY (upstream behavior, unmodified)                          │
│   /opsx:apply <name>                                                │
│      ↓                                                              │
│   Implements tasks; generates code.                                 │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                  │
│   7. REVIEW (system side, internal)                                 │
│   /opsx:review <name> --as system                                        │
│      ↓                                                              │
│   Wraps verify-change (Pass 0) + 6 system-wide passes (baseline,    │
│   cohesion, surface walk, perspectives, critical paths, drift).     │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   8. REVIEW (system side, external)                                 │
│   /opsx:review-external <name> --as system                          │
│      ↓                                                              │
│   Same handoff pattern as step 4, with system-review focus.         │
│                                                                     │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   9. RESOLVE again                                                  │
│   /opsx:address-reviews                                             │
│                                                                     │
│                  (cycle 7-9 until clean)                            │
│                          │                                          │
│                          ▼                                          │
│                                                                     │
│   10. ARCHIVE                                                       │
│   /opsx:archive <name>                                              │
│      ↓                                                              │
│   Auto-invokes /opsx:audit-drift as pre-archive sweep. Critical-    │
│   drift findings prompt user (not block). On confirm: runs sync-    │
│   specs; moves change to openspec/changes/archive/<name>/.          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Periodic (not change-bound):
   /opsx:audit-drift                  ◄── standalone project-wide scan
```

Each numbered step has a `/opsx:` command. The cycles at 3-5 and 7-9 typically run 3-5 iterations on real changes.

---

## Guiding principles

Three stances that shape every design decision in orbit:

1. **Coherence with openspec form and spirit** — orbit speaks openspec's vocabulary, follows its file layout, adopts its reporting conventions. Acid test: if orbit and upstream were merged tomorrow, would the diff read like a contributor's PR?

2. **Up-front cost trumps downstream cost** — defaults lean toward comprehensive; pay LLM/compute cost now to catch issues before they escape. Subagent parallelism (`--parallel`) ships in v1 because the downstream benefit (better signal, scales to bigger projects) is bigger than the per-run token cost.

3. **Specs as the ultimate source of truth** — orbit's tooling assumes the spec set is authoritative; code is one realization. A fresh AI handed the curated `openspec/specs/` should reproduce the system. Users who hold the opposite stance still benefit from the editorial reviews and drift audits; `/opsx:distill-specs` (v2) is the opt-in lever for the spec-truth camp specifically.

## Cross-cutting disciplines

Beyond the three guiding principles above, orbit codifies three cross-cutting **execution disciplines** in the `orbit-conventions` spec. Together they bracket the authoring lifecycle: read-before-reference at authoring time, completeness at modification time, pushback at review time.

> **Read-before-reference discipline (authoring-time)**. The AI MUST read the actual definition of any specific named construct (function signature, type/interface, object shape, file path, spec requirement name) before generating code, tests, specs, or documentation that references it. Inference from training-data patterns or "this is how things are usually structured" is NOT a substitute for reading the actual definition. Specific named constructs require reading; conceptual reasoning (pattern names, architectural style) does not.
>
> **Change completeness discipline (modification-time)**. Any substantive modification to a change-in-flight must be applied fully across all affected artifacts before being declared done. Applies regardless of how the modification was triggered. After mechanical replacement (sed, find-replace), a manual sweep MUST follow looking for residue the pattern didn't reach AND new bugs the pattern introduced. Known residue MUST NOT be left for review to catch — that violates the cost-up-front principle and corrupts the review signal. For external review, cleanup MUST complete BEFORE the prompt file is pushed.
>
> **Pushback discipline (review-time)**. When responding to flagged issues — from inline `@review:` markers, `/opsx:address-reviews --from-file` external findings, or any other source — verify the claim against current state before fixing. Stale findings (issue already fixed) get reported with evidence and suppressed; don't re-edit already-fixed state.

### Recommended `CLAUDE.md` snippet for adopters

If you want to reinforce orbit's disciplines at the project level (recommended), copy this snippet into your project's `CLAUDE.md`:

```markdown
## Working with orbit

This project uses [openspec-orbit](https://github.com/las-sal/openspec-orbit), an opinionated `.claude/` overlay on `@fission-ai/openspec`. Orbit codifies three disciplines worth reinforcing here, one for each phase of authoring work:

**Read-before-reference (authoring-time)**. When you generate code, tests, specs, or documentation that names a specific construct in this codebase — a function, type, interface, field, file path, spec requirement, CLI flag — read the actual definition first. Use `Read`, `grep`, or `openspec instructions` as appropriate. Do NOT assume the shape based on common patterns or training-data conventions. If you can't verify the reference, ask or flag with `@review:`; never invent a plausible-looking reference and proceed silently. Conceptual reasoning (architectural patterns, design style) does not require this verification — the discipline kicks in when a specific named construct enters the output.

**Change completeness (modification-time)**. Substantive modifications to a change-in-flight (renames, capability merges, scope changes, marker conventions, file paths) must be applied fully across ALL affected artifacts before being declared done — specs, sketches, README, design.md, tasks.md, explore.md, command bodies. After mechanical replacement (sed, find-replace), do a manual sweep for residue patterns and new bugs the mechanical replacement may have introduced. Known residue MUST NOT be left for a downstream review to catch — that violates orbit's cost-up-front principle and corrupts the review signal.

**Pushback (review-time)**. When responding to flagged issues — from inline `@review:` markers, `/opsx:address-reviews --from-file` external findings, or any other source — verify the claim against current state before fixing. Stale findings (issue already fixed) get reported with evidence and suppressed; don't re-edit already-fixed state.
```

Behavior of orbit's commands does not depend on this snippet being present (the disciplines are baked into each command's SKILL.md). The snippet is a project-level reinforcement; orbit's review/audit/address commands carry the discipline self-contained.

---

## Command reference

In rough workflow order. Each command has a full design sketch in [`openspec/changes/bootstrap-openspec-orbit/sketches/`](./openspec/changes/bootstrap-openspec-orbit/sketches/).

### `/opsx:explore [<name>]`

> **What's new**: orbit turns conversational explore-mode work into durable design records, without changing what explore *is*. The upstream "thinking partner stance" is preserved: no fixed steps, no required outputs, no implementation. What's added is a set of **capture affordances** that activate as the conversation produces capture-worthy content — so the discipline-of-write-it-down doesn't have to be re-explained to the AI each session.
>
> Specifically:
>
> - **Five capture types**, each with a trigger pattern and a target file: decisions (→ `explore.md`), perspectives (→ `openspec/lenses/perspectives.md`), critical paths (→ `openspec/lenses/critical-paths.md`), conventions (→ `<topic>_convention.md`), references (→ `explore.md`). Decisions are captured proactively with a brief acknowledgment; the other four are *offered*, and the user decides.
> - **`explore.md`** — a five-section file (Premise / Decisions / Open questions / Considered & out / References) maintained proactively as the conversation progresses. Becomes the seed that `/opsx:propose` consumes to generate the formal change artifacts.
> - **Three invocation modes**: bare (`/opsx:explore`) for pure think-mode with no file; named (`/opsx:explore <name>`) to start or resume a captured exploration; crystallized — bare invocation that opens a name prompt after 2+ substantive decisions emerge, so think-mode work doesn't get lost when it turns out to matter.

Think mode for a change. orbit adds capture triggers: when conversation produces a durable convention, perspective, critical path, or decision, explore offers to write it to the right file. Offer, don't auto-capture.

Three invocation modes:

- **Bare** (`/opsx:explore`) — pure think; no file.
- **Named** (`/opsx:explore foo`) — creates `openspec/explore/foo/explore.md` or resumes an existing one.

Plus a state transition: a **bare invocation can crystallize into a named exploration mid-conversation**. After 2+ substantive decisions emerge, explore offers: "We have enough material here to capture — what should we call this exploration?" On accept, explore creates `openspec/explore/<name>/explore.md`, back-fills with what's been discussed, and continues as the named mode. This is not a separate invocation — it's a transition from bare → named driven by what the conversation produces.

Five capture types and their target files:

| Capture | Target |
|---|---|
| Convention | `<topic>_convention.md` at project root |
| Perspective | `openspec/lenses/perspectives.md` |
| Critical path | `openspec/lenses/critical-paths.md` |
| Decision | `openspec/explore/<name>/explore.md` Decisions section |
| Reference | `openspec/explore/<name>/explore.md` References section |

Full design: [`sketches/explore.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/explore.md)

---

### `/opsx:propose <name>`

> **What's new**: consumes `explore.md` and promotes the staging directory. Preserves upstream's standalone behavior.

Generates the standard openspec change artifacts (`proposal.md`, `design.md`, `specs/<capability>/spec.md`, `tasks.md`). Two modes:

- **Consume mode** — when `openspec/explore/<name>/` exists, propose reads `explore.md`, prompts for handling of Open questions, generates artifacts, then *moves* the staging directory to `openspec/changes/<name>/`. The exploration record persists as historical context.
- **Standalone mode** — when no explore directory exists, behaves exactly like upstream propose (description → artifacts).

Section mapping (consume mode):

| `explore.md` section | Feeds into |
|---|---|
| Premise | `proposal.md` motivation |
| Decisions | spec deltas + `design.md` decisions + `tasks.md` |
| Open questions | Resolve now / defer as `@review:` marker / abandon |
| Considered & out | `design.md` "Alternatives considered" |
| References | Contextual reads during generation |

Full design: [`sketches/propose.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/propose.md)

---

### `/opsx:review <name> [--as proposal|system]`

> **What's new**: single editorial review command with `--as` mode flag. No upstream equivalent.

One command, two modes:

- **`--as proposal`** runs 9 passes over a change's pre-implementation artifacts (proposal/design/specs/tasks/explore). Pre-apply gate.
- **`--as system`** wraps upstream `verify-change` as Pass 0 and adds 6 system-wide passes. Post-apply, pre-archive gate.
- When `--as` is omitted, the mode is inferred from `tasks.md` state (unchecked tasks → `proposal`; all checked + code exists → `system`; ambiguous → user is prompted).

#### Proposal-mode passes

| # | Pass | What it checks |
|---|---|---|
| 1 | Structure & Delta Integrity | Required artifacts present; ADDED/MODIFIED/REMOVED/RENAMED valid; `openspec validate` passes |
| 2 | Internal Coherence | Proposal aligns with design aligns with specs aligns with tasks; no scope creep |
| 3 | Cross-Doc Coherence | `CLAUDE.md`, `project.md`, `*_convention.md` still accurate after change |
| 4 | Archive Consistency | New ADDED don't contradict baseline; RENAMED FROM symbols exist; REMOVED still referenced |
| 5 | Codegen Readiness | No implicit requirements; no decisions left to codegen; no ambiguity |
| 6 | Gap Hunt (generative completeness) | Could a fresh AI implement this from these specs alone? |
| 7 | Drift Hunt | Old vocabulary lingering; consistency with `*_convention.md` |
| 8 | Inline Review Marker Residue | Any `@review:` markers still present (must be addressed before apply) |
| 9 | Pre-Handoff Sweep | "Anything else before I ship?" final read |

#### System-mode passes

| # | Pass | What it checks |
|---|---|---|
| 0 | `verify-change` (upstream) | Tasks done; deltas implemented; design adhered |
| 1 | Baseline Compliance | Does this change break behaviors in *archived* `openspec/specs/`? |
| 2 | Cohesion | Callers/dependents outside the tasks — signature drift, ripple effects |
| 3 | Surface Walk | Every CLI/MCP/HTTP surface still coherent? (surfaces derived from `openspec/specs/`) |
| 4 | Perspective Reviews | Validate from each registered caller-perspective in `openspec/lenses/perspectives.md` |
| 5 | Critical-Path Scan | Each flow in `openspec/lenses/critical-paths.md`, walked end-to-end |
| 6 | Drift / Residue | Calls `/opsx:audit-drift` as a library function |

Output (both modes): standard 3-dimension scorecard (Completeness / Correctness / Coherence) with CRITICAL / WARNING / SUGGESTION severities and file:line refs. Final-assessment gate text varies by mode (`/opsx:apply` for proposal, `/opsx:archive` for system).

#### Flags

```
/opsx:review <name> [--as proposal|system]
  [--fast | --full | --thorough]    depth (default --full)
  [--parallel]                       subagent parallelism for heavy passes
  [--focus <lens>]                   rename / flip / refactor / extension / hotpath
  [--fresh]                          clean-context subagent for main work
  [--strict]                         fail-fast on first CRITICAL
  [--mark]                           proposal mode only: drop @review: markers based on findings
  [--skip-verify]                   system mode only: skip Pass 0 if verify-change ran separately
```

Full design: [`sketches/review.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/review.md)

---

### `/opsx:audit-drift`

> **What's new**: project-wide scan for drift in captured knowledge vs. reality. No upstream equivalent.

Four scan categories:

1. **Vocabulary residue** — renamed/removed terms lingering in non-delta'd specs, `project.md`, `CLAUDE.md`, `*_convention.md`
2. **Lens staleness** — entries in `openspec/lenses/` referencing surfaces/capabilities that no longer exist
3. **Cross-doc consistency** — different docs describing the same thing inconsistently
4. **Archive coherence** — recently archived changes whose `sync-specs` step missed updates

Three invocation paths:

- **Standalone** — user invokes when "something feels off"
- **Library call** — `/opsx:review --as system` Pass 6 invokes it internally
- **Auto-invoked** — `/opsx:archive` calls it before completing (opt-out via `--skip-audit`)

Flags:

```
/opsx:audit-drift
  [--fast | --full | --thorough]
  [--parallel]
  [--focus <area>]                   vocabulary | lenses | docs | archive
  [--since <ref>]                    window for archive coherence scan
  [--strict]
```

Full design: [`sketches/audit-drift.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/audit-drift.md)

---

### `/opsx:address-reviews [<scope>]`

> **What's new**: resolves `@review:` markers and external-review findings. No upstream equivalent.

The resolution counterpart to the generative review commands. Its **primary use case** is closing the cross-AI review cycle: ingest the external-review findings file produced by codex (or any external AI following the orbit handoff format) via `--from-file`, and walk each finding with pushback discipline — verify against current state, classify (trivial fix / decision / stale / unresolvable), apply or defer, log.

A **secondary mode** scans the repo for inline `@review:` markers anywhere (default whole repo with safe exclusions). Useful when you've annotated specs, code, or configs during your own read-through and want a structured walk-through with the same pushback discipline. Markers are removed on resolution so they don't leak into canonical artifacts.

Four enforcement wins justify the skill over "just ask the AI":

| Discipline | Without the skill |
|---|---|
| Convention durability | AI uses different syntax each session; tags don't get found later |
| Pushback discipline | AI re-fixes already-fixed state, churns the diff |
| Marker removal invariant | Markers leak into canonical specs |
| Multi-file-type uniformity | Per-file-type marker conventions diverge |

Lifecycle:

```
1. Discover → grep -rn "@review:" scope (default: whole repo with safe exclusions)
              OR parse --from-file <path>
2. Triage   → show numbered list; user can scope
3. Walk each (sequential):
   a. Verify against current state (pushback)
   b. Classify: trivial fix / decision required / stale / unresolvable
   c. Apply (or defer per user choice)
   d. Remove marker
4. Ripple flag → list affected related files (lean v1; no auto-cascade)
5. Report → resolution log: ✓ resolved / ⚠ stale / ⏸ deferred / ✗ escalated
```

Flags:

```
/opsx:address-reviews [<scope>]
  [--from-file <path>]               ingest external-review findings as virtual markers
  [--list]                            preview only; don't act
  [--only <pattern>]                  narrow scan scope
  [--keep-resolved-markers]           debug; don't remove on resolution
```

Full design: [`sketches/address-reviews.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/address-reviews.md)

---

### `/opsx:review-external <change-name> [--as proposal|system]`

> **What's new**: packages a review request for an external AI. No upstream equivalent.

Packages a review request for an external AI (codex, fresh Claude, GPT, etc.) to run a second-opinion review. Output is two things:

1. **A versioned prompt file** written to `openspec/changes/<name>/.orbit-runs/external-prompt-<as>-<TS>.md` — committed to the repo so the external AI can fetch it from GitHub.
2. **A short invocation snippet** emitted to chat (3-5 lines): the prompt file path, a 1-3 sentence copy-paste-ready instruction for the external AI ("Pull <repo URL> and read <prompt-file-path>; follow its instructions"), and the eventual findings path for the user to pass to `/opsx:address-reviews --from-file`.

The user pushes the prompt file, pastes the invocation snippet into the external AI, it pulls the repo and reads the prompt file, then writes findings to `openspec/changes/<name>/.orbit-runs/external-<as>-<TS>.md` (or outputs markdown for the user to save if chat-only).

`--as` mode picks the focus:

| Mode | When to use | What the external AI reviews |
|---|---|---|
| `--as proposal` | Pre-apply | proposal/design/specs/tasks/explore.md — the 9 proposal-mode passes |
| `--as system` | Post-apply | full codebase + baseline + diff — the 7 system-mode passes |
| (no flag) | Inferred | From `tasks.md` state: unchecked → proposal; all checked + code → system |

Flags:

```
/opsx:review-external <change-name>
  [--as proposal|system]             default inferred from change state
```

See [The external review cycle](#the-external-review-cycle) below for the full walkthrough.

Full design: [`sketches/review-external.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/review-external.md)

---

### `/opsx:archive <change-name>`

> **What's new**: pre-archive auto-invokes `/opsx:audit-drift`. Preserves upstream's archive behavior.

Auto-invokes `/opsx:audit-drift` as a pre-archive sweep. Critical-drift findings trigger a three-way prompt (address now / proceed / abort) — not a hard gate. User can archive with known drift if intentional. Writes an archive run summary to `.orbit-runs/archive-<TS>.json` capturing the audit outcome and user decision.

After the audit step: standard upstream behavior — runs `sync-specs`, moves change to `openspec/changes/archive/<name>/`.

Flags:

```
/opsx:archive <change-name>
  [--skip-audit]                     bypass the audit-drift sweep
```

Full design: [`sketches/archive.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/archive.md)

---

## The external review cycle

The reason `/opsx:review-external` and `--from-file` exist: the manual cross-AI review pattern was already working (across ~80 sessions in the originator's home-control / home-env / home-control-test projects), but the copy-paste-per-finding friction was real. orbit makes it frictionless.

### The full cycle, concretely

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                       │
│  IN YOUR AUTHORING SESSION                                            │
│                                                                       │
│  1. Run the internal review first                                     │
│     /opsx:review foo --as proposal                                         │
│     (or /opsx:review foo --as system after apply)                          │
│     → 3-dim scorecard in chat; address what you can                   │
│                                                                       │
│  2. Address findings you agree with                                   │
│     /opsx:address-reviews                                             │
│     (walks any @review: markers you've left in artifacts)             │
│                                                                       │
│  3. Generate the handoff package                                      │
│     /opsx:review-external foo --as proposal                           │
│     → prompt file written:                                            │
│       openspec/changes/foo/.orbit-runs/                               │
│         external-prompt-proposal-<TS>.md                              │
│     → tiny invocation snippet printed to chat (3-5 lines)             │
│                                                                       │
│  4. Push prompt file to GitHub so external AI can fetch it            │
│     git add openspec/changes/foo/.orbit-runs/ && git commit && push   │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘

         │
         │  copy the 3-5 line invocation snippet
         ▼

┌──────────────────────────────────────────────────────────────────────┐
│                                                                       │
│  IN CODEX (or fresh Claude / other AI)                                │
│                                                                       │
│  5. Paste the invocation snippet                                      │
│     (something like:                                                  │
│       "Pull https://github.com/<you>/<repo> and read                  │
│        openspec/changes/foo/.orbit-runs/external-prompt-proposal-     │
│        <TS>.md. Follow the instructions inside.")                     │
│                                                                       │
│  6. External AI pulls the repo and reads the prompt file              │
│     (the prompt file tells it: read change foo + CLAUDE.md +          │
│      project.md + lenses + iteration history, run the 9 proposal      │
│      passes, write findings in the specified format)                  │
│                                                                       │
│  7. External AI WRITES findings to:                                   │
│     openspec/changes/foo/.orbit-runs/external-proposal-<TS>.md        │
│                                                                       │
│     PLUS outputs the FULL findings markdown in chat (same content     │
│     as the file — every severity section, every finding's title,     │
│     file:line, and description). The chat output is your immediate    │
│     read; the file is the canonical record for --from-file parsing.  │
│                                                                       │
│     Format codex writes (in both places):                             │
│                                                                       │
│     # External Review: foo (iteration 2)                              │
│     **Reviewer**: codex                                               │
│     **Date**: 2026-05-18                                              │
│                                                                       │
│     ## CRITICAL                                                       │
│     ### Thermostat target replacement rule is over-broad              │
│     **File**: specs/home-control/spec.md:65                           │
│     **Description**: Lists thermostat in "in-flight target            │
│       replacement" alongside lock/garage/blinds/security, but         │
│       thermostat drift is autonomous; targetTemperature writes are    │
│       instant. Either exclude thermostat or add specific wording.     │
│                                                                       │
│     ## WARNING                                                        │
│     ...                                                                │
│                                                                       │
│  8. External AI COMMITS AND PUSHES the findings file (if git access). │
│     Commit message: "External review (<as>, iter <N>): <change>"      │
│     + one-line summary. Authoring AI can then pull and ingest         │
│     without manual intervention.                                       │
│                                                                       │
│  (If codex environment is chat-only and can't write files, codex      │
│   outputs the markdown above; user copy-pastes it to the path.)       │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘

         │
         │  user pulls latest (or codex pushed)
         ▼

┌──────────────────────────────────────────────────────────────────────┐
│                                                                       │
│  BACK IN AUTHORING SESSION                                            │
│                                                                       │
│  8. Ingest the findings                                               │
│     /opsx:address-reviews --from-file \                               │
│       openspec/changes/foo/.orbit-runs/external-proposal-<TS>.md      │
│                                                                       │
│  9. address-reviews parses the findings file, treats each finding     │
│     as a virtual @review: marker, walks each:                         │
│       - Pushback: verify against current state                        │
│       - Classify: trivial / decision / stale / unresolvable           │
│       - Apply (or defer per user choice)                              │
│       - Log to resolution log                                         │
│                                                                       │
│  10. Resolution log displayed in chat                                 │
│      The external findings file persists in .orbit-runs/ as           │
│      historical record.                                               │
│                                                                       │
│  (cycle back to step 1 if iteration is needed)                        │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

### Why the format is rigid

The CRITICAL / WARNING / SUGGESTION + `### Title` + `**File**: path:line` + `**Description**` structure is parsed deterministically by `address-reviews --from-file`. The handoff prompt tells codex this exact format, so:

- No ambiguity about how to write findings
- Each finding becomes a virtual marker with severity, location, content
- Pushback discipline applies per-finding the same way it applies to inline markers
- The findings file is the historical record, kept in `.orbit-runs/` indefinitely

### Iteration tracking across runs

```
openspec/changes/foo/.orbit-runs/
├── review-proposal-2026-05-17T14-22.json     ← internal proposal run #1
├── review-proposal-2026-05-18T09-10.json     ← internal proposal run #2
├── external-proposal-2026-05-17T16-30.md     ← external proposal review #1 (codex)
├── external-proposal-2026-05-18T11-45.md     ← external proposal review #2 (codex)
├── review-system-2026-05-19T10-00.json       ← internal system run #1
└── external-system-2026-05-19T15-22.md       ← external system review #1
```

Each command increments its own iteration counter. The handoff prompt for round N shows codex "this is iteration N; prior findings were …" so codex has cycle context.

### Pushback discipline matters most here

Codex sees a snapshot. By the time you're ingesting its findings, the snapshot is already stale. The pushback step in `address-reviews` (verify against current state before fixing) is what keeps you from re-editing already-fixed code based on a stale external finding. This is documented as a real failure mode (see `OPENSPEC_LESSONS.md` — kept local; not published).

---

## Project-level structures orbit relies on

orbit adds two new top-level structures inside `openspec/`:

### `openspec/explore/<name>/`

Staging area for `/opsx:explore`. Contains:

- `explore.md` — five sections (Premise / Decisions / Open questions / Considered & out / References)
- Any sibling captures (`sketches/`, draft conventions, etc.)

`/opsx:propose` MOVES this directory to `openspec/changes/<name>/` when promoting. The `explore.md` and siblings persist alongside the generated proposal/design/specs/tasks as historical record.

### `openspec/lenses/`

The judgment layer — what code can't tell you, captured durably:

- `openspec/lenses/perspectives.md` — named callers worth validating from (e.g., "Claude Desktop using MCP", "Swift host calling Python service"). Each entry has surfaces it uses, validation criteria, typical call patterns.
- `openspec/lenses/critical-paths.md` — named user flows worth walking end-to-end. Each entry has touchpoints and expected behavior.

Surfaces themselves are *not* in `lenses/` — capabilities in `openspec/specs/<capability>/` ARE the surfaces. lenses is purely the subjective layer.

Files grow via `/opsx:explore` capture triggers (offer, don't auto). Empty `lenses/` causes graceful degradation: system-mode Passes 4/5 skip with a note; Pass 3 still runs against derived surfaces.

**Entry shape — `openspec/lenses/perspectives.md`**:

```markdown
## <Perspective name>

**Surfaces**: <capability names this perspective interacts with>

**Description**: <who this caller is, what they want>

**Typical call patterns**: <how they exercise the surface>
```

Example:

```markdown
## Claude Desktop using MCP

**Surfaces**: mcp-server, tool-registry

**Description**: End-user-facing AI assistant; calls our MCP server to expose tools to the user's conversation. Cares about: low-latency tool listing, clear tool descriptions, predictable error messages.

**Typical call patterns**: tools/list on startup; tools/call per user request; rarely re-fetches the registry mid-session.
```

**Entry shape — `openspec/lenses/critical-paths.md`**:

```markdown
## <Flow name>

**Description**: <what the user is trying to do>

**Touchpoints**: <capabilities / tools / surfaces involved, in order>

**Expected behavior**: <what should happen end-to-end>
```

Example:

```markdown
## First-time user adds their first device

**Description**: A user has just installed the app and wants to register a smart device. Most critical onboarding flow — failure here loses the user.

**Touchpoints**: device-discovery → device-pairing → device-registry → home-state-sync

**Expected behavior**: Discovery completes within 5s; pairing prompts user only for required info; registry persists immediately; subsequent app starts show the device without re-pairing.
```

### `openspec/changes/<name>/.orbit-runs/` (per-change)

Iteration history for a change. **Committed to the repo, not gitignored** — the iteration record is real evidence of the review cycle and supports team handoffs. Dot-prefix signals "orbit metadata, not part of the canonical openspec change," so upstream's view of the change directory stays clean.

Contains both internal-run summaries (JSON, one per review/audit/archive invocation) and external-review findings (markdown, one per `/opsx:review-external` invocation). Travels with the change into `openspec/changes/archive/<name>/.orbit-runs/` when archived, preserving the full review history alongside the archived artifacts.

**File naming inside `.orbit-runs/`**:

| Filename pattern | Written by | Contents |
|---|---|---|
| `review-proposal-<TS>.json` | `/opsx:review --as proposal` | Proposal-mode review run summary |
| `review-system-<TS>.json` | `/opsx:review --as system` | System-mode review run summary |
| `audit-drift-<TS>.json` | `/opsx:audit-drift` | Drift audit run summary |
| `address-reviews-<TS>.json` | `/opsx:address-reviews` | Marker / external-finding resolution log |
| `external-prompt-<as>-<TS>.md` | `/opsx:review-external` | The handoff prompt for the external AI |
| `external-<as>-<TS>.md` | External AI (per `/opsx:review-external` instructions) | The findings file, ingested via `/opsx:address-reviews --from-file` |
| `archive-<TS>.json` (in archived location) | `/opsx:archive` | Archive run summary (audit findings, user decision, sync-specs results) |

`<TS>` is ISO-8601 with hyphens, e.g., `2026-05-18T14-35-23Z`. `<as>` is `proposal` or `system`.

**Internal-run JSON summary format** — every internal command writes a JSON file with a consistent shape:

```jsonc
{
  "command": "review" | "audit-drift" | "address-reviews" | "archive",
  "timestamp": "<ISO-8601>",
  // command-specific fields vary, but every summary includes:
  //   - a flags object capturing the invocation flags
  //   - a findings_summary or resolution_summary object with severity / outcome counts
  //   - a final_assessment string (or null when the command is a library call)
  // See per-command schemas at .claude/skills/<skill>/references/<schema>.md
}
```

The full per-command schemas live alongside each skill at `.claude/skills/<skill>/references/<schema>.md`:

- `openspec-review/references/run-summary-schema.md`
- `openspec-audit-drift/references/run-summary-schema.md`
- `openspec-address-reviews/references/run-summary-schema.md`
- `openspec-archive-change/references/archive-summary-schema.md`

**External-review markdown findings format** — see the worked example in [The external review cycle](#the-external-review-cycle) section above. The format is parsed by `/opsx:address-reviews --from-file` and must follow the exact structure (severity sections + `### Title` + `**File**:` + `**Description**:` field labels) for the parser to work. Full parser contract: `.claude/skills/openspec-address-reviews/references/external-findings-format.md`.

### `<topic>_convention.md` files at project root

Conventions are the **AI-readable rules layer** — durable patterns that apply broadly across the codebase (e.g., "files use kebab-case", "errors look like X"). Distinct from:

- `CLAUDE.md` (orientation; references convention files)
- `openspec/specs/<cap>/spec.md` (capability-specific behavior)
- `openspec/lenses/` (subjective judgment about what matters for review)
- Linters / pre-commit hooks (syntactic enforcement — orbit doesn't replace these; convention files carry rationale and apply to broader rules tooling can't catch)

**Where they live**: at the project root, named `<topic>_convention.md` (e.g., `naming_convention.md`, `error_handling_convention.md`).

**Internal format** — four sections in fixed order; sections optional when not applicable:

```markdown
# <Topic> Convention

## Purpose
<1-2 sentences: why this convention exists>

## Rules
1. <rule> — <rationale>
2. <rule> — <rationale>

## Examples
**Correct:**
- <example>

**Incorrect:**
- <example>

## Exceptions
<cases where this convention doesn't apply, if any>
```

**Captured during `/opsx:explore`**: when the user describes a durable rule, explore offers to write it to the appropriate `<topic>_convention.md` using the structured format. If a matching file exists, the new rule is appended; if the statement contradicts an existing rule, explore surfaces the contradiction and offers to update.

**Consumed by orbit commands** — conventions aren't just documentation, they're read by review and audit commands:

| Command | What it does with conventions |
|---|---|
| `/opsx:review --as proposal` Pass 3 | Checks proposal / design / spec deltas align with declared conventions |
| `/opsx:review --as proposal` Pass 7 | Flags vocabulary or naming that contradicts conventions |
| `/opsx:review --as system` Pass 2 | Light check that the change's code follows conventions |
| `/opsx:audit-drift` Cat. 3 | Checks conventions don't contradict each other / `CLAUDE.md` / `project.md` |
| `/opsx:audit-drift` Cat. 1 | Flags removed terms still referenced in convention files |
| `/opsx:explore` | Reads relevant convention files when conversation touches their topics; offers updates on contradictions |
| `/opsx:propose` | Honors conventions during artifact generation (e.g., kebab-case capability names) |
| `/opsx:address-reviews` | Ripple-flag lists relevant `*_convention.md` files when resolutions touch their topics |

**Staleness handling**: `/opsx:audit-drift` Category 3 reports conventions that contradict current state — convention says kebab-case but most files are snake_case, etc. — with a recommendation to update the convention or migrate the code.

---

## Marker convention: `@review:`

Inline review markers in any file type. Same syntax across markdown, source code, configs:

```
# spec.md (markdown — bare, sits in prose)
Each device exposes a state attribute. @review: should we
explicitly forbid sensitive devices from this?

# foo.ts (source — inside file-type's comment)
function authorize(token: string) {
  // @review: what if token has expired but is still well-formed?
  return decode(token);
}

# config.yaml (config — inside file-type's comment)
auth:
  token_ttl: 3600  # @review: should this be configurable per env?
```

`/opsx:address-reviews` greps `@review:` regardless of file type. On resolution, markers are removed (so they don't leak into canonical specs).

Unresolvable markers have three options (per-marker user choice):

- File as a follow-up task in `tasks.md` + remove the marker (default)
- Convert to permanent `@todo:` marker
- Leave with `@review(escalated): ...` and explanation

---

## Repo layout

```
.claude/
├── commands/opsx/                ← slash command bodies (orbit ships overrides + new)
│   ├── explore.md  propose.md  apply.md  archive.md  verify.md
│   ├── new.md  continue.md  fast-forward.md  sync.md  onboard.md  bulk-archive.md
│   └── review.md  review-external.md  audit-drift.md  address-reviews.md   ← orbit (new)
└── skills/                        ← skill definitions
    ├── openspec-explore/         ← upstream + orbit additions (capture types, three modes)
    ├── openspec-propose/         ← upstream + orbit additions (consume mode)
    ├── openspec-archive-change/  ← upstream + orbit additions (pre-archive audit + summary)
    │   └── references/
    ├── openspec-review/          ← orbit new (proposal + system modes)
    │   └── references/           ← run-summary schema
    ├── openspec-review-external/ ← orbit new
    │   └── references/           ← prompt template, mode sections
    ├── openspec-audit-drift/     ← orbit new
    │   └── references/           ← run-summary schema
    ├── openspec-address-reviews/ ← orbit new
    │   └── references/           ← external-findings format, run-summary schema
    └── openspec-*/                ← other upstream skills (verify-change, apply, etc.)

openspec/
├── changes/                       ← in-flight changes + archive (upstream)
│   └── <name>/.orbit-runs/       ← orbit addition: per-change iteration history
├── specs/                         ← canonical baseline specs (upstream)
├── explore/                       ← orbit addition: staging for /opsx:explore
│   └── <name>/
│       ├── explore.md
│       └── sketches/             (optional)
├── lenses/                        ← orbit addition: judgment layer
│   ├── perspectives.md
│   └── critical-paths.md
├── .orbit-runs/                   ← orbit addition: standalone runs (not change-scoped)
└── config.yaml                    ← openspec config (upstream)

CLAUDE.md                          ← orbit addition: project-level discipline reinforcement
LICENSE                            ← MIT
README.md                          ← this file
.gitignore
```

---

## Design records

The complete design exploration that produced orbit lives in this repo as a dogfooded example of the workflow:

- **`openspec/changes/bootstrap-openspec-orbit/explore.md`** — full design record. Guiding principles, decisions (grouped by area), considered alternatives, references.
- **`openspec/changes/bootstrap-openspec-orbit/sketches/`** — detailed per-command sketches:
  - [`review.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/review.md) (unified — both modes)
  - [`audit-drift.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/audit-drift.md)
  - [`address-reviews.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/address-reviews.md)
  - [`review-external.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/review-external.md)
  - [`explore.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/explore.md)
  - [`propose.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/propose.md)
  - [`archive.md`](./openspec/changes/bootstrap-openspec-orbit/sketches/archive.md)

This repo eats its own dogfood: the exploration that produced orbit was conducted using the very workflow orbit captures.

---

## v2 / deferred work

Tracked as GitHub issues:

- [#1](https://github.com/las-sal/openspec-orbit/issues/1) — Caching of pass results in system-mode review (and proposal-mode review)
- [#2](https://github.com/las-sal/openspec-orbit/issues/2) — Define `--thorough` mode extras precisely for review commands
- [#3](https://github.com/las-sal/openspec-orbit/issues/3) — Comprehensive `/opsx:address-reviews` features (paste, cascade, severity filtering, strict, parallel, categorized markers, system-side `--mark`, auto-rerun)

Plus `/opsx:distill-specs` (canonical-spec hygiene — periodic curation toward regen-readiness) deferred to v2 with scope notes captured in `explore.md`.

---

## Installation

orbit is a `.claude/` overlay on `@fission-ai/openspec`. Install in three steps:

### 1. Set up upstream OpenSpec first

If you haven't already:

```bash
npx @fission-ai/openspec@latest init
```

This creates `openspec/` and `.claude/` with the 11 upstream openspec-* skills (plus the `feedback` skill). Verify with `openspec list`.

### 2. Overlay orbit

```bash
git clone https://github.com/las-sal/openspec-orbit /tmp/orbit
cp -r /tmp/orbit/.claude/commands/opsx/. .claude/commands/opsx/
cp -r /tmp/orbit/.claude/skills/. .claude/skills/
rm -rf /tmp/orbit
```

This adds four new orbit commands (`/opsx:review`, `/opsx:review-external`, `/opsx:audit-drift`, `/opsx:address-reviews`) and layers orbit additions on top of three upstream skills (`openspec-explore`, `openspec-propose`, `openspec-archive-change`). The other nine upstream skills are unchanged.

### 3. (Recommended) Add the orbit discipline reminder to your CLAUDE.md

Copy the snippet from this README's [Cross-cutting disciplines](#cross-cutting-disciplines) section into your project's `CLAUDE.md`. Orbit's commands work without it (the disciplines are baked into each SKILL.md), but the snippet reinforces them at the project level for non-orbit-command AI work.

### Verify

After overlay, the available skills should include `openspec-review`, `openspec-review-external`, `openspec-audit-drift`, `openspec-address-reviews`, plus the existing upstream skills now showing "openspec-orbit" in their `metadata.author` for the three modified ones.

```bash
ls .claude/skills/openspec-*/SKILL.md | wc -l   # 15 total (11 upstream openspec-* + 4 new orbit)
ls .claude/commands/opsx/*.md                    # 15 commands
```

### Partial adoption is fine

Orbit is designed to gracefully degrade. You can install just the new commands and skip the upstream-skill modifications by copying only specific files. Each new orbit command is independent of the others. Each modification to an upstream skill is purely additive (the upstream content is preserved verbatim).

### Updates

Re-run the overlay to pick up new orbit versions. orbit follows upstream's MIT license. Pin to a specific commit if you need stability:

```bash
git clone --branch v0.1.0 --depth 1 https://github.com/las-sal/openspec-orbit /tmp/orbit
```

Long-term: package as a proper Claude Code plugin (Phase 2).

---

## License

MIT — same as upstream [@fission-ai/openspec](https://github.com/Fission-AI/OpenSpec). See [LICENSE](./LICENSE).
