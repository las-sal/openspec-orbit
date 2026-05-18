# orbit-review

## Purpose

The unified `/opsx:review` command — editorial review of an OpenSpec change in either proposal mode (pre-apply, 9 passes over artifacts) or system mode (post-apply, `verify-change` as Pass 0 + 6 system-wide passes). Both modes share the 3-dimension scorecard (Completeness / Correctness / Coherence), the severity ladder (CRITICAL / WARNING / SUGGESTION), pushback discipline, and `.orbit-runs/` persistence. Generates findings but does not resolve them; resolution flows through `/opsx:address-reviews`.

## Requirements

### Requirement: Review command available with `--as` mode flag

The system SHALL expose a single `/opsx:review` command that runs an editorial review of an OpenSpec change. The `--as proposal` flag reviews pre-implementation artifacts (proposal/design/specs/tasks); the `--as system` flag reviews the whole product state after a change has been applied.

#### Scenario: Invoke with change name and explicit mode

- **WHEN** the user invokes `/opsx:review <change-name> --as proposal` or `/opsx:review <change-name> --as system`
- **THEN** the command reads the change's artifacts and runs the appropriate set of passes for that mode

#### Scenario: Invoke without change name

- **WHEN** the user invokes `/opsx:review` with no argument
- **THEN** the command runs `openspec list --json`, presents the active changes via `AskUserQuestion`, and uses the user's selection as the change scope

#### Scenario: Mode inference when `--as` omitted

- **WHEN** the user invokes `/opsx:review <change-name>` without `--as`
- **THEN** the command infers the mode from `tasks.md` state: unchecked task boxes → `proposal`; all tasks checked and code exists → `system`; ambiguous → prompt the user via `AskUserQuestion` to pick; the inferred mode is shown in the report header

### Requirement: Proposal mode runs nine passes

The system SHALL execute 9 distinct review passes in proposal mode: (1) Structure & Delta Integrity, (2) Internal Coherence, (3) Cross-Doc Coherence, (4) Archive Consistency, (5) Codegen Readiness, (6) Gap Hunt (generative completeness probe), (7) Drift Hunt, (8) Inline Review Marker Residue, (9) Pre-Handoff Sweep.

#### Scenario: Proposal mode default depth runs all passes

- **WHEN** the command runs as `--as proposal` with default `--full` depth
- **THEN** all 9 passes execute sequentially and each produces findings (possibly zero) of severity CRITICAL, WARNING, or SUGGESTION

#### Scenario: Proposal mode `--fast` runs cheap subset only

- **WHEN** the command runs as `--as proposal` with `--fast`
- **THEN** only Passes 1 (Structure & Delta), 7 (Drift Hunt), and 8 (Inline Review Marker Residue) execute, and the remaining passes are reported as skipped

#### Scenario: Pass 6 Gap Hunt operational heuristic (proposal mode)

- **WHEN** Pass 6 runs
- **THEN** for each requirement in the change's spec deltas, the AI asks (and answers in the finding when affirmative): (a) are there unstated assumptions an implementer would have to invent (e.g., "what file path?", "what default value?")? (b) are error or edge-case paths specified, not just happy paths? (c) are state transitions explicit, including invalid transitions? (d) for any "X SHALL do Y", is Y precise enough that two implementers would produce the same behavior? Findings cite the requirement's file:line and the specific gap; suggest concrete spec additions to close the gap

#### Scenario: Pass 8 marker residue detection (operational)

- **WHEN** Pass 8 runs
- **THEN** the AI greps the change directory for `@review:` markers and distinguishes two cases: (a) **actual unresolved markers** — markers in proposal/design/spec/tasks/explore.md content that represent unaddressed review notes (these are CRITICAL findings and MUST be addressed before applying); (b) **documentation appearances of the marker syntax** — `@review:` text inside code blocks, examples, or scenarios that document the marker convention itself (these are NOT findings). The distinction is whether the marker is inside a fenced code block / inline-code span / explicit "example" prose context (documentation) vs. sitting bare in artifact content (unresolved). When ambiguous, classify as CRITICAL and let the user decide during `/opsx:address-reviews`.

### Requirement: System mode runs verify-change + six system-wide passes

