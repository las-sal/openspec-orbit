# orbit-conventions

## Purpose

Cross-cutting conventions consumed by all orbit commands: `@review:` marker syntax (plus `@review(escalated):` and `@todo:` companion forms), `openspec/lenses/` judgment-layer entry shapes, `openspec/explore/<name>/` staging-directory structure, `explore.md` five-section format, `.orbit-runs/` persistence layout (committed; per-change iteration history; moves with archive), internal-run JSON summary format, external-review markdown findings format, 3-dimension scorecard reporting convention, severity ladder (CRITICAL / WARNING / SUGGESTION) with false-positive bias, actionable-findings requirement, stock final-assessment phrasings, the **three execution disciplines** that bracket the authoring lifecycle (read-before-reference / change-completeness / pushback), the **distribution model** (overlay, not CLI fork), the **verb-prefix naming taxonomy** (`verify-*` / `review-*` / `audit-*` / `distill-*`), the **convention file** format and consumer behavior, and the boundary with linter/hook tooling.

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
- **THEN** the `.orbit-runs/` directory moves to `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/` as part of the change content

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
