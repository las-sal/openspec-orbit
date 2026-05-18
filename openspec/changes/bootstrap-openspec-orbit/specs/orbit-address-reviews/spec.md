## ADDED Requirements

### Requirement: Address-reviews command available

The system SHALL expose a `/opsx:address-reviews` command that scans for `@review:` markers in the repo (or ingests external-review findings via `--from-file`) and walks each through resolution with pushback discipline.

#### Scenario: Default scan

- **WHEN** the user invokes `/opsx:address-reviews` with no arguments
- **THEN** the command greps the whole repo for `@review:` markers, respecting safe exclusions (`.git`, `node_modules`, `dist`, `build`)

#### Scenario: Scoped scan

- **WHEN** the user invokes `/opsx:address-reviews <scope>` with a path or pattern
- **THEN** the command restricts the scan to the specified scope while keeping safe exclusions

### Requirement: `--from-file` ingest of external-review findings

The system SHALL accept a `--from-file <path>` flag and parse the file's markdown structure into virtual markers for resolution.

#### Scenario: Parse external findings file

- **WHEN** the command runs with `--from-file <path>` and the file follows the orbit external-review markdown format
- **THEN** the parser extracts each finding (severity, title, file:line, description) as a virtual marker and walks it through the same lifecycle as inline markers, except no source-file marker text exists to remove

#### Scenario: Malformed input

- **WHEN** the `--from-file` path is malformed or missing required sections
- **THEN** the command reports a parse error with format guidance and exits without acting on partial input

### Requirement: Discover → triage → walk → ripple flag → report lifecycle

The system SHALL execute five lifecycle stages: (1) discover markers (or load from file), (2) triage with user (numbered list, optional scoping), (3) walk each sequentially with pushback discipline, (4) ripple flag (list affected related files without auto-cascading), (5) report a resolution log.

#### Scenario: Triage scoping

- **WHEN** markers are discovered and presented as a numbered list
- **THEN** the user can specify subset scoping (e.g., "just 1-3") and only those markers are walked

### Requirement: Pushback discipline applied per marker

The system SHALL verify each marker against current state before fixing.

#### Scenario: Already fixed at HEAD

- **WHEN** the marker's claim references something already fixed in the current state (verified via `grep`, `git log`, or file inspection)
- **THEN** the marker is classified as "stale, suppressed"; current state and evidence (commit hash or grep result) are reported; the marker is removed without further edit

#### Scenario: Still applies

- **WHEN** the marker's claim is still valid against current state
- **THEN** the resolution proceeds to classification

#### Scenario: Pushback decision procedure

- **WHEN** the AI verifies a marker against current state
- **THEN** it follows this procedure in order: (1) identify the marker's referenced symbol, name, or concept; (2) `grep -rn` for current presence in expected locations (the file the marker is in, related files, baseline specs); (3) if the symbol is absent where the marker expects it, run `git log -S "<symbol>" --since=<reasonable-window>` (default: since the marker file was last modified) to confirm intentional removal; (4) read the relevant file's current content; (5) compare to the marker's claim and decide: still applies / already fixed / partially applies; (6) on already-fixed, report the commit hash or current-content evidence as part of suppression

### Requirement: Classification of each marker

The system SHALL classify each marker as one of: trivial fix, decision required, stale, or unresolvable.

#### Scenario: Classification heuristics

- **WHEN** the AI classifies a marker after pushback
- **THEN** it applies these heuristics in order: (a) **stale** — pushback determined the issue is already resolved at HEAD; (b) **trivial fix** — the resolution is a single-line edit or a few-line localized edit with one obvious correct answer (no design implication, no scope question, no ambiguity in intent); (c) **decision required** — the resolution requires ambiguity resolution, a design choice between defensible alternatives, a scope decision, or has implications beyond the immediate location; (d) **unresolvable** — the resolution needs information not currently available (e.g., depends on a deferred decision, requires a future capability, blocked on external input)

