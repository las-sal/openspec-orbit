# orbit-conventions

## Purpose

Cross-cutting conventions consumed by all orbit commands: `@review:` marker syntax (plus `@review(escalated):` and `@todo:` companion forms), `openspec/lenses/` judgment-layer entry shapes, `openspec/explore/<name>/` staging-directory structure, `explore.md` five-section format, `.orbit-runs/` persistence layout (committed; per-change iteration history; moves with archive), internal-run JSON summary format, external-review markdown findings format, 3-dimension scorecard reporting convention, severity ladder (CRITICAL / WARNING / SUGGESTION) with false-positive bias, actionable-findings requirement, stock final-assessment phrasings, the **three execution disciplines** that bracket the authoring lifecycle (read-before-reference / change-completeness / pushback), the **distribution model** (pegged engine + orbit-owned surface), **upstream version pinning** (a specific `@fission-ai/openspec` version that orbit is contractually tested against), the **overlay file disposition** (per-file categorization: orbit-authored / orbit-modified / upstream-required primitive / not-shipped), the **verb-prefix naming taxonomy** (`verify-*` / `review-*` / `audit-*` / `distill-*`), the **convention file** format and consumer behavior, and the boundary with linter/hook tooling.

## Requirements

### Requirement: `@review:` marker convention

The system SHALL recognize `@review: <content>` as the inline review marker convention across all file types.

#### Scenario: Markdown — bare

- **WHEN** a markdown file contains `@review: <text>` in prose
- **THEN** the marker is recognized as an inline review request and is discoverable by `/opsx:address-reviews`

#### Scenario: Source code — inside file-type comments

- **WHEN** a source code file (TypeScript, JavaScript, Swift, Go, Rust, C, etc.) contains `// @review: <text>` (or `/* @review: <text> */`)
- **THEN** the marker is recognized; the marker text is the `@review:` portion onward

#### Scenario: Hash-comment files

- **WHEN** a Python, Ruby, shell, YAML, TOML, or similar hash-comment file contains `# @review: <text>`
- **THEN** the marker is recognized

#### Scenario: Single grep pattern

- **WHEN** a tool scans for markers
- **THEN** the grep pattern `@review:` finds markers across all supported file types without per-type wrappers

### Requirement: `@review(escalated):` and `@todo:` companion conventions

The system SHALL recognize two companion markers used by `/opsx:address-reviews` for unresolvable items: `@review(escalated): <content>` (deliberately persisted unresolved review) and `@todo: <content>` (permanent TODO).

#### Scenario: Escalated marker recognized

- **WHEN** a file contains `@review(escalated): <text>`
- **THEN** tools treat it as deliberately persisted; address-reviews does not re-prompt it unless the user explicitly opts in

#### Scenario: TODO marker recognized

- **WHEN** a file contains `@todo: <text>`
- **THEN** tools treat it as a permanent TODO, not as a review item

### Requirement: `openspec/lenses/` directory structure

The system SHALL define `openspec/lenses/` as the project-level judgment layer.

#### Scenario: Required files

- **WHEN** the lenses layer is in use
- **THEN** it contains `openspec/lenses/perspectives.md` and `openspec/lenses/critical-paths.md`

#### Scenario: Other lens files permitted

- **WHEN** future orbit lens types emerge (e.g., `error-paths.md`, `performance-budgets.md`)
- **THEN** they can be added to `openspec/lenses/` following the same content shape

### Requirement: `openspec/lenses/perspectives.md` entry shape

The system SHALL use a consistent entry shape for named perspectives.

#### Scenario: Per-entry structure

- **WHEN** a perspective entry is written
- **THEN** it uses the form: `## <Perspective Name>` heading, `**Surfaces**: <surface IDs>`, description prose, `**Typical call patterns**:` list

### Requirement: `openspec/lenses/critical-paths.md` entry shape

The system SHALL use a consistent entry shape for named critical paths.

#### Scenario: Per-entry structure

- **WHEN** a critical path entry is written
- **THEN** it uses the form: `## <Flow Name>` heading, description prose, `**Touchpoints**:` numbered list, `**Expected behavior**:` list (SLOs / contracts / invariants)

### Requirement: Lenses files grown via explore capture

The system SHALL grow `openspec/lenses/` files via `/opsx:explore` capture triggers, not via direct user authoring required.

#### Scenario: Empty lenses on fresh project

- **WHEN** a project is newly set up with orbit
- **THEN** `openspec/lenses/` may be empty; orbit commands degrade gracefully

#### Scenario: Captured during explore

- **WHEN** the user describes a caller or critical flow during exploration and accepts the capture offer
- **THEN** the new entry is appended to the appropriate file in `openspec/lenses/`

### Requirement: Surfaces NOT redeclared in lenses

The system SHALL derive surfaces from `openspec/specs/<capability>/spec.md` rather than redeclaring them in lenses.

#### Scenario: Surface reference

- **WHEN** a perspective entry references a surface ID
- **THEN** the ID corresponds to a capability under `openspec/specs/<capability>/`; the surface's behavior is defined by that capability's spec, not duplicated in lenses

### Requirement: `openspec/explore/<name>/` staging directory

The system SHALL define `openspec/explore/<name>/` as the staging area for `/opsx:explore` work prior to promotion via `/opsx:propose`.

#### Scenario: Created by explore in named/crystallized modes

- **WHEN** `/opsx:explore` creates a named exploration
- **THEN** the directory `openspec/explore/<name>/` is created with an `explore.md` file

#### Scenario: Sibling files supported

- **WHEN** additional captures occur during exploration
- **THEN** sibling files (e.g., `sketches/*.md`, draft conventions) may be written under `openspec/explore/<name>/`

#### Scenario: Promoted via propose

