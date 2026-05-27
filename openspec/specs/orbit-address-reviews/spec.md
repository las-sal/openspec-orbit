# orbit-address-reviews

## Purpose

The `/opsx:address-reviews` command (lean v1) — resolves `@review:` markers anywhere in the repo (or external-review findings via `--from-file`) by walking each through pushback → classify → fix → ripple-flag → remove-marker. Pushback is the primary discipline: verify against current state before fixing, suppress stale findings with evidence. Output is a resolution log (NOT a scorecard) with ✓ Resolved / ⚠ Stale / ⏸ Deferred / ✗ Escalated counts. Marker-removal invariant ensures markers don't leak into canonical artifacts.

## Requirements

### Requirement: Address-reviews command available

The system SHALL expose a `/opsx:address-reviews` command that scans for `@review:` markers in the repo (or ingests external-review findings via `--from-file`) and walks each through resolution with pushback discipline.

#### Scenario: Default scan

- **WHEN** the user invokes `/opsx:address-reviews` with no arguments
- **THEN** the command greps the whole repo for `@review:` markers, respecting safe exclusions (`.git`, `node_modules`, `dist`, `build`)

#### Scenario: Scoped scan

- **WHEN** the user invokes `/opsx:address-reviews <scope>` with a path or pattern
- **THEN** the command restricts the scan to the specified scope while keeping safe exclusions

### Requirement: `--from-file` ingest of review findings file

The system SHALL accept a `--from-file <path>` flag and parse the file's content into virtual markers for resolution. The parser SHALL auto-detect the file's format via content sniff and support TWO format families: (a) external-review markdown (produced by external AIs per `references/external-findings-format.md`), (b) internal findings JSON (produced by `/opsx:review` or `/opsx:audit-drift`, per `references/internal-findings-format.md`). V1 accepts JSON files whose top-level `command` field is either `"review"` or `"audit-drift"`; other `command` values are rejected with a clean error. Each finding becomes a virtual marker that walks the same lifecycle as inline markers, with marker-removal as a no-op (no source-file marker text exists to remove for either virtual-marker type).

#### Scenario: Parse external findings markdown

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace prefix is `# External Review:` (markdown header) and the body follows the orbit external-review markdown format
- **THEN** the parser extracts each finding (severity, title, file:line, description) as a virtual marker with `source: "external"` and walks it through the same lifecycle as inline markers, except no source-file marker text exists to remove

#### Scenario: Parse internal review JSON

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace character is `{` AND the file parses as valid JSON AND the top-level `command` field is `"review"` AND the JSON shape matches the orbit internal-run review summary (per orbit-conventions `Internal-run JSON summary format` + the review-specific extensions documented at `.claude/skills/openspec-review/references/run-summary-schema.md`)
- **THEN** the parser extracts each entry in the JSON's `findings[]` array as a virtual marker — mapping `severity` directly, `title` directly, `file:line` from `file` + `line` fields, `description` from the entry's `recommendation` field — tags `source: "internal-review"` on each, and walks each through the same lifecycle as inline markers, with marker-removal as a no-op

#### Scenario: Parse internal audit-drift JSON

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace character is `{` AND the file parses as valid JSON AND the top-level `command` field is `"audit-drift"` AND the JSON shape matches the orbit audit-drift summary (per orbit-conventions `Internal-run JSON summary format` + the audit-drift-specific extensions documented at `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`)
- **THEN** the parser extracts each entry in the JSON's `findings[]` array as a virtual marker — mapping `severity` directly, `title` directly, `file:line` from `file` + `line` fields, `description` from the entry's `recommendation` field, with the audit-drift `category` field (`"1"`–`"4"`) used as the provenance/pass-slot detail in the virtual marker — tags `source: "audit-drift"` on each, and walks each through the same lifecycle as inline markers, with marker-removal as a no-op. The audit-drift virtual-marker lifecycle is identical to internal-review except for the `source` tag value and the provenance detail (`category` rather than `pass`); the resolution log and address-reviews emit are otherwise indistinguishable.

#### Scenario: Auto-detect content sniff routes to correct parser

- **WHEN** the command runs with `--from-file <path>` and the file's content needs format routing
- **THEN** the parser inspects only the first non-whitespace token to route: leading `{` routes to JSON parser; leading `# External Review:` routes to markdown parser (the only markdown format orbit produces); anything else triggers the format-mismatch error. The sniff does NOT depend on file extension or pathname.

#### Scenario: Unsupported JSON command field

- **WHEN** `--from-file <path>` resolves to a JSON file whose top-level `command` field is anything other than `"review"` or `"audit-drift"` (e.g., `"address-reviews"`, `"apply"`, `"archive"`, `"propose"`, `"explore"`, `"new"`, `"continue"`, `"ff"`, `"review-external"`)
- **THEN** the parser emits a clean error message naming the supported `command` values (`"review"`, `"audit-drift"`) and the unsupported value detected, and exits without acting on any findings. The message references both supported format reference files so the user can self-diagnose. Notably, `command: "address-reviews"` is rejected to avoid recursive cycles (a walk that produces its own input would loop).