#### Scenario: Trivial fix

- **WHEN** a marker is classified as trivial fix
- **THEN** the command proposes the edit, applies it, and removes the marker (no `AskUserQuestion` needed)

#### Scenario: Decision required

- **WHEN** a marker is classified as decision required
- **THEN** the command surfaces 2-4 concrete options via `AskUserQuestion` (rather than asking open-ended); applies the user's choice; removes the marker

#### Scenario: Unresolvable — default file as task

- **WHEN** the marker can't be resolved now and the user has not specified an alternative
- **THEN** the command files a follow-up task in `tasks.md` and removes the marker

#### Scenario: Unresolvable — convert to permanent TODO

- **WHEN** the user chooses to convert an unresolvable marker
- **THEN** the marker text is replaced with `@todo: <content>` and the resolution log notes the conversion

#### Scenario: Unresolvable — escalate

- **WHEN** the user chooses to escalate
- **THEN** the marker is replaced with `@review(escalated): <content with explanation>` and the resolution log notes the escalation

### Requirement: Marker removal invariant

The system SHALL remove the original `@review:` marker on resolution unless `--keep-resolved-markers` is set.

#### Scenario: Marker removed by default

- **WHEN** a marker has been resolved (trivial fix applied, decision applied, or stale-suppression)
- **THEN** the marker text is deleted from its source file

#### Scenario: `--keep-resolved-markers` debug flag

- **WHEN** the command runs with `--keep-resolved-markers`
- **THEN** markers remain in their source files even after resolution (debug use)

### Requirement: Ripple flag without auto-cascade

The system SHALL list related files affected by each resolution without automatically editing them.

#### Scenario: Resolution touches normative content

- **WHEN** a marker's resolution edits a normative spec or design statement
- **THEN** the resolution log lists potentially-affected related files (sibling specs in the change dir, `CLAUDE.md`, `openspec/project.md`, `*_convention.md`, `openspec/lenses/`)

#### Scenario: Out-of-scope files not flagged

- **WHEN** computing related files for a ripple flag
- **THEN** baseline specs in `openspec/specs/<capability>/` and source code are NOT flagged (those are `/opsx:apply` + `sync-specs` territory)

### Requirement: Resolution log output

The system SHALL emit a resolution log (not a 3-dimension scorecard) summarizing per-item outcomes.

#### Scenario: Output structure

- **WHEN** the command completes
- **THEN** the output includes a summary table with counts of ✓ Resolved, ⚠ Stale, ⏸ Deferred, ✗ Escalated; followed by per-section listings of each marker's file:line, brief description, action taken, and any related files flagged

#### Scenario: Final assessment

- **WHEN** the resolution log is complete
- **THEN** the final assessment reports remaining markers in scope (0 if clean, plus count of any deliberately persisted escalated markers) and suggests next step (e.g., "re-run /opsx:review --as proposal to confirm clean baseline")

### Requirement: Marker convention across file types

The system SHALL recognize `@review:` markers uniformly across file types: bare in markdown; inside the file type's comment syntax in source code and configs.

#### Scenario: Markdown marker

- **WHEN** a markdown file contains `@review: <text>` (bare)
- **THEN** the marker is discovered and resolution applies

#### Scenario: Source code marker (C-style comment)

- **WHEN** a source file contains `// @review: <text>` or `/* @review: <text> */`
- **THEN** the marker is discovered; resolution can either remove the marker text alone (leaving the comment) or remove the comment entirely if it contained only the marker

#### Scenario: Source code marker (hash comment)

- **WHEN** a source file or config contains `# @review: <text>`
- **THEN** the marker is discovered with the same removal behavior

### Requirement: No new markers without user consent

The system SHALL NOT create new `@review:` markers as part of its own operation.

#### Scenario: address-reviews does not write markers

- **WHEN** address-reviews runs
- **THEN** the only marker creation possible is by `/opsx:review --as proposal --mark` (a different command); address-reviews never writes new markers
