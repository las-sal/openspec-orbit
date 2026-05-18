## ADDED Requirements

### Requirement: Review-proposal command available

The system SHALL expose a `/opsx:review-proposal` command that runs an editorial review of a change's pre-implementation artifacts (proposal, design, spec deltas, tasks, explore.md).

#### Scenario: Invoke with change name

- **WHEN** the user invokes `/opsx:review-proposal <change-name>` and the change directory exists at `openspec/changes/<change-name>/`
- **THEN** the command reads the change's artifacts and runs the review passes

#### Scenario: Invoke without change name

- **WHEN** the user invokes `/opsx:review-proposal` with no argument
- **THEN** the command runs `openspec list --json`, presents the active changes via `AskUserQuestion`, and uses the user's selection as the change scope

### Requirement: Nine review passes executed

The system SHALL execute 9 distinct review passes over the change artifacts: (1) Structure & Delta Integrity, (2) Internal Coherence, (3) Cross-Doc Coherence, (4) Archive Consistency, (5) Codegen Readiness, (6) Gap Hunt (generative completeness probe), (7) Drift Hunt, (8) Inline Review Marker Residue, (9) Pre-Handoff Sweep.

#### Scenario: Pass 6 Gap Hunt operational heuristic

- **WHEN** Pass 6 runs
- **THEN** for each requirement in the change's spec deltas, the AI asks (and answers in the finding when affirmative): (a) are there unstated assumptions an implementer would have to invent (e.g., "what file path?", "what default value?")? (b) are error or edge-case paths specified, not just happy paths? (c) are state transitions explicit, including invalid transitions? (d) for any "X SHALL do Y", is Y precise enough that two implementers would produce the same behavior? Findings cite the requirement's file:line and the specific gap; suggest concrete spec additions to close the gap

#### Scenario: Pass 8 marker residue detection (operational)

- **WHEN** Pass 8 runs
- **THEN** the AI greps the change directory for `@review:` markers and distinguishes two cases: (a) **actual unresolved markers** — markers in proposal/design/spec/tasks/explore.md content that represent unaddressed review notes (these are CRITICAL findings and MUST be addressed before `/opsx:apply`); (b) **documentation appearances of the marker syntax** — `@review:` text inside code blocks, examples, or scenarios that document the marker convention itself (these are NOT findings). The distinction is whether the marker is inside a fenced code block / inline-code span / explicit "example" prose context (documentation) vs. sitting bare in artifact content (unresolved). When ambiguous, classify as CRITICAL and let the user decide during `/opsx:address-reviews`.

#### Scenario: Default depth runs all passes

- **WHEN** the command runs with default `--full` depth
- **THEN** all 9 passes execute sequentially and each produces findings (possibly zero) of severity CRITICAL, WARNING, or SUGGESTION

#### Scenario: `--fast` runs cheap subset only

- **WHEN** the command runs with `--fast`
- **THEN** only Passes 1 (Structure & Delta), 7 (Drift Hunt), and 8 (Inline Review Marker Residue) execute, and the remaining passes are reported as skipped

### Requirement: Findings rolled into 3-dimension scorecard

The system SHALL roll the 9 passes' findings into a 3-dimension scorecard (Completeness / Correctness / Coherence) per the orbit-conventions reporting standard.

#### Scenario: Scorecard mapping

- **WHEN** findings are reported
- **THEN** Pass 1, 5, and 6 contribute to Completeness; Pass 2 and 4 contribute to Correctness; Pass 3, 7, 8, and 9 contribute to Coherence

#### Scenario: Severity ladder applied

- **WHEN** any pass produces a finding
- **THEN** the finding is tagged CRITICAL, WARNING, or SUGGESTION with bias toward lower severity when uncertain; includes a file:line reference and an actionable recommendation

### Requirement: Final assessment phrasings

The system SHALL emit a final-assessment line in the report using one of three stock phrasings.

#### Scenario: At least one CRITICAL

- **WHEN** the report contains one or more CRITICAL findings
- **THEN** the final assessment reads `X critical issue(s) found. Fix before /opsx:apply.`

#### Scenario: No CRITICAL, only WARNING or SUGGESTION

