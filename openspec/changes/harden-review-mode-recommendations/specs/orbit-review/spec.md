## MODIFIED Requirements

### Requirement: Final assessment phrasings depend on mode

The system SHALL emit a final-assessment line in the report using one of the stock phrasings below, with gate text varying by mode. System-mode "no CRITICAL" cases additionally vary on the external-review state for the change — orbit-review SHALL inspect both the most recent `external-system-*.md` file's content (for findings presence) AND any `address-reviews-*.json` whose `source_path` references that external file (for resolution status), to determine which of five convergence states applies: (1) no prior external, (2) external clean, (3) external resolved via address-reviews, (4) external present with unresolved findings, (5) external stale relative to artifacts.

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

#### Scenario: System mode — no CRITICAL, no prior external system review

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND no `external-system-*.md` file exists in the change's `.orbit-runs/` directory
- **THEN** the final assessment recommends external review before archive, citing the empirical evidence. Exact stock phrasing for "all clear" sub-case: `All checks passed. Recommend /opsx:review-external for fresh-context cross-check before archive (per orbit-conventions Review mode decision framework; in-context system review missed 3 of 3 real bugs in bootstrap-orbit-status-cli's first archived cycle). Or /opsx:review --fresh for a lighter pass (same model, fresh context). Proceed directly to /opsx:archive if accepting that risk.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. Recommend /opsx:review-external for fresh-context cross-check before archive (per orbit-conventions Review mode decision framework). Or /opsx:review --fresh for a lighter pass. Proceed to /opsx:archive (with noted improvements) if accepting the in-context-only risk.`

#### Scenario: System mode — no CRITICAL, external content is clean (Path A convergence)

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND the most recent `external-system-*.md` file in the change's `.orbit-runs/` directory has zero findings across all severities (parsed as `## CRITICAL`, `## WARNING`, and `## SUGGESTION` sections each containing only `None.` and no `### <title>` entries; the external-review prompt template's documented convention for a clean external) AND no later artifact-changing event (latest `apply-*.json` or `address-reviews-*.json` filename `<TS>` token) is newer than the external file's filename token
- **THEN** the final assessment treats the external as converged via clean content. Exact stock phrasing for "all clear" sub-case: `All checks passed. External system review (clean) converged. Ready to archive.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. External system review (clean) converged. Ready to archive (with noted improvements).`

#### Scenario: System mode — no CRITICAL, external resolved via address-reviews (Path B convergence)

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND the most recent `external-system-*.md` file exists AND the most recent `address-reviews-*.json` whose `source_path` field references that external file has `resolution_summary.deferred == 0` AND `resolution_summary.escalated == 0` (all findings either resolved, stale-suppressed, or out-of-scope) AND no later artifact-changing event (latest `apply-*.json` or any later `address-reviews-*.json` for OTHER inputs) is newer than the address-reviews JSON's filename token
- **THEN** the final assessment treats the external as converged via resolution. Exact stock phrasing for "all clear" sub-case: `All checks passed. External system review findings resolved via /opsx:address-reviews. Ready to archive.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. External system review findings resolved via /opsx:address-reviews. Ready to archive (with noted improvements).`

#### Scenario: System mode — no CRITICAL, external present with unresolved findings

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND the most recent `external-system-*.md` has one or more `### <title>` entries under any severity (i.e., NOT clean per Path A) AND EITHER no `address-reviews-*.json` exists with `source_path` referencing that external file OR the most recent such address-reviews JSON shows `resolution_summary.deferred > 0` OR `resolution_summary.escalated > 0`
- **THEN** the final assessment recommends address-reviews next, not archive. Exact stock phrasing for "all clear" sub-case: `All checks passed against current state, but the prior /opsx:review-external has unresolved findings. Run /opsx:address-reviews --from-file <external-path> to walk them before archive. Or accept the unresolved-external risk and proceed to /opsx:archive.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. Prior /opsx:review-external has unresolved findings. Run /opsx:address-reviews --from-file <external-path> to walk them before archive.`

#### Scenario: System mode — no CRITICAL, external stale relative to artifact changes

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND a `external-system-*.md` file exists AND an `apply-*.json` exists with a filename `<TS>` token LATER than the external file's filename token (i.e., implementation changed after the external review evaluated the artifacts)
- **THEN** the final assessment recommends re-running external because the prior external evaluation no longer applies to the current artifact state. Exact stock phrasing for "all clear" sub-case: `All checks passed against current state, but the prior /opsx:review-external is older than the most recent apply step (artifacts changed since external evaluated). Re-run /opsx:review-external to validate against current product state before archive. Or /opsx:review --fresh for a lighter re-check.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. Prior /opsx:review-external is older than the most recent apply step. Re-run /opsx:review-external to validate against current product state. Or /opsx:review --fresh for a lighter re-check.`

