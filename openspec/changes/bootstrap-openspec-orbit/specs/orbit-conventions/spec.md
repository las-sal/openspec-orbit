## ADDED Requirements

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

The system SHALL define `openspec/changes/<change-name>/.orbit-runs/` as the per-change iteration history directory.

#### Scenario: Committed, not gitignored

- **WHEN** orbit writes a run summary or external-review file
- **THEN** the file is intended to be committed (the directory should not be in `.gitignore`); it represents real iteration history

#### Scenario: Internal run summary file name pattern

- **WHEN** a review or audit command writes a summary
- **THEN** the file is named `<command>-<ISO-timestamp>.json` (e.g., `review-proposal-2026-05-17T14-22.json`)

#### Scenario: External review file name pattern

- **WHEN** `/opsx:review-external` produces a findings file (or external AI does, following the prompt)
- **THEN** the file is named `external-<as>-<ISO-timestamp>.md` (e.g., `external-proposal-2026-05-17T16-30.md`)

#### Scenario: Travels with archive

- **WHEN** the change is archived
- **THEN** the `.orbit-runs/` directory moves to `openspec/changes/archive/<name>/.orbit-runs/` as part of the change content

### Requirement: Internal-run JSON summary format

The system SHALL use a consistent JSON shape for internal-run summaries.

#### Scenario: Summary fields

- **WHEN** a review/audit/archive command writes a JSON summary
- **THEN** it includes at minimum: `command`, `timestamp`, `change` (if change-scoped), `iteration` (if applicable), `findings_summary` (counts by severity, optionally by pass/category), and `finding_titles` (array of brief titles)

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
  where `<gate>` is `/opsx:apply` for proposal-mode review, `/opsx:archive` for system-mode review, and varies per audit-drift invocation context

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

### Requirement: orbit does not replace linter/hook tooling

The system SHALL define convention files as an AI-readable rules layer that coexists with — does not replace — automated tooling like linters, formatters, and pre-commit hooks.

#### Scenario: Automated tooling preserved

- **WHEN** a project uses linters (`.eslintrc`, `.prettierrc`, etc.) or pre-commit hooks
- **THEN** orbit's convention files do NOT duplicate or replace those tools; the linter remains the authoritative enforcement mechanism for syntactic rules, while convention files carry rationale and apply to broader rules tooling can't catch

#### Scenario: Light vs heavy enforcement boundary

- **WHEN** an orbit command (e.g., `/opsx:review --as system` Pass 2) "lightly checks" code against conventions
- **THEN** "light" means: grep-pattern matching against expected names; spot-checks against named rules; spot-reading of representative files. "Heavy" means: full AST analysis; exhaustive walk of every line; per-file conformance certification — these are the linter's job, not orbit's. Light checks surface SUGGESTIONS for the user to investigate; the AI does not certify full conformance

### Requirement: Distribution model — overlay, not CLI fork

The system SHALL be distributed as a `.claude/` overlay rather than as a fork of the upstream OpenSpec CLI.

#### Scenario: Overlay shape

- **WHEN** orbit ships
- **THEN** the deliverable is markdown content under `.claude/commands/opsx/` and `.claude/skills/openspec-*/`, plus supporting docs; the upstream `@fission-ai/openspec` CLI binary is unchanged

#### Scenario: Consumers install after openspec init

- **WHEN** a user adopts orbit in a project
- **THEN** they first run upstream `openspec init` to scaffold the project's `.claude/` and `openspec/` directories, then drop orbit's overrides + new files into the same locations; orbit does not include or replace upstream's CLI

### Requirement: Verb-prefix naming taxonomy preserved

The system SHALL preserve the four-verb-prefix naming taxonomy across all current and future orbit commands.

#### Scenario: Verb categories

- **WHEN** a new orbit command is proposed
- **THEN** its name uses one of the four established verbs based on its operation: `verify-*` (upstream — structural correctness checks), `review-*` (orbit — opinionated editorial passes), `audit-*` (orbit — scan for drift/residue/staleness), `distill-*` (orbit — reduce to essential)

#### Scenario: New verb requires justification

- **WHEN** a proposed command does not fit any of the four established verbs
- **THEN** the proposal MUST justify the new verb explicitly (in `design.md` or equivalent) before the command's spec is accepted; ad-hoc verb proliferation is rejected