- **WHEN** the report contains zero CRITICAL findings but at least one WARNING or SUGGESTION
- **THEN** the final assessment reads `No critical issues. Y warning(s) to consider. Ready to apply (with noted improvements).`

#### Scenario: All clear

- **WHEN** the report contains no findings of any severity
- **THEN** the final assessment reads `All checks passed. Ready to apply.`

### Requirement: Special-case lenses supported via `--focus`

The system SHALL accept a `--focus <lens>` flag that adds emphasis to specific passes without skipping others.

#### Scenario: `--focus rename`

- **WHEN** the user invokes the command with `--focus rename`
- **THEN** Pass 7 (Drift Hunt), Pass 3 (Cross-Doc), and Pass 4 (Archive Consistency) run with elevated rigor; all other passes still execute

#### Scenario: `--focus flip`

- **WHEN** the user invokes the command with `--focus flip`
- **THEN** Pass 2 (Internal Coherence) emphasizes directionality and Pass 5 (Codegen Readiness) emphasizes that surfaces are specified for both directions

### Requirement: `--mark` flag drops review markers

The system SHALL support a `--mark` flag that, after the review report is generated, writes `@review: <text>` markers into the relevant artifacts based on the findings.

#### Scenario: `--mark` enabled

- **WHEN** the command runs with `--mark`
- **THEN** for each finding, a corresponding `@review: <finding text>` marker is inserted at the finding's file:line, enabling unified resolution via `/opsx:address-reviews`

#### Scenario: `--mark` disabled (default)

- **WHEN** the command runs without `--mark`
- **THEN** findings are reported in chat only; no markers are written to any file

### Requirement: Parallel execution via `--parallel`

The system SHALL support a `--parallel` flag that spawns subagents for the heavier passes (Internal Coherence, Archive Consistency, Gap Hunt) to run concurrently.

#### Scenario: `--parallel` enabled

- **WHEN** the command runs with `--parallel`
- **THEN** Passes 2, 4, and 6 execute concurrently in separate subagents and their findings are merged into the unified report

### Requirement: Pushback discipline on stale findings

The system SHALL verify each finding against current state before reporting it.

#### Scenario: Stale finding suppressed

- **WHEN** an internal pass would produce a finding that no longer applies (the issue was fixed since the finding's basis was captured)
- **THEN** the finding is suppressed with an explanatory note ("stale finding suppressed: <evidence>") and not included in the user-facing report

### Requirement: Internal run summary persisted

The system SHALL write a JSON run summary to `openspec/changes/<change-name>/.orbit-runs/review-proposal-<TS>.json` after each invocation.

#### Scenario: Summary written after run

- **WHEN** the command completes
- **THEN** a JSON file is created at `.orbit-runs/review-proposal-<ISO-timestamp>.json` containing the command name, timestamp, change name, iteration number, findings summary (counts by severity and by pass), and finding titles

### Requirement: Iteration note when prior runs exist

The system SHALL include a one-sentence iteration note in the report when prior run summaries exist for the same change.

#### Scenario: Second run on same change

- **WHEN** the command runs on a change that has at least one prior `review-proposal-<TS>.json` in `.orbit-runs/`
- **THEN** the report includes a note such as "Note: N of these findings appeared in the last run on <date>. M new this run."

#### Scenario: First run on a change

- **WHEN** the command runs on a change with no prior `review-proposal-<TS>.json` in `.orbit-runs/`
- **THEN** the report omits the iteration note, or notes "First review-proposal run for this change."

### Requirement: Graceful degradation on missing inputs

The system SHALL degrade gracefully when expected inputs are absent.

#### Scenario: No `openspec/specs/` baseline

- **WHEN** Pass 4 (Archive Consistency) would run but `openspec/specs/` is empty
- **THEN** Pass 4 is skipped with a note "no baseline to check against" and other passes proceed normally

#### Scenario: No `openspec/lenses/`

- **WHEN** Pass 3 (Cross-Doc) would reference lenses content but `openspec/lenses/` is empty or absent
- **THEN** lens-driven cross-checks within Pass 3 are skipped with a note; the rest of Pass 3 runs normally