#### Scenario: Convergence-state precedence when multiple states apply

- **WHEN** the iteration-aware logic encounters a change-state where multiple convergence scenarios could match (e.g., external is clean AND artifacts changed; OR external has unresolved findings AND artifacts also changed since)
- **THEN** orbit-review SHALL apply the precedence order: (1) `external stale relative to artifact changes` takes precedence over Path A clean / Path B resolved / unresolved-findings — if artifacts changed after the external, the external's evaluation is no longer trustworthy regardless of its content or resolution. (2) `external present with unresolved findings` takes precedence over Path A clean / Path B resolved — unresolved findings are a stronger signal than a stale-but-resolved external (unresolved-still-stale gets recommended-to-re-run via the stale rule above, then unresolved via address-reviews after re-run). (3) Path A clean and Path B resolved are mutually exclusive (either the external is clean OR it had findings; can't be both); precedence between them doesn't apply.

#### Scenario: Iteration-aware logic does NOT block archive

- **WHEN** the user invokes `/opsx:archive` after a system-mode review whose final-assessment recommended external review, address-reviews, or re-running external
- **THEN** orbit's archive flow proceeds normally (per orbit-archive-modifications behavior); the recommendation is advisory, NOT a gate. The user retains the choice to skip the recommended action; orbit's role is to surface the recommendation, not enforce it. (The decision to NOT add a pre-archive external-review-presence check is intentional per design.md Non-Goals.)

#### Scenario: Edge-case assumptions for the iteration-aware logic

- **WHEN** the iteration-aware logic encounters edge cases not explicitly enumerated in the prior scenarios (corrupted/empty `external-system-*.md`; malformed `address-reviews-*.json` whose JSON fields cannot be read; multiple matching files of the same type; filename-token parse failures; address-reviews JSON whose `source_path` references a non-existent external file)
- **THEN** the v1 behavior follows these documented assumptions: (a) **Multiple matching files** — when multiple matching files of the same type exist (multiple `external-system-*.md`, multiple `address-reviews-*.json`, multiple `apply-*.json`), the most-recent filename `<TS>` token determines comparison (lexical sort works because the token format is `YYYY-MM-DDTHH-MM-SSZ` — ISO-8601 with colons replaced; later timestamps sort lexically later). (b) **Unparseable filename token** — a file whose `<TS>` token cannot be parsed is treated as absent for comparison purposes. (c) **External markdown parse failure** — if the external file cannot be parsed for the Path A "clean content" check (e.g., severity headers missing, malformed markdown), it is treated as NOT-clean (defaulting to the unresolved-findings or stale branch); Path A requires confident parse success. (d) **address-reviews JSON parse failure** — if the JSON cannot be read or `resolution_summary` is missing required fields, it is treated as if no address-reviews has run for that external (defaulting to the unresolved-findings branch). (e) **Dangling source_path** — an `address-reviews-*.json` whose `source_path` references a non-existent file is ignored for convergence detection. Future drift-audit Cat 4 or downstream tools MAY add validation for these cases; orbit-review's v1 stays simple.

#### Scenario: Iteration-aware logic does NOT inspect apply-chunk emit chain

- **WHEN** determining "artifacts changed after external" for the stale-detection logic
- **THEN** orbit-review SHALL inspect `apply-*.json` filename tokens (the apply skill emits one per chunk-end per orbit-conventions). The CURRENT v1 check considers ANY `apply-*.json` later than the external file as evidence of artifact change, without distinguishing between mid-implementation chunks (where the external was correctly run against partial state) and post-final-chunk applies. This is intentional v1 simplification — orbit doesn't currently track which chunks were applied when; the assumption is that ANY apply-chunk after external is meaningful enough to warrant re-running external. Future enhancement could refine this against the change's task-completion graph.

#### Scenario: Recommendation prose is auditable by drift-audit

- **WHEN** the stock-phrasings prose drifts from the `Review mode decision framework` requirement's criteria, OR the cited empirical evidence reference (`bootstrap-orbit-status-cli` 3-of-3) becomes inaccurate
- **THEN** `/opsx:audit-drift` Category 3 (cross-doc consistency) SHALL surface the drift on next run. The recommendation prose's empirical citation is treated as documentation that can rot; drift-audit catches the rot.