The system SHALL execute 7 review passes in system mode: Pass 0 delegates to upstream `verify-change`; Passes 1–6 are orbit additions covering Baseline Compliance, Cohesion, Surface Walk, Perspective Reviews, Critical-Path Scan, Drift/Residue.

#### Scenario: Pass 0 delegates to upstream verify-change

- **WHEN** the command runs as `--as system` and `--skip-verify` is not specified
- **THEN** Pass 0 invokes `/opsx:verify <change-name>` (via the upstream `openspec-verify-change` skill) and folds its CRITICAL/WARNING/SUGGESTION findings into the report under the appropriate scorecard dimensions

#### Scenario: System mode default depth runs all passes

- **WHEN** the command runs as `--as system` with default `--full` depth
- **THEN** Passes 0–6 all execute (subject to `--skip-verify`) and each produces findings (possibly zero)

#### Scenario: System mode `--fast` runs cheap subset only

- **WHEN** the command runs as `--as system` with `--fast`
- **THEN** Pass 0 (verify-change), Pass 1 (Baseline Compliance), and Pass 6 (Drift/Residue) execute; Passes 2–5 are reported as skipped

#### Scenario: Pass 1 — Baseline Compliance

- **WHEN** Pass 1 runs in system mode
- **THEN** the command reads every requirement in archived `openspec/specs/*/spec.md` (not just the change's deltas) and reports CRITICAL when a baseline requirement appears violated; WARNING when a baseline scenario looks regressed

#### Scenario: Pass 2 — Cohesion walk

- **WHEN** Pass 2 runs in system mode
- **THEN** the command identifies files the change touched (from `git diff` and the tasks list) and walks callers/dependents not in the change's tasks for signature drift, semantic shifts, or side effects; flags WARNING for caller-side contract drift, CRITICAL when a downstream is clearly broken

#### Scenario: Pass 3 — Surface walk derives surfaces from specs

- **WHEN** Pass 3 runs in system mode
- **THEN** the command enumerates surfaces from `openspec/specs/<capability>/spec.md` (capabilities ARE surfaces) and checks each entry remains coherent after the change; flags WARNING for inconsistencies, CRITICAL when a surface was removed unintentionally

#### Scenario: Pass 4 — Perspective reviews from lenses

- **WHEN** Pass 4 runs in system mode and `openspec/lenses/perspectives.md` exists
- **THEN** for each named perspective, the command simulates typical call patterns; flags SUGGESTION/WARNING when interactions are awkward, inconsistent, or surprising from a registered caller's POV

#### Scenario: Pass 4 — No perspectives defined

- **WHEN** Pass 4 runs in system mode but `openspec/lenses/perspectives.md` is empty or absent
- **THEN** Pass 4 is skipped with a note "no perspectives defined; skip Pass 4"

#### Scenario: Pass 5 — Critical-path scan from lenses

- **WHEN** Pass 5 runs in system mode and `openspec/lenses/critical-paths.md` exists
- **THEN** for each named flow, the command walks the code end-to-end checking for breakage, regression, or drift; flags CRITICAL for path breakage, WARNING for regression risk

#### Scenario: Pass 5 — No critical paths defined

- **WHEN** Pass 5 runs in system mode but `openspec/lenses/critical-paths.md` is empty or absent
- **THEN** Pass 5 is skipped with a note "no critical paths defined; skip Pass 5"

#### Scenario: Pass 6 — Drift / Residue via audit-drift

- **WHEN** Pass 6 runs in system mode
- **THEN** the command invokes `/opsx:audit-drift` as a library function and folds its findings into the report under the Coherence dimension

### Requirement: Findings rolled into a shared 3-dimension scorecard

The system SHALL roll all passes (mode-specific) into a 3-dimension scorecard (Completeness / Correctness / Coherence) per the orbit-conventions reporting standard, regardless of mode.

#### Scenario: Proposal-mode scorecard mapping

- **WHEN** the command runs in proposal mode and findings are reported
- **THEN** Pass 1 (Structure & Delta), Pass 5 (Codegen Readiness), and Pass 6 (Gap Hunt) contribute to Completeness; Pass 2 (Internal Coherence) and Pass 4 (Archive Consistency) contribute to Correctness; Pass 3 (Cross-Doc), Pass 7 (Drift Hunt), Pass 8 (Inline Review Marker Residue), and Pass 9 (Pre-Handoff Sweep) contribute to Coherence

#### Scenario: System-mode scorecard mapping

- **WHEN** the command runs in system mode and findings are reported
- **THEN** Pass 0 contributes to all three dimensions per verify-change's own mapping (Completeness: task-completion + spec-coverage findings; Correctness: requirement-implementation-mapping + scenario-coverage findings; Coherence: design-adherence + code-pattern-consistency findings); Pass 1 (Baseline Compliance) contributes to Correctness; Pass 2 (Cohesion) contributes to Correctness; Pass 3 (Surface Walk) contributes to Coherence; Pass 4 (Perspective Reviews) contributes to Coherence; Pass 5 (Critical-Path Scan) contributes to both Completeness and Correctness; Pass 6 (Drift/Residue) contributes to Coherence

#### Scenario: Severity ladder applied

- **WHEN** any pass produces a finding
- **THEN** the finding is tagged CRITICAL, WARNING, or SUGGESTION with bias toward lower severity when uncertain; includes a file:line reference and an actionable recommendation

### Requirement: Final assessment phrasings depend on mode

The system SHALL emit a final-assessment line in the report using one of three stock phrasings, with the gate text varying by mode.

#### Scenario: Proposal mode — at least one CRITICAL

- **WHEN** running as `--as proposal` and the report contains one or more CRITICAL findings
- **THEN** the final assessment reads `X critical issue(s) found. Fix before /opsx:apply.`

#### Scenario: Proposal mode — no CRITICAL, only WARNING or SUGGESTION

- **WHEN** running as `--as proposal` and the report contains zero CRITICAL findings but at least one WARNING or SUGGESTION
- **THEN** the final assessment reads `No critical issues. Y warning(s) to consider. Ready to apply (with noted improvements).`

#### Scenario: Proposal mode — all clear

- **WHEN** running as `--as proposal` and the report contains no findings of any severity
- **THEN** the final assessment reads `All checks passed. Ready to apply.`

#### Scenario: System mode — at least one CRITICAL

- **WHEN** running as `--as system` and the report contains one or more CRITICAL findings
- **THEN** the final assessment reads `X critical issue(s) found. Fix before /opsx:archive.`

#### Scenario: System mode — no CRITICAL, only WARNING or SUGGESTION

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings but at least one WARNING or SUGGESTION
- **THEN** the final assessment reads `No critical issues. Y warning(s) to consider. Ready to archive (with noted improvements).`

#### Scenario: System mode — all clear

- **WHEN** running as `--as system` and the report contains no findings of any severity
- **THEN** the final assessment reads `All checks passed. Ready to archive.`

### Requirement: Special-case lenses supported via `--focus`

The system SHALL accept a `--focus <lens>` flag that adds emphasis to specific passes without skipping others.

#### Scenario: Proposal-mode `--focus rename`

- **WHEN** the command runs as `--as proposal --focus rename`
- **THEN** Pass 7 (Drift Hunt), Pass 3 (Cross-Doc), and Pass 4 (Archive Consistency) run with elevated rigor; all other passes still execute

#### Scenario: Proposal-mode `--focus flip`

- **WHEN** the command runs as `--as proposal --focus flip`
- **THEN** Pass 2 (Internal Coherence) emphasizes directionality and Pass 5 (Codegen Readiness) emphasizes that surfaces are specified for both directions

#### Scenario: System-mode `--focus hotpath`

- **WHEN** the command runs as `--as system --focus hotpath`
- **THEN** Pass 5 (Critical-Path Scan) runs with elevated rigor; all other passes still execute

### Requirement: `--mark` flag is proposal-mode only

The system SHALL support a `--mark` flag that, after the review report is generated, writes `@review: <text>` markers into the relevant artifacts based on the findings; this flag is meaningful only in proposal mode.

#### Scenario: `--mark` enabled in proposal mode

- **WHEN** the command runs as `--as proposal --mark`
- **THEN** for each finding, a corresponding `@review: <finding text>` marker is inserted at the finding's file:line, enabling unified resolution via `/opsx:address-reviews`

#### Scenario: `--mark` ignored in system mode (v1)

- **WHEN** the command runs as `--as system --mark`
- **THEN** the flag is accepted but produces a note `--mark` writes to source files is v2 — see issue #3; ignored for v1 system-mode runs

### Requirement: `--skip-verify` flag is system-mode only

The system SHALL support a `--skip-verify` flag in system mode to bypass Pass 0 when the user has already run `/opsx:verify` separately.

#### Scenario: `--skip-verify` flag in system mode

- **WHEN** the command runs as `--as system --skip-verify`
- **THEN** Pass 0 is skipped with a note "verify-change skipped per --skip-verify flag" and Passes 1–6 execute

#### Scenario: `--skip-verify` ignored in proposal mode

- **WHEN** the command runs as `--as proposal --skip-verify`
- **THEN** the flag is ignored (there is no Pass 0 in proposal mode); no error

### Requirement: Parallel execution via `--parallel`

The system SHALL support a `--parallel` flag that spawns subagents for heavy passes to run concurrently. Which passes are parallelized depends on mode.

#### Scenario: Proposal mode `--parallel`

- **WHEN** the command runs as `--as proposal --parallel`
- **THEN** Passes 2 (Internal Coherence), 4 (Archive Consistency), and 6 (Gap Hunt) execute concurrently in separate subagents and their findings are merged into the unified report

#### Scenario: System mode `--parallel`

- **WHEN** the command runs as `--as system --parallel`
- **THEN** Passes 2, 3, 4, and 5 execute concurrently in separate subagents and their findings are merged into the unified report

### Requirement: Pushback discipline on stale findings

The system SHALL verify each finding against current state before reporting it, regardless of mode.

#### Scenario: Stale finding suppressed

- **WHEN** an internal pass would produce a finding for an issue already resolved in current state (verified via `grep`, `git log`, or file inspection)
- **THEN** the finding is suppressed with an explanatory note ("stale finding suppressed: <evidence>") and not included in the user-facing report

### Requirement: Internal run summary persisted per mode

The system SHALL write a JSON run summary to `openspec/changes/<change-name>/.orbit-runs/review-<mode>-<TS>.json` after each invocation, where `<mode>` is `proposal` or `system`.

#### Scenario: Summary written after run

- **WHEN** the command completes
- **THEN** a JSON file is created at `.orbit-runs/review-<mode>-<ISO-timestamp>.json` containing the command name, timestamp, change name, mode, iteration number, findings summary (counts by severity and by pass), and finding titles

### Requirement: Iteration note when prior runs exist

The system SHALL include a one-sentence iteration note in the report when prior run summaries exist for the same change and mode.

#### Scenario: Subsequent run on same change in same mode

- **WHEN** the command runs and `.orbit-runs/` contains at least one prior `review-<mode>-<TS>.json` matching the current mode
- **THEN** the report includes a note such as "Note: N of these findings appeared in the last run on <date>. M new this run."

#### Scenario: First run for a mode on a change

- **WHEN** the command runs and `.orbit-runs/` contains no prior `review-<mode>-<TS>.json` matching the current mode
- **THEN** the report omits the iteration note, or notes "First proposal-mode run for this change." / "First system-mode run for this change."

### Requirement: Graceful degradation on missing inputs

The system SHALL degrade gracefully when expected inputs are absent.

#### Scenario: No `openspec/specs/` baseline (proposal mode)

- **WHEN** Pass 4 (Archive Consistency) would run in proposal mode but `openspec/specs/` is empty
- **THEN** Pass 4 is skipped with a note "no baseline to check against" and other passes proceed normally

#### Scenario: No `openspec/lenses/` (system mode)

- **WHEN** Passes 4 or 5 would run in system mode but `openspec/lenses/` is empty or absent
- **THEN** the affected passes skip with a note; Pass 3 (Surface Walk) still runs against capabilities derived from `openspec/specs/`

#### Scenario: No project context docs (proposal mode)

- **WHEN** Pass 3 (Cross-Doc Coherence) would run in proposal mode but `CLAUDE.md` / `project.md` / `*_convention.md` are absent
- **THEN** lens-driven cross-checks within Pass 3 are skipped with a note; the rest of Pass 3 runs normally
