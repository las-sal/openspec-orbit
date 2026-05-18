## ADDED Requirements

### Requirement: Review-system command available

The system SHALL expose a `/opsx:review-system` command that runs an editorial review of the whole-product state after a change has been applied.

#### Scenario: Invoke with change name

- **WHEN** the user invokes `/opsx:review-system <change-name>` and the change has been applied (code generated)
- **THEN** the command reads the change's artifacts, baseline specs, and the codebase, and runs the review passes

#### Scenario: Invoke without change name

- **WHEN** the user invokes `/opsx:review-system` with no argument
- **THEN** the command runs `openspec list --json`, presents the relevant changes via `AskUserQuestion`, and uses the user's selection

### Requirement: Pass 0 delegates to upstream verify-change

The system SHALL invoke upstream `/opsx:verify-change` as Pass 0 and fold its findings into the unified report.

#### Scenario: verify-change runs first

- **WHEN** `--skip-verify` is not specified
- **THEN** Pass 0 invokes `/opsx:verify-change <change-name>` (via the upstream `openspec-verify-change` skill) and incorporates its CRITICAL/WARNING/SUGGESTION findings into the review-system report under the appropriate scorecard dimensions

#### Scenario: `--skip-verify` flag

- **WHEN** the command runs with `--skip-verify`
- **THEN** Pass 0 is skipped with a note "verify-change skipped per --skip-verify flag" and Passes 1–6 execute

### Requirement: Six system-wide passes executed

The system SHALL execute 6 system-wide passes in addition to Pass 0: (1) Baseline Compliance, (2) Cohesion, (3) Surface Walk, (4) Perspective Reviews, (5) Critical-Path Scan, (6) Drift / Residue.

#### Scenario: Default depth runs all passes

- **WHEN** the command runs with default `--full` depth
- **THEN** Passes 0–6 all execute (subject to `--skip-verify`) and each produces findings (possibly zero)

#### Scenario: `--fast` runs cheap subset only

- **WHEN** the command runs with `--fast`
- **THEN** Pass 0 (verify-change), Pass 1 (Baseline Compliance), and Pass 6 (Drift/Residue) execute; Passes 2–5 are reported as skipped

### Requirement: Pass 1 — Baseline Compliance

The system SHALL check the change's code against every requirement in archived `openspec/specs/*/spec.md`, not just the change's deltas.

#### Scenario: Baseline regression found

- **WHEN** Pass 1 walks an archived requirement and finds the change's code violates it
- **THEN** a CRITICAL finding is reported with the baseline requirement's file:line and a recommendation to either add a delta updating the requirement or fix the implementation

#### Scenario: Baseline empty (new project)

- **WHEN** Pass 1 runs and `openspec/specs/` is empty
- **THEN** Pass 1 trivially passes with a note "no baseline to check against"

### Requirement: Pass 2 — Cohesion walk

The system SHALL identify files the change touched and walk callers/dependents not in the change's task list for signature drift, semantic shifts, or side effects.

#### Scenario: Caller signature drift detected

- **WHEN** Pass 2 finds a caller that passes a now-mismatched type or invokes a renamed method
- **THEN** a WARNING (or CRITICAL if clearly broken) finding is reported with the caller's file:line and a recommendation

### Requirement: Pass 3 — Surface walk derives surfaces from specs

The system SHALL enumerate surfaces from `openspec/specs/<capability>/spec.md` (capabilities ARE surfaces) and check each entry remains coherent after the change.

#### Scenario: Surface entry inconsistent after change

- **WHEN** Pass 3 finds a CLI command, MCP tool, HTTP endpoint, etc. whose documented behavior diverges from current code
- **THEN** a WARNING finding is reported with the surface's file:line and a recommendation

#### Scenario: Surface removed unintentionally

- **WHEN** Pass 3 finds a documented surface that no longer exists in code, and the proposal does not mark it as removed
- **THEN** a CRITICAL finding is reported

### Requirement: Pass 4 — Perspective reviews from lenses

The system SHALL read `openspec/lenses/perspectives.md` and, for each named perspective, simulate typical call patterns to validate from the caller's POV.

#### Scenario: Perspective produces awkward interaction

- **WHEN** Pass 4 simulates a registered perspective's call pattern and finds the calling surface awkward, inconsistent, or surprising from that POV
- **THEN** a SUGGESTION or WARNING finding is reported with the perspective's name and the surface's file:line

#### Scenario: No perspectives defined