- **WHEN** `/opsx:propose <name>` consumes a staging directory
- **THEN** the entire `openspec/explore/<name>/` directory is moved to `openspec/changes/<name>/`

### Requirement: `explore.md` five-section format

The system SHALL use a fixed five-section format for `explore.md`.

#### Scenario: Section order

- **WHEN** an `explore.md` file is created
- **THEN** the sections appear in this order: Premise, Decisions, Open questions, Considered & out, References

#### Scenario: Status header

- **WHEN** an `explore.md` file is created
- **THEN** it includes a `> **Status**: exploring...` header indicating the file is a pre-propose record

### Requirement: `.orbit-runs/` per-change persistence

The system SHALL use one of the following `.orbit-runs/` persistence locations, scoped by the type of work being persisted:

- **`openspec/changes/<change-name>/.orbit-runs/`** — per-change iteration history for active changes. Used by editorial commands (review, address-reviews, audit-drift inline), workflow commands operating on a named change (propose, new, continue, ff, apply, verify), lifecycle commands (archive), and review-external T0. The primary persistence location.
- **`openspec/explore/<name>/.orbit-runs/`** — per-exploration iteration history for named-mode `/opsx:explore` sessions BEFORE the exploration is promoted to a formal change via `/opsx:propose`. Moves into `openspec/changes/<name>/.orbit-runs/` when `/opsx:propose <name>` consumes the staging directory (per the orbit-propose consume-mode convention).
- **`openspec/.orbit-runs/`** — project-scope iteration history for commands that have no change scope (currently: `/opsx:audit-drift` invoked with no `<change-name>` argument, i.e., project-wide standalone). Created if it doesn't exist.

In all three locations, files are committed (the directory SHOULD NOT be in `.gitignore`) and follow the same naming pattern (`<command>-<ISO-timestamp>.json` for internal-run JSONs; `external-<as>-<ISO-timestamp>.md` for external-review files).

#### Scenario: Active-change emit goes to changes/<name>/.orbit-runs/

- **WHEN** orbit writes a run summary for a named, active change (e.g., `/opsx:review foo`, `/opsx:apply foo`, `/opsx:archive foo`)
- **THEN** the file lands at `openspec/changes/foo/.orbit-runs/<command>-<TS>.json`

#### Scenario: Named-mode exploration emit goes to explore/<name>/.orbit-runs/

- **WHEN** `/opsx:explore foo` (named mode) writes its run-summary JSON at a conversation boundary
- **THEN** the file lands at `openspec/explore/foo/.orbit-runs/explore-<TS>.json` (NOT `openspec/changes/foo/.orbit-runs/`, since `foo` is still in the staging area pre-`/opsx:propose`)

#### Scenario: Project-wide standalone emit goes to openspec/.orbit-runs/