#### Scenario: Internal JSON missing `findings[]` field

- **WHEN** `--from-file <path>` resolves to a JSON file with a supported `command` (`"review"` or `"audit-drift"`) AND valid JSON parse AND no `findings` field at the top level (or `findings` present but not an array)
- **THEN** the parser treats this as a malformed-input parse error — emits a clean error message naming the missing/malformed `findings[]` requirement and references `references/internal-findings-format.md` for the expected shape; exits without acting

#### Scenario: Internal JSON with empty `findings[]`

- **WHEN** `--from-file <path>` resolves to a JSON file with a supported `command` (`"review"` or `"audit-drift"`) AND `findings: []` (empty array — the source command ran cleanly and surfaced no findings)
- **THEN** the parser succeeds with zero virtual markers; the resolution log reports an empty walk (✓ Resolved: 0, ⚠ Stale: 0, ⏸ Deferred: 0, ✗ Escalated: 0) with a one-line note `Source JSON had no findings to walk; resolution log is informational.` and exits cleanly. This is NOT an error — a clean review or clean audit-drift run with `findings: []` is the expected state for an all-clear pass.

#### Scenario: Malformed input — neither format matches

- **WHEN** the `--from-file` path resolves to a file whose first non-whitespace prefix is neither `{` nor `# External Review:`
- **THEN** the command emits a clean error message naming both supported formats (referencing `references/external-findings-format.md` and `references/internal-findings-format.md`) plus the observed leading-content snippet, and exits without acting on partial input

#### Scenario: Malformed input — JSON parse failure

- **WHEN** the `--from-file` path looks like JSON (leading `{`) but fails to parse as valid JSON
- **THEN** the command emits a clean parse-error message naming the file and the JSON parse-error position (best-effort), and exits without acting on partial input. The user fixes the file and re-runs.

#### Scenario: Malformed input — markdown missing required sections

- **WHEN** the `--from-file` path looks like markdown (leading `# External Review:`) but is missing required sections OR has broken field labels OR otherwise can't be parsed cleanly per the external-findings parser contract
- **THEN** the command reports a parse error with format guidance (referencing `references/external-findings-format.md`) and exits without acting on partial input

### Requirement: Discover → triage → walk → ripple flag → report lifecycle

The system SHALL execute five lifecycle stages: (1) discover markers (or load from file), (2) triage with user (numbered list, optional scoping), (3) walk each sequentially with pushback discipline, (4) ripple flag (list affected related files without auto-cascading), (5) report a resolution log.

#### Scenario: Triage scoping

- **WHEN** markers are discovered and presented as a numbered list
- **THEN** the user can specify subset scoping (e.g., "just 1-3") and only those markers are walked

### Requirement: Pushback discipline applied per marker

The system SHALL verify each marker against current state before fixing — regardless of whether the marker is inline (grep-found), markdown-virtual (`--from-file` external), or JSON-virtual (`--from-file` internal review or audit-drift JSON).

#### Scenario: Already fixed at HEAD

- **WHEN** the marker's claim references something already fixed in the current state (verified via `grep`, `git log`, or file inspection)
- **THEN** the marker is classified as "stale, suppressed"; current state and evidence (commit hash or grep result) are reported; the marker is removed without further edit

#### Scenario: Still applies

- **WHEN** the marker's claim is still valid against current state
- **THEN** the resolution proceeds to classification

#### Scenario: Pushback decision procedure

- **WHEN** the AI verifies a marker against current state
- **THEN** it follows this procedure in order: (1) identify the marker's referenced symbol, name, or concept; (2) `grep -rn` for current presence in expected locations (the file the marker is in, related files, baseline specs); (3) if the symbol is absent where the marker expects it, run `git log -S "<symbol>" --since=<reasonable-window>` (default: since the marker file was last modified) to confirm intentional removal; (4) read the relevant file's current content; (5) compare to the marker's claim and decide: still applies / already fixed / partially applies; (6) on already-fixed, report the commit hash or current-content evidence as part of suppression

#### Scenario: Internal JSON virtual markers receive fresh pushback

- **WHEN** the command ingests an internal review JSON OR audit-drift JSON via `--from-file` and walks its `findings[]` entries as virtual markers
- **THEN** each entry receives fresh pushback against current state — the source command (`/opsx:review` or `/opsx:audit-drift`) that produced the JSON already filtered findings into its own `stale_suppressed[]` array at source-run time, BUT pushback is re-applied at address-reviews time because state may have changed between the source run and the resolve (e.g., user fixed an issue ad-hoc; another commit landed). The lifecycle does NOT skip pushback for any JSON-virtual marker regardless of source `command`.

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
