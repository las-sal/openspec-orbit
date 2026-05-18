## ADDED Requirements

### Requirement: Audit-drift command available

The system SHALL expose a `/opsx:audit-drift` command that performs a project-wide scan for drift between captured knowledge and reality.

#### Scenario: Invoke standalone

- **WHEN** the user invokes `/opsx:audit-drift` with no arguments
- **THEN** the command scans the project for drift across the four scan categories and reports findings

#### Scenario: Invoke as library function

- **WHEN** `/opsx:review --as system` Pass 6 invokes audit-drift internally
- **THEN** audit-drift runs with the same logic and its findings are folded into the review report (system mode) under the Coherence dimension

#### Scenario: Invoke from archive pre-sweep

- **WHEN** `/opsx:archive` auto-invokes audit-drift before completing
- **THEN** audit-drift runs with the same logic and its findings are presented to the user as a pre-archive prompt

### Requirement: Four scan categories executed

The system SHALL execute four scan categories: (1) Vocabulary residue, (2) Lens staleness, (3) Cross-doc consistency, (4) Archive coherence.

#### Scenario: Default depth runs all categories

- **WHEN** the command runs with default `--full` depth
- **THEN** all four categories execute and each produces findings (possibly zero)

#### Scenario: `--fast` runs cheap subset only

- **WHEN** the command runs with `--fast`
- **THEN** Category 1 (Vocabulary residue) and Category 2 (Lens staleness) execute; Categories 3 and 4 are reported as skipped

### Requirement: Category 1 — Vocabulary residue

The system SHALL build a residue-pattern set from archived changes' deltas (RENAMED FROM symbols, REMOVED requirement names) and grep target docs for residue.

#### Scenario: Stale vocabulary in non-delta'd spec

- **WHEN** an archived change RENAMED `BridgeServer` to `HostLifecycle`, and Category 1 finds `BridgeServer` still referenced in a non-delta'd `openspec/specs/<capability>/spec.md`
- **THEN** a WARNING finding is reported with the file:line, the rename context, and a recommendation to either delta the file in a future change or apply a hotfix commit

#### Scenario: Stale vocabulary in governing doc

- **WHEN** Category 1 finds residue in `project.md` or `CLAUDE.md`
- **THEN** a CRITICAL finding is reported because governing docs are followed by future implementers

### Requirement: Category 2 — Lens staleness

The system SHALL check each entry in `openspec/lenses/` references against current state.

#### Scenario: Perspective references missing capability

- **WHEN** a perspective in `openspec/lenses/perspectives.md` references a surface `<capability>` that no longer exists under `openspec/specs/<capability>/`
- **THEN** a WARNING finding is reported with the perspective's file:line and a recommendation to update or remove the entry

#### Scenario: Critical path touchpoint unresolvable

- **WHEN** a critical path in `openspec/lenses/critical-paths.md` lists a touchpoint that references a non-existent capability or tool
- **THEN** a WARNING finding is reported

### Requirement: Category 3 — Cross-doc consistency

The system SHALL extract structured claims from `CLAUDE.md`, `openspec/project.md`, and `*_convention.md`, and check for inconsistencies across docs and against current specs.

#### Scenario: Structured claim extraction

- **WHEN** Category 3 reads a doc
- **THEN** it extracts at minimum these claim categories: (a) **named entities** — capability names, surface names, tool names, file paths mentioned; (b) **quantitative claims** — port numbers, counts, version pins, size limits; (c) **architectural assertions** — "X talks to Y", "X is the server, Y is the client", "X must be in place before Y"; (d) **rules/conventions** — naming patterns, error formats, file location conventions referenced in non-convention docs

#### Scenario: Two docs disagree on a fact

- **WHEN** Category 3 finds the same fact described differently in two docs (e.g., port number, architecture flow, naming)
- **THEN** a WARNING finding is reported with both file:line references and a recommendation to reconcile

#### Scenario: Doc-vs-spec disagreement

- **WHEN** Category 3 finds a governing doc's claim materially disagrees with a current spec
- **THEN** a CRITICAL finding is reported

### Requirement: Category 4 — Archive coherence

The system SHALL check recently archived changes (default: last 5; override with `--since <ref>`) for sync-specs propagation failures.

#### Scenario: ADDED requirement not propagated to baseline

- **WHEN** Category 4 finds an archived change with an ADDED requirement that does not appear in the corresponding `openspec/specs/<capability>/spec.md`
- **THEN** a CRITICAL finding is reported

#### Scenario: REMOVED requirement still in baseline

- **WHEN** Category 4 finds an archived change marked a requirement REMOVED but the requirement still appears in baseline
- **THEN** a CRITICAL finding is reported

### Requirement: Findings rolled into 3-dimension scorecard

The system SHALL roll the four categories into the standard 3-dimension scorecard.

#### Scenario: Mapping

- **WHEN** findings are reported
- **THEN** Category 4 contributes to Completeness; Categories 1 and 2 contribute to Correctness; Category 3 contributes to Coherence

### Requirement: Final assessment phrasings vary by invocation context

The system SHALL emit a final-assessment line whose gate text depends on how audit-drift was invoked.

#### Scenario: Standalone — at least one CRITICAL

- **WHEN** invoked standalone with at least one CRITICAL finding
- **THEN** the final assessment reads `X critical issue(s) found.` (no gate mentioned)

#### Scenario: Pre-archive — at least one CRITICAL

- **WHEN** invoked as an archive pre-sweep with at least one CRITICAL finding
- **THEN** the final assessment reads `X critical issue(s) found. Address before /opsx:archive?` (prompt, not gate)

#### Scenario: Library call from review-system

- **WHEN** invoked as a library function by review-system
- **THEN** no standalone final-assessment line is emitted; findings are folded into review-system's report

### Requirement: Flag family parity with review commands

The system SHALL support `--fast` / `--full` / `--thorough`, `--parallel`, `--focus <area>` (vocabulary / lenses / docs / archive), `--strict`, and `--since <ref>`.

#### Scenario: `--focus` limits scope

- **WHEN** the command runs with `--focus lenses`
- **THEN** only Category 2 (Lens staleness) executes; other categories are skipped with notes

#### Scenario: `--since` overrides archive window

- **WHEN** the command runs with `--since <git-ref>`
- **THEN** Category 4 scans archived changes since the specified ref instead of the default window

### Requirement: Internal run summary persisted

The system SHALL write a JSON run summary to `openspec/changes/<change-name>/.orbit-runs/audit-drift-<TS>.json` when invoked in a change context (e.g., from review-system or pre-archive); when invoked standalone with no change context, summaries are written to a project-level location (e.g., `openspec/.orbit-runs/audit-drift-<TS>.json`).

#### Scenario: Summary written after run

- **WHEN** the command completes
- **THEN** a JSON summary is created with command name, timestamp, findings summary by category and severity, and finding titles