- **WHEN** `/opsx:audit-drift` runs with no change argument and writes its run-summary JSON
- **THEN** the file lands at `openspec/.orbit-runs/audit-drift-<TS>.json` (create the directory if it doesn't exist); the JSON's `change` field is `null`

#### Scenario: Exploration .orbit-runs/ travels with promotion to changes/

- **WHEN** `/opsx:propose foo` (consume mode) moves the staging directory `openspec/explore/foo/` into `openspec/changes/foo/`
- **THEN** the contents of `openspec/explore/foo/.orbit-runs/` (including any prior `explore-<TS>.json` emits) are preserved and now live at `openspec/changes/foo/.orbit-runs/`; subsequent emits go to the new location

#### Scenario: Committed, not gitignored (all three locations)

- **WHEN** orbit writes a run summary or external-review file to any of the three `.orbit-runs/` locations
- **THEN** the file is intended to be committed (none of the three directories should be in `.gitignore`); each represents real iteration history

#### Scenario: Internal run summary file name pattern

- **WHEN** a review, audit, address-reviews, archive, or workflow command writes a summary
- **THEN** the file is named `<command>-<ISO-timestamp>.json` (e.g., `review-proposal-2026-05-21T00-18-14Z.json`, `apply-2026-05-21T13-34-12Z.json`)

#### Scenario: External review file name pattern

- **WHEN** `/opsx:review-external` produces a prompt file (or external AI produces a findings file following the prompt)
- **THEN** the file is named `external-prompt-<as>-<ISO-timestamp>.md` for prompts and `external-<as>-<ISO-timestamp>.md` for findings (e.g., `external-prompt-proposal-2026-05-21T01-32-52Z.md`, `external-proposal-2026-05-21T01-39-56Z.md`)

#### Scenario: Travels with archive

- **WHEN** the change is archived via `/opsx:archive`
- **THEN** the `.orbit-runs/` directory at `openspec/changes/<name>/.orbit-runs/` moves to `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/` as part of the change content (the project-wide `openspec/.orbit-runs/` location is NOT moved — it's project-scope and persists across changes)

### Requirement: Internal-run JSON summary format

The system SHALL use a consistent JSON shape for run-summary emissions across **all orbit commands that mutate artifacts** — workflow, editorial, and lifecycle alike. Every run-summary JSON SHALL include a universal spine; commands extend the spine with per-kind and per-command extensions.

The universal spine (6 required fields):

```
command          string       identifies which command emitted (matches filename prefix)
timestamp        string       ISO-8601 UTC; JSON field uses standard colon format `YYYY-MM-DDTHH:MM:SSZ`
                              (e.g., `"2026-05-21T13:34:12Z"`). Filename embeds a colon-replaced `<TS>` token
                              `YYYY-MM-DDTHH-MM-SSZ` (e.g., `propose-2026-05-21T13-34-12Z.json`) because
                              colons aren't filesystem-safe across all platforms.
change           string|null  the change name (or null for project-scope commands)
final_assessment string       narrative of what just happened (human-readable)
next_recommended string       verbatim recommendation, suitable for orbit-status best-effort parse
kind             enum         "workflow" | "editorial" | "lifecycle"
```

Per-kind extensions:

- **`kind: "workflow"`** — emitted by `explore`, `propose`, `new`, `continue`, `ff`, `apply`, `verify`. Per-command extensions are defined in the `orbit-run-summary-emit` capability (e.g., `apply.chunk_complete`, `verify.verdict`, `explore.decisions_captured`).
- **`kind: "editorial"`** — emitted by `review`, `address-reviews`, `audit-drift`, `review-external`. Per-command extensions include: `iteration` (when applicable to the command — e.g., review iter-N, address-reviews iter-N), `findings_summary` (counts by severity; included when findings are present — i.e., review/address-reviews/audit-drift completion emits; review-external at T0 emits before external findings return and SHALL omit this field), `finding_titles` (array of brief titles; included with `findings_summary`, omitted in the same cases), plus command-specific fields defined in per-skill schema references at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`.
- **`kind: "lifecycle"`** — emitted by `archive` only. Per-command extensions include: `archive_path`, `audit`, `sync_specs` (transitional — persists from pre-#6 architecture when `/opsx:sync-specs` was a separate command; openspec-orbit#6 will deprecate/remove `/opsx:sync-specs` entirely, at which point this field will be removed or repurposed in a follow-up change), `unresolved_markers`, `user_decision`, plus other fields defined in the archive skill.

**Canonical examples** (one per kind, illustrating spine + per-kind extensions):

Workflow-kind example — `apply-2026-05-21T13-34-12Z.json` written at chunk 2 completion of a chunked apply:

```json
{
  "command": "apply",
  "timestamp": "2026-05-21T13:34:12Z",
  "change": "add-detail-flag",
  "final_assessment": "Completed chunk 2 of 5 (inventory+parsing); 28 of 76 tasks done.",
  "next_recommended": "/opsx:apply add-detail-flag — next chunk: phase+attention+recommendation engine",
  "kind": "workflow",
  "tasks_completed": 28,
  "tasks_remaining": 48,
  "chunk": "2 of 5",
  "chunk_name": "inventory+parsing",
  "chunk_complete": true,
  "tasks_completed_this_session": 16
}
```

Editorial-kind example — `review-proposal-2026-05-21T00-18-14Z.json` written by `/opsx:review --as proposal` iter-1:

```json
{
  "command": "review",
  "timestamp": "2026-05-21T00:18:14Z",
  "change": "emit-run-summary-jsons-from-workflow-commands",
  "final_assessment": "5 CRITICAL findings + 7 WARNING + 7 SUGGESTION. Ready for /opsx:address-reviews.",
  "next_recommended": "/opsx:address-reviews emit-run-summary-jsons-from-workflow-commands --from-file <findings-bridge>",
  "kind": "editorial",
  "mode": "proposal",
  "iteration": 1,
  "depth": "full",
  "passes_run": ["1","2","3","4","5","6","7","8","9"],
  "findings_summary": {
    "critical": 5,
    "warning": 7,
    "suggestion": 7
  }
}
```

Lifecycle-kind example — `archive-2026-05-20T16-31-59Z.json` written after `/opsx:archive bootstrap-orbit-status-cli`:

```json
{
  "command": "archive",
  "timestamp": "2026-05-20T16:31:59Z",
  "change": "bootstrap-orbit-status-cli",
  "final_assessment": "Archived bootstrap-orbit-status-cli to openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/.",
  "next_recommended": "Change archived. Run /opsx:new or /opsx:explore to start the next change, or /opsx:audit-drift for a project-wide drift check.",
  "kind": "lifecycle",
  "archive_path": "openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/",
  "audit": {
    "ran": true,
    "findings_summary": { "critical": 0, "warning": 0, "suggestion": 0 }
  },
  "user_decision": "proceeded_with_no_critical",
  "warnings": []
}
```

#### Scenario: Universal spine present on every emit

- **WHEN** any orbit command (workflow, editorial, or lifecycle) writes a run-summary JSON
- **THEN** the JSON contains the 6 spine fields with valid values: `command`, `timestamp`, `change` (or null), `final_assessment`, `next_recommended`, and `kind`

#### Scenario: Workflow command emit shape

- **WHEN** a workflow command (e.g., `/opsx:propose`, `/opsx:apply`, `/opsx:verify`) writes its JSON
- **THEN** `kind` equals `"workflow"`; per-command extensions are present per the `orbit-run-summary-emit` capability (which defines the specific extension fields per workflow command)

#### Scenario: Editorial command emit shape (with findings)

- **WHEN** an editorial command that has produced findings (`/opsx:review`, `/opsx:address-reviews`, `/opsx:audit-drift`) writes its JSON
- **THEN** `kind` equals `"editorial"`; per-command extensions include `iteration` (when the command tracks iterations), `findings_summary` (counts by severity), `finding_titles` (array of brief titles), plus command-specific fields documented at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`

#### Scenario: Editorial command emit shape (pre-findings T0 case)

- **WHEN** `/opsx:review-external` writes its T0 JSON (prompt packaged, external findings not yet returned)
- **THEN** `kind` equals `"editorial"`; per-command extensions include `mode`, `prompt_path`, `target`, `awaiting_findings: true` — and OMIT `findings_summary` and `finding_titles` because no findings exist yet

#### Scenario: Lifecycle command emit shape

- **WHEN** `/opsx:archive` writes its JSON
- **THEN** `kind` equals `"lifecycle"`; per-command extensions include `archive_path`, `audit`, `sync_specs`, `unresolved_markers`, `user_decision` per the archive skill's emit

#### Scenario: orbit-status tier-1 reader parses next_recommended uniformly across kinds

- **WHEN** orbit-status reads any run-summary JSON (regardless of `kind`) and best-effort parses `next_recommended` per `orbit-status-recommendation/spec.md:7`
- **THEN** the leading `/opsx:<verb> [args]` token (if present) is extracted into `command` and `args`; on parse failure (e.g., prose recommendation), the full string is preserved in `reason`

#### Scenario: Future consumers route by `kind` field rather than parsing filename prefix

- **WHEN** a downstream consumer (dashboard, CI bot, IDE plugin) reads `.orbit-runs/*.json` files
- **THEN** the consumer MAY route by reading the `kind` field directly rather than pattern-matching filename prefixes (e.g., `review-*`, `address-reviews-*`); filename prefix routing remains valid as a fallback

### Requirement: External-review findings markdown format

The system SHALL use a consistent markdown shape for external-review findings files.

#### Scenario: Required structure

- **WHEN** an external-review findings file is written
- **THEN** it contains: `# External Review: <change> (iteration N)` title, `**Reviewer**:` and `**Date**:` fields, severity sections (`## CRITICAL`, `## WARNING`, `## SUGGESTION`), each containing `### <Title>` entries with `**File**: <path>:<line>` and `**Description**: <text>` fields; optional `## Notes` section

#### Scenario: Parseable by --from-file

- **WHEN** `/opsx:address-reviews --from-file <path>` reads this format
- **THEN** the parser extracts per-finding severity, title, file:line, and description

### Requirement: 3-dimension scorecard reporting convention

The system SHALL use a consistent 3-dimension scorecard for review and audit reports.

#### Scenario: Standard dimensions

- **WHEN** a review or audit command emits a scorecard
- **THEN** the dimensions are Completeness / Correctness / Coherence (matching upstream `verify-change`)

#### Scenario: Status column carries metrics

- **WHEN** the scorecard is rendered as a table
- **THEN** the Status column contains brief metrics (e.g., "X/Y tasks, N reqs", "2 baseline regressions, 1 cohesion drift"), not just issue counts

### Requirement: Severity ladder

The system SHALL use a consistent severity ladder across reports.

#### Scenario: Three severities

- **WHEN** a finding is reported
- **THEN** it is one of CRITICAL, WARNING, or SUGGESTION; never any other severity name

#### Scenario: False-positive bias

- **WHEN** a command is uncertain whether a finding rises to a given severity
- **THEN** it picks the lower severity (SUGGESTION over WARNING; WARNING over CRITICAL)

### Requirement: Actionable findings

The system SHALL include actionable detail in every finding.

#### Scenario: File:line + recommendation

- **WHEN** a finding is reported
- **THEN** it includes a `file:line` reference (or equivalent location) and a specific actionable recommendation; never vague text like "consider reviewing X"

### Requirement: Final-assessment phrasings

The system SHALL use stock phrasings for the final-assessment line of review and audit reports.

#### Scenario: Stock phrasings

- **WHEN** a scorecard report is emitted
- **THEN** the final-assessment line uses one of:
  - `X critical issue(s) found. Fix before <gate>.`
  - `No critical issues. Y warning(s) to consider. Ready <gate> (with noted improvements).`
  - `All checks passed. Ready <gate>.`
  where `<gate>` is `/opsx:apply` for `/opsx:review --as proposal`, `/opsx:archive` for `/opsx:review --as system`, and varies per audit-drift invocation context

### Requirement: Pushback discipline

The system SHALL verify findings against current state before reporting (for review/audit) or before fixing (for address-reviews).

#### Scenario: Discipline applies across commands

- **WHEN** a finding's basis may have changed since the finding was generated or proposed
- **THEN** the command checks current state (grep, git log, file read) before acting; if the finding is stale, it is suppressed (review/audit) or skipped (address-reviews) with evidence

### Requirement: Convention files at project root

The system SHALL place convention files (e.g., `naming_convention.md`) at the project root rather than under `openspec/` or in a subdirectory.

#### Scenario: Convention file location

- **WHEN** a convention is captured during explore
- **THEN** the target file is at the project root (e.g., `naming_convention.md`); `CLAUDE.md` (also at root) may reference these files

#### Scenario: Topic-specific naming

- **WHEN** a new convention category is captured
- **THEN** the file name follows the pattern `<topic>_convention.md` (e.g., `naming_convention.md`, `error_handling_convention.md`)

### Requirement: Convention file internal format

The system SHALL use a defined light-structured format for convention files: four named sections in a fixed order, with sections optional when not applicable to the topic.

#### Scenario: Required section order

- **WHEN** a convention file is created
- **THEN** the sections appear in this order when present: `## Purpose`, `## Rules`, `## Examples`, `## Exceptions`

#### Scenario: Purpose section

- **WHEN** a Purpose section is present
- **THEN** it contains 1-2 sentences explaining why the convention exists

#### Scenario: Rules section

- **WHEN** a Rules section is present
- **THEN** it contains numbered or bulleted rule statements, each with a brief rationale; rules use SHALL / MUST for normative force where applicable

#### Scenario: Examples section

- **WHEN** an Examples section is present
- **THEN** it shows correct usage (and may show incorrect usage) so AI and human readers can disambiguate edge cases

#### Scenario: Exceptions section

- **WHEN** an Exceptions section is present
- **THEN** it explicitly enumerates the cases where the convention does not apply

#### Scenario: Optional sections omitted

- **WHEN** a section does not apply to the convention's topic
- **THEN** the section header MAY be omitted; the remaining sections still appear in the canonical order

### Requirement: Convention file consumers

The system SHALL have multiple orbit commands consume convention files, with each consumer's behavior defined.

#### Scenario: `/opsx:review --as proposal` Pass 3 — cross-doc coherence

- **WHEN** Pass 3 (Cross-Doc Coherence) runs
- **THEN** it reads all `*_convention.md` files at project root and checks that the change's proposal, design, and spec deltas align with declared conventions; violations are reported as findings

#### Scenario: `/opsx:review --as proposal` Pass 7 — drift hunt

- **WHEN** Pass 7 (Drift Hunt) runs
- **THEN** it cross-checks the change's artifacts against current conventions, flagging vocabulary or naming that contradicts a declared convention

#### Scenario: `/opsx:review --as system` Pass 2 — cohesion

- **WHEN** Pass 2 (Cohesion) runs
- **THEN** it lightly checks that the change's code follows declared conventions; heavy syntactic enforcement is out of scope (that's a linter's job)

#### Scenario: `/opsx:audit-drift` Category 3 — cross-doc consistency

- **WHEN** Category 3 runs
- **THEN** it checks that conventions do not contradict each other, and that conventions do not contradict `CLAUDE.md` or `project.md`

#### Scenario: `/opsx:explore` reads conventions for context

- **WHEN** explore is running and the conversation touches a topic with an existing convention file
- **THEN** the AI reads the relevant `*_convention.md` to inform its responses; it offers updates if the conversation contradicts the existing convention

#### Scenario: `/opsx:propose` honors conventions during artifact generation

- **WHEN** propose generates artifacts (consume or standalone mode)
- **THEN** the generated content respects declared conventions (e.g., capability names follow `naming_convention.md`)

#### Scenario: `/opsx:address-reviews` ripple flag includes conventions

- **WHEN** a marker's resolution touches a topic that has a convention file
- **THEN** the resolution log's ripple-flag list includes the relevant `*_convention.md` for user awareness

### Requirement: Convention staleness handling

The system SHALL detect and surface stale convention files.

#### Scenario: Convention contradicts current state

- **WHEN** `/opsx:audit-drift` Category 3 finds that a convention contradicts the code's current behavior (e.g., convention says kebab-case but most files are snake_case)
- **THEN** a WARNING is reported with a recommendation to either update the convention to match reality or migrate the code to match the convention

#### Scenario: Convention contradicts another convention

- **WHEN** Category 3 finds two conventions making incompatible claims
- **THEN** a WARNING is reported with both file:line references

#### Scenario: Convention file references removed concepts

- **WHEN** a convention file mentions a term/symbol/concept that has been removed from the codebase per archived change deltas
- **THEN** Category 1 (Vocabulary residue) flags the file:line with a recommendation to update or remove the reference

### Requirement: Convention scope distinct from other knowledge layers

The system SHALL maintain conventions as a distinct knowledge layer from CLAUDE.md, project.md, specs, and lenses.

#### Scenario: Convention vs. CLAUDE.md

- **WHEN** a user describes a durable rule that applies broadly (e.g., "files use kebab-case")
- **THEN** the capture target is `<topic>_convention.md`, not `CLAUDE.md`; CLAUDE.md is reserved for orientation and may reference convention files

#### Scenario: Convention vs. spec

- **WHEN** a user describes a behavioral requirement specific to a capability
- **THEN** the capture target is the capability's spec, not a convention file; conventions apply broadly across the codebase, specs describe capability-specific behavior

#### Scenario: Convention vs. lens

- **WHEN** a user describes a caller or critical flow worth validating from
- **THEN** the capture target is `openspec/lenses/`, not a convention file; lenses describe judgment about *what matters for review*, conventions describe *how the system should be built*

### Requirement: Change completeness discipline

The system SHALL apply substantive modifications to a change-in-flight fully across all affected artifacts before the modification is declared done. Residue cleanup is not optional and is not deferred to subsequent review cycles. This requirement applies regardless of how the modification was triggered — formal openspec command, mid-cycle decision captured during exploration, or direct user request in chat all qualify equally.

#### Scenario: Substantive modification triggers full completeness sweep

- **WHEN** the AI makes a substantive change to a change-in-flight (rename, capability merge, scope change, command consolidation, marker-convention change, file-naming change, or any modification that touches multiple artifacts)
- **THEN** before declaring the modification done, the AI: (a) greps the change directory + `README.md` + sketches + `design.md` + `tasks.md` + `explore.md` for all plausible variants of the old form (capitalized, snake-cased, hyphenated, slash-prefixed-command, capability-name, file-name, JSON-key); (b) classifies each match as current-state or legitimate-historical (in Considered & out, alternatives-considered, prior-decision context); (c) fixes the current-state matches; (d) re-validates with `openspec validate <change>` and re-runs the grep to confirm zero current-state residue

#### Scenario: Mechanical replacement requires manual follow-up

- **WHEN** the AI uses `sed`, find-replace, or other mechanical replacement to apply a rename or pattern change
- **THEN** it MUST follow up with a manual review of each touched file looking for: (a) residue the mechanical pattern didn't reach (places the pattern was too narrow); (b) new bugs introduced by overly-broad replacement (e.g., file-name corruption like `"proposal-mode review.md"` produced by a pattern that should have stopped at word boundaries, or duplicate-entry corruption when two distinct names collapse to the same target); (c) awkward phrasing artifacts (e.g., parenthetical inserts like `"review (system mode) wraps"` that read clumsily and need a second-pass rewording)

#### Scenario: Known residue MUST NOT be left for review

- **WHEN** the AI is about to invoke `/opsx:review`, `/opsx:review-external`, or any other review-cycle command after a substantive modification
- **THEN** it MUST first verify completeness per the previous scenarios; known residue (residue the AI is aware of) MUST NOT be deferred to the review cycle to catch — leaving known issues for a reviewer corrupts the review signal (the reviewer wastes effort on things the AI knew about) and violates the cost-up-front guiding principle (small cost now to surface known issues, save the larger cost of having the reviewer rediscover them)

#### Scenario: Modification scope distinguishes substantive from cosmetic

- **WHEN** evaluating whether a modification requires the full completeness sweep
- **THEN** modifications that affect command names, capability names, file paths, marker conventions, file-format conventions, JSON schemas, or any term used across multiple artifacts qualify as **substantive** (full sweep required); modifications that affect a single isolated typo, a single comment in a single file, or a wording polish in a single paragraph are **cosmetic** (no full sweep required)

#### Scenario: Cleanup-before-prompt-push timing for external review

- **WHEN** the AI plans to follow up a substantive modification with `/opsx:review-external` (writing a prompt file and pushing it for an external AI to pull)
- **THEN** the completeness sweep MUST complete BEFORE the prompt file is pushed; doing the cleanup concurrent with an in-progress external review means the external AI is analyzing stale state and produces mixed-state findings the authoring AI then has to pushback-suppress at ingest time — friction that is avoidable by ordering cleanup first

#### Scenario: Lesson origin in `explore.md`

- **WHEN** the AI executes this requirement
- **THEN** it MAY reference the specific origin: this discipline was codified after a real-time failure during the bootstrap-openspec-orbit dogfood where (a) a major rename (`/opsx:review-proposal` + `/opsx:review-system` → unified `/opsx:review --as <mode>`) was applied via sed; (b) the AI knew residue remained; (c) the AI proposed letting the next review cycle catch it; (d) the user correctly pushed back that this violated the cost-up-front principle; (e) the AI's corrective cleanup landed concurrent with the in-progress external review, producing the mixed-state findings problem the previous scenario describes. The lesson and its origin are captured in `explore.md` Decisions for future readers.

### Requirement: Read-before-reference discipline

The system SHALL read the actual definition of any specific named construct (function signature, type/interface, class definition, object shape, named symbol, file path, import target, API surface, spec requirement name) before generating code, tests, specs, or documentation that references it. Inference from training-data patterns, common conventions, or "this is how this is usually structured" is NOT a substitute for reading the actual definition. This discipline applies whenever the AI authors content that names or invokes specific local constructs.

#### Scenario: Test code references production code

- **WHEN** the AI is writing a test that calls, mocks, asserts on, or otherwise references a specific production-code construct (a function, method, class, object, API endpoint, schema field, etc.)
- **THEN** the AI MUST read the production code first via `Read`, `grep`, or `openspec instructions` — and use the actual signature/shape verbatim; the AI MUST NOT assume the shape based on conventions (e.g., must NOT assume `user.email` when the code may actually use `user.emailAddress` or `user.contact_email`)

#### Scenario: Implementation references existing code

- **WHEN** the AI is writing implementation that calls an existing function, instantiates an existing class, imports from an existing module, or constructs an object of an existing type
- **THEN** the AI MUST read the existing definition first — function signatures must match, type/interface shapes must match, import paths must verify, named exports must exist

#### Scenario: Spec deltas reference baseline capabilities

- **WHEN** a spec delta (`## ADDED`, `## MODIFIED`, `## REMOVED`, `## RENAMED Requirements`) references a baseline requirement by name
- **THEN** the AI MUST read the baseline spec at `openspec/specs/<capability>/spec.md` to confirm the requirement exists with the exact name being referenced; a `RENAMED FROM` symbol that doesn't exist in baseline is a structural error caught at the authoring step, not at review

#### Scenario: Cross-spec references in specs

- **WHEN** a scenario in one spec references a construct, command, or concept defined in another spec
- **THEN** the AI MUST grep for the referenced name (e.g., `grep -rn "<symbol>" openspec/`) before authoring the reference; if the name doesn't resolve, either the reference is wrong or the cross-spec target is missing — both surface at authoring time, not as a downstream finding

#### Scenario: Tool invocations or CLI commands

- **WHEN** the AI is invoking a CLI tool or shell command that takes specific flags / subcommands / arguments
- **THEN** the AI MUST verify the flag exists (via `<tool> --help`, `openspec --help`, manpage, or documentation) before using it; the AI MUST NOT guess flags based on "common patterns" (e.g., must NOT assume `--verbose` exists if the tool documents `-v` or `--debug`)

#### Scenario: Scope of "read" is the specific construct, not the whole module

- **WHEN** the AI applies this discipline
- **THEN** "read" means reading enough of the construct to verify the reference — typically a `grep -n <symbol>` to locate, followed by reading 10–50 lines of context around the definition; it does NOT mean "read the entire file" or "read the entire codebase" — that would be infeasible. The bar is: enough context to know the construct exists and has the shape you're using.

#### Scenario: When verification is genuinely uncertain

- **WHEN** the AI cannot verify a reference (the file doesn't exist, the symbol isn't found, the documentation is ambiguous)
- **THEN** the AI MUST either (a) ask the user via `AskUserQuestion` for clarification, or (b) report the uncertainty in the output (`@review: unable to verify <reference>; <fallback assumption>`), or (c) refuse to generate the reference. The AI MUST NOT invent a plausible-looking reference and proceed silently.

#### Scenario: Conceptual reasoning vs. specific references

- **WHEN** the AI is writing prose, design rationale, or architectural discussion at a conceptual level (e.g., "this looks like an Observer pattern", "the API follows REST conventions")
- **THEN** read-before-reference does NOT require verifying every detail — conceptual reasoning is allowed. The discipline kicks in when a SPECIFIC NAMED CONSTRUCT enters the output (a specific function name, a specific field, a specific import path); at that boundary, verification is required.

#### Scenario: Lesson origin

- **WHEN** the AI executes this requirement
- **THEN** it MAY reference the specific origin: this discipline was codified after a real-world failure during home-env development where the AI wrote a test case that assumed the structure of an object based on common patterns instead of reading the actual code. The test passed locally (because the AI's mock matched its own assumption) but the underlying assumption was wrong. The same failure mode in production code rather than test code would have shipped broken behavior. The discipline distinguishes authoring-time verification (read the source before referencing) from review-time pushback (verify against current state when responding to findings) and modification-time completeness (apply changes fully across all affected artifacts).

### Requirement: orbit does not replace linter/hook tooling

The system SHALL define convention files as an AI-readable rules layer that coexists with — does not replace — automated tooling like linters, formatters, and pre-commit hooks.

#### Scenario: Automated tooling preserved

- **WHEN** a project uses linters (`.eslintrc`, `.prettierrc`, etc.) or pre-commit hooks
- **THEN** orbit's convention files do NOT duplicate or replace those tools; the linter remains the authoritative enforcement mechanism for syntactic rules, while convention files carry rationale and apply to broader rules tooling can't catch

#### Scenario: Light vs heavy enforcement boundary

- **WHEN** an orbit command (e.g., `/opsx:review --as system` Pass 2) "lightly checks" code against conventions
- **THEN** "light" means: grep-pattern matching against expected names; spot-checks against named rules; spot-reading of representative files. "Heavy" means: full AST analysis; exhaustive walk of every line; per-file conformance certification — these are the linter's job, not orbit's. Light checks surface SUGGESTIONS for the user to investigate; the AI does not certify full conformance

### Requirement: Distribution model — pegged engine + orbit-owned surface

The system SHALL be distributed as a `.claude/` overlay pegged to a specific upstream `@fission-ai/openspec` version, not as a fork of the upstream OpenSpec CLI. Orbit owns the `.claude/` surface (skills, commands, supporting docs); upstream supplies the CLI binary as a pinned engine.

#### Scenario: Overlay shape

- **WHEN** orbit ships
- **THEN** the deliverable is markdown content under `.claude/commands/opsx/` and `.claude/skills/openspec-*/`, plus supporting docs; the upstream `@fission-ai/openspec` CLI binary is unchanged

#### Scenario: Consumers install after openspec init at pinned version

- **WHEN** a user adopts orbit in a project
- **THEN** they first install upstream `@fission-ai/openspec` at orbit's pinned version (see `Upstream version pinning` requirement), run upstream `openspec init` to scaffold the project's `.claude/` and `openspec/` directories, then drop orbit's overrides + new files into the same locations; orbit does not include or replace upstream's CLI

#### Scenario: Pinned upstream version is contractual

- **WHEN** orbit is in use against an upstream version other than the pinned one
- **THEN** orbit's behavior is undefined; users are responsible for verifying their installed upstream version matches orbit's pin (per `Upstream version pinning` requirement)

#### Scenario: Overlay-overwrite behavior is intentional under pegging

- **WHEN** orbit's overlay copies files into `.claude/skills/` or `.claude/commands/opsx/` that share names with upstream-`init`-installed files
- **THEN** the overwrite is the intended distribution mechanism; under pegging there is no "fresh upstream copy to preserve" — the pinned upstream version produces deterministic files, and orbit's overrides are the orbit-owned versions for that pinned upstream

### Requirement: Upstream version pinning

The system SHALL declare a specific pinned upstream `@fission-ai/openspec` version that orbit is tested against and contractually depends on. Orbit's behavior outside the pinned version is undefined. Version upgrade is a deliberate change-proposal event, not an automatic ingest.

#### Scenario: Pinned version declared in CLAUDE.md

- **WHEN** a user or AI reads `CLAUDE.md` at the orbit repo root
- **THEN** the pinned upstream version is clearly stated in the introductory or installation context

#### Scenario: Pinned version declared in README

- **WHEN** a user reads the orbit `README.md`
- **THEN** the install section names the exact pinned version (e.g., `@fission-ai/openspec@1.3.1`) as a required prerequisite

#### Scenario: Doc-only enforcement is acceptable

- **WHEN** orbit ships without an install script (current state)
- **THEN** documentation-only declaration is sufficient enforcement; install-time version-check is tracked as a separate future enhancement (orbit issue #26)

#### Scenario: Version upgrade is a deliberate change

- **WHEN** orbit's contributors consider upgrading the pinned upstream version
- **THEN** the upgrade is proposed as its own OpenSpec change (with explicit upgrade-and-port scope), not performed as silent maintenance; the new pinned version replaces the prior one in CLAUDE.md + README + this requirement's documented value

#### Scenario: Upstream improvements do not auto-propagate

- **WHEN** upstream releases `@fission-ai/openspec` v1.3.2, v1.4.0, or later while orbit is pinned to an earlier version
- **THEN** orbit users do not receive those improvements until a deliberate upgrade change lands in orbit; this is an accepted trade-off (per design.md D-arch-1)

### Requirement: Overlay file disposition

The system SHALL classify every file orbit ships in `.claude/` into one of four disposition categories. Each category dictates the file's lifecycle in orbit's overlay.

#### Scenario: Orbit-authored — full ownership

- **WHEN** orbit ships a file with no upstream-derived content (e.g., `openspec-review`, `openspec-audit-drift`, `openspec-review-external`, `openspec-address-reviews`)
- **THEN** the file is fully owned by orbit; orbit's contributors edit it freely; the file MAY use the `openspec-*` directory naming convention for consistency with siblings even when the body is 100% orbit-authored

#### Scenario: Orbit-modified — upstream body with `# Orbit additions`

- **WHEN** orbit ships a file that bundles upstream's body with an appended `# Orbit additions` section (e.g., `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`)
- **THEN** the upstream body is the pinned-version snapshot; orbit's additions append behavior or constraints; this pattern is transitional and a subject of future Option-2 work (dropping the additions pattern in favor of fully orbit-authored versions of these skills)

#### Scenario: Upstream-required primitive — kept verbatim

- **WHEN** orbit depends on an upstream skill as a callable primitive (invoked at the orchestration layer, not modified at the skill-body layer)
- **THEN** orbit ships the upstream skill unchanged; orbit's use of the primitive matches upstream's internal use pattern; no `# Orbit additions` are needed because orbit's interaction is at the invocation layer, not the skill body

#### Scenario: Not shipped — pruned from overlay

- **WHEN** an upstream skill is truly unmodified, not used by orbit, and provides no orbit-mission value (e.g., `feedback`, which sends user feedback to Fission-AI and has no role in orbit's pegged workflow)
- **THEN** orbit does NOT ship the file in its overlay; the skill is simply absent from orbit's `.claude/skills/` directory. NOTE: because the overlay install model is `cp -r` (which does not delete target-only files), absence-in-overlay is necessary but not sufficient to ensure absence-in-user-project. The README install/update sections SHALL document explicit `rm` commands for pruned files so user-project state matches overlay intent.

#### Scenario: Per-skill disposition is documented

- **WHEN** orbit's contributors evaluate whether to ship a new upstream skill that lands in a future pinned-version upgrade
- **THEN** the disposition decision is recorded in the change proposal that performs the upgrade; the new skill is assigned to one of the four categories with rationale

#### Scenario: Commands follow the same 4-category framework

- **WHEN** orbit ships (or chooses not to ship) a slash-command file in `.claude/commands/opsx/`
- **THEN** the file's disposition is one of the four categories on the same criteria as skills: orbit-authored (e.g., `opsx/review.md`, `opsx/audit-drift.md`, `opsx/review-external.md`, `opsx/address-reviews.md`); orbit-modified (mirrors an upstream-bound capability with orbit-specific behavior, e.g., `opsx/propose.md`, `opsx/explore.md`, `opsx/archive.md`, `opsx/apply.md`, `opsx/verify.md`, `opsx/continue.md`, `opsx/fast-forward.md`, `opsx/new.md`, `opsx/bulk-archive.md`, `opsx/sync.md`); upstream-required primitive (none currently — orbit does not depend on any upstream command file as a callable primitive); not shipped (rare — applied per-file when an upstream command provides no orbit-mission value)

#### Scenario: Command-file disposition documented in the same change as a removal or addition

- **WHEN** orbit decides a command should be added to the overlay or pruned from it
- **THEN** the decision is recorded in the change proposal that performs the addition/removal with the disposition category cited; pruned commands must include `rm` step in README install/update documentation (analogous to pruned-skill handling)

### Requirement: Verb-prefix naming taxonomy preserved

The system SHALL preserve the four-verb-prefix naming taxonomy across all current and future orbit commands.

#### Scenario: Verb categories

- **WHEN** a new orbit command is proposed
- **THEN** its name uses one of the four established verbs based on its operation: `verify-*` (upstream — structural correctness checks), `review-*` (orbit — opinionated editorial passes), `audit-*` (orbit — scan for drift/residue/staleness), `distill-*` (orbit — reduce to essential)

#### Scenario: New verb requires justification

- **WHEN** a proposed command does not fit any of the four established verbs
- **THEN** the proposal MUST justify the new verb explicitly (in `design.md` or equivalent) before the command's spec is accepted; ad-hoc verb proliferation is rejected

### Requirement: Install documentation describes actual install surface

The README's installation, update, and uninstall sections SHALL describe the actual install surface produced by the pinned `@fission-ai/openspec` version (per `Upstream version pinning`). Skill counts, command counts, file lists, CLI invocations, and behavior warnings SHALL match what a fresh sandbox install produces. This requirement exists because the README-vs-install-surface drift has caused two cluster-2 change cuts (issue #6's original delete-9-files framing and the original `lean-overlay-and-add-orbit-onboard` chunks 3-5 cut on 2026-05-23); discipline alone was not sufficient to prevent recurrence.

#### Scenario: README skill and command counts match install output

- **WHEN** a reader follows the README's documented install steps in a fresh sandbox (`mktemp -d` + same Node version as the pinned upstream supports)
- **THEN** the resulting `.claude/skills/` and `.claude/commands/opsx/` directories contain exactly the files the README's "What you should see after install" section enumerates, with counts matching exactly; no file is documented that isn't installed, and no installed file is omitted from the documentation

#### Scenario: CLI invocations in README are non-interactively executable

- **WHEN** a reader executes the CLI commands documented in the README's install, update, or uninstall sections in a non-interactive shell (stdin closed)
- **THEN** each command succeeds (or exits with the specific error/warning the README documents); no command produces an unexpected error from missing required flags, no command hangs on a prompt the README didn't warn about, no command's exit behavior contradicts the prose

#### Scenario: README-modifying changes pair with sandbox verification

- **WHEN** a change proposal modifies README install/update/uninstall sections
- **THEN** the change SHALL include an apply-time fresh-sandbox verification task that runs the documented CLI sequence end-to-end and confirms the file-system state matches the documentation; verification failures escalate via `@review(escalated):` rather than ship undocumented behavior

#### Scenario: Upgrade and overlay-change proposals include README sync

- **WHEN** a change proposal upgrades the pinned upstream version (per `Upstream version pinning`) OR modifies overlay file disposition (per `Overlay file disposition` — adding to or removing from the four categories)
- **THEN** the proposal SHALL include a README install-section synchronization task that updates counts, file lists, and CLI invocations to match the new install surface; the synchronization task is required before the upgrade/overlay-change is considered apply-complete

#### Scenario: User-facing behavior warnings are accurate, not pessimistic

- **WHEN** the README documents the behavior of a CLI command (e.g., `init --force` overwrite scope, `cp -r` overlay non-deletion behavior)
- **THEN** the warning text SHALL accurately describe what the command does in the pinned version, neither understating nor overstating risk; overly-cautious warnings that don't match actual behavior are themselves drift and SHALL be corrected to match sandbox-verified facts
