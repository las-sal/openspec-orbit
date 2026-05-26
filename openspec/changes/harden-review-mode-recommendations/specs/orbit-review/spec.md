## MODIFIED Requirements

### Requirement: Final assessment phrasings depend on mode

The system SHALL emit a final-assessment line in the report using one of the stock phrasings below, with gate text varying by mode. System-mode "no CRITICAL" cases additionally vary on whether external system review has run for the change yet (iteration-aware logic) — when no `external-system-*.md` file exists in the change's `.orbit-runs/`, the recommendation nudges toward external; when external has run and converged clean, the recommendation simplifies to archive-ready.

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

#### Scenario: System mode — no CRITICAL, prior external converged clean

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND an `external-system-*.md` file exists in the change's `.orbit-runs/` directory AND the most recent internal `review-system-*.json` has a filename `<TS>` token (per orbit-conventions `Internal-run JSON summary format`) LATER than the most recent `external-system-*.md`'s filename token (i.e., external ran first, then internal re-ran clean afterward)
- **THEN** the final assessment simplifies to archive-ready without the external recommendation. Exact stock phrasing for "all clear" sub-case: `All checks passed. External system review converged clean. Ready to archive.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case: `No critical issues. Y warning(s) to consider. External system review converged. Ready to archive (with noted improvements).`

#### Scenario: System mode — no CRITICAL, external present but stale

- **WHEN** running as `--as system` and the report contains zero CRITICAL findings AND an `external-system-*.md` file exists in the change's `.orbit-runs/` directory AND the most recent internal `review-system-*.json` has a filename `<TS>` token (per orbit-conventions `Internal-run JSON summary format`) EARLIER than the most recent `external-system-*.md`'s filename token (i.e., external ran, then artifacts changed without re-running external, then internal review now reports clean)
- **THEN** the final assessment recommends re-running external because the prior external evaluation may no longer apply. Exact stock phrasing for "all clear" sub-case: `All checks passed against current state, but the prior /opsx:review-external is older than the most recent artifact changes. Re-run /opsx:review-external to validate against the current product state before archive. Or /opsx:review --fresh for a lighter re-check.` Exact stock phrasing for "Only WARNING/SUGGESTION" sub-case parallels with the warning-count prefix.

#### Scenario: Iteration-aware logic does NOT block archive

- **WHEN** the user invokes `/opsx:archive` after a system-mode review whose final-assessment recommended external review
- **THEN** orbit's archive flow proceeds normally (per orbit-archive-modifications behavior); the recommendation is advisory, NOT a gate. The user retains the choice to skip external; orbit's role is to surface the recommendation, not enforce it. (The decision to NOT add a pre-archive external-review-presence check is intentional per design.md Non-Goals.)

#### Scenario: Edge-case assumptions for the iteration-aware logic

- **WHEN** the iteration-aware comparison logic encounters edge cases not explicitly enumerated in the prior scenarios (corrupted/empty `external-system-*.md`; multiple matching `external-system-*.md` or `review-system-*.json` files; filename-token parse failures)
- **THEN** the v1 behavior follows these documented assumptions: (a) An `external-system-*.md` SHALL be considered present regardless of content; corruption detection is out of scope and orbit's review skill does NOT validate the file's contents. (b) When multiple matching files exist, the most-recent filename `<TS>` token determines the comparison (lexical sort works because the token format is `YYYY-MM-DDTHH-MM-SSZ` — ISO-8601 with colons replaced; later timestamps sort lexically later). (c) If a filename `<TS>` token cannot be parsed, that file is treated as absent for comparison purposes (does not satisfy "external present"). Future drift-audit Cat 4 or downstream tools MAY add validation for these cases; orbit-review's v1 stays simple.

#### Scenario: Recommendation prose is auditable by drift-audit

- **WHEN** the stock-phrasings prose drifts from the `Review mode decision framework` requirement's criteria, OR the cited empirical evidence reference (`bootstrap-orbit-status-cli` 3-of-3) becomes inaccurate
- **THEN** `/opsx:audit-drift` Category 3 (cross-doc consistency) SHALL surface the drift on next run. The recommendation prose's empirical citation is treated as documentation that can rot; drift-audit catches the rot.