- **WHEN** Pass 4 runs but `openspec/lenses/perspectives.md` is empty or absent
- **THEN** Pass 4 is skipped with a note "no perspectives defined; skip Pass 4"

### Requirement: Pass 5 — Critical-path scan from lenses

The system SHALL read `openspec/lenses/critical-paths.md` and, for each named flow, walk the code end-to-end checking for breakage, regression, or drift.

#### Scenario: Critical path broken

- **WHEN** Pass 5 walks a critical path and finds it no longer functions
- **THEN** a CRITICAL finding is reported with the path's name and the breakage location

#### Scenario: No critical paths defined

- **WHEN** Pass 5 runs but `openspec/lenses/critical-paths.md` is empty or absent
- **THEN** Pass 5 is skipped with a note "no critical paths defined; skip Pass 5"

### Requirement: Pass 6 — Drift / Residue via audit-drift

The system SHALL invoke `/opsx:audit-drift` as a library function and fold its findings into the review-system report.

#### Scenario: audit-drift surfaces residue

- **WHEN** Pass 6 runs and audit-drift finds vocabulary residue, lens staleness, doc inconsistency, or archive coherence issues
- **THEN** those findings are included in the review-system report under the Coherence dimension

### Requirement: Findings rolled into 3-dimension scorecard

The system SHALL roll Passes 0–6 into the standard 3-dimension scorecard.

#### Scenario: Roll-up mapping

- **WHEN** findings are reported
- **THEN** Pass 0 contributes to all three dimensions per verify-change's own mapping (Completeness: task-completion + spec-coverage findings; Correctness: requirement-implementation-mapping + scenario-coverage findings; Coherence: design-adherence + code-pattern-consistency findings); Pass 1 contributes to Correctness; Pass 2 contributes to Correctness; Pass 3 contributes to Coherence; Pass 4 contributes to Coherence; Pass 5 contributes to both Completeness and Correctness; Pass 6 contributes to Coherence

### Requirement: Final assessment phrasings (gate is /opsx:archive)

The system SHALL emit one of three stock final-assessment phrasings, with the gate text referencing `/opsx:archive`.

#### Scenario: At least one CRITICAL

- **WHEN** the report contains one or more CRITICAL findings
- **THEN** the final assessment reads `X critical issue(s) found. Fix before /opsx:archive.`

#### Scenario: No CRITICAL, only WARNING or SUGGESTION

- **WHEN** the report contains zero CRITICAL findings but at least one WARNING or SUGGESTION
- **THEN** the final assessment reads `No critical issues. Y warning(s) to consider. Ready to archive (with noted improvements).`

#### Scenario: All clear

- **WHEN** the report contains no findings of any severity
- **THEN** the final assessment reads `All checks passed. Ready to archive.`

### Requirement: Flag family parity with review-proposal

The system SHALL support the same flag family as `/opsx:review-proposal`: `--fast` / `--full` / `--thorough`, `--parallel`, `--focus`, `--fresh`, `--strict`. Plus `--skip-verify` specific to this command.

#### Scenario: `--parallel` spawns subagents for heavy passes

- **WHEN** the command runs with `--parallel`
- **THEN** Passes 2, 3, 4, and 5 execute concurrently in separate subagents and their findings are merged into the unified report

#### Scenario: `--focus hotpath`

- **WHEN** the command runs with `--focus hotpath`
- **THEN** Pass 5 (Critical-Path Scan) runs with elevated rigor; all other passes still execute

### Requirement: Pushback discipline on stale findings

The system SHALL verify each finding against current state before reporting.

#### Scenario: Stale finding suppressed

- **WHEN** a pass would produce a finding for an issue already resolved in current state
- **THEN** the finding is suppressed with an explanatory note and not included in the user-facing report

### Requirement: Internal run summary persisted

The system SHALL write a JSON run summary to `openspec/changes/<change-name>/.orbit-runs/review-system-<TS>.json` after each invocation.

#### Scenario: Summary written after run

- **WHEN** the command completes
- **THEN** a JSON file is created at `.orbit-runs/review-system-<ISO-timestamp>.json` containing the command name, timestamp, change name, iteration number, findings summary, and finding titles

### Requirement: Iteration note when prior runs exist

The system SHALL include a one-sentence iteration note in the report when prior `review-system-<TS>.json` files exist for the same change.

#### Scenario: Subsequent run on same change

- **WHEN** the command runs on a change that has at least one prior `review-system-<TS>.json` in `.orbit-runs/`
- **THEN** the report includes a note such as "Note: N of these findings appeared in the last run on <date>. M new this run."
