## ADDED Requirements

### Requirement: Upstream archive behavior preserved

The system SHALL preserve upstream `/opsx:archive`'s core behavior: validate the change is ready, run `sync-specs` to merge deltas into baseline, and move the change directory to `openspec/changes/archive/<YYYY-MM-DD>-<name>/`.

#### Scenario: Standard archive flow

- **WHEN** the user invokes `/opsx:archive <change-name>` for an applied change with no orbit-specific intervention
- **THEN** the upstream sequence runs: validation, sync-specs, move-to-archive — exactly as upstream

### Requirement: Pre-archive audit-drift invocation

The system SHALL auto-invoke `/opsx:audit-drift` as a pre-archive sweep before completing the archive operation, unless `--skip-audit` is set.

#### Scenario: Audit runs before archive

- **WHEN** `/opsx:archive <change-name>` is invoked without `--skip-audit`
- **THEN** the command invokes `/opsx:audit-drift` as a library function before any move or sync-specs operation; the audit's findings are presented in the archive output

#### Scenario: `--skip-audit` bypass

- **WHEN** the command runs with `--skip-audit`
- **THEN** Pre-archive audit is skipped; the archive run summary records `audit_skipped_via_flag: true`

### Requirement: Critical-drift prompt, not gate

The system SHALL prompt the user (not block) when audit-drift reports critical findings.

#### Scenario: Critical drift detected

- **WHEN** the pre-archive audit reports one or more CRITICAL findings
- **THEN** the command prompts via `AskUserQuestion`: "Address now / Proceed with archive / Abort?" with the audit's full findings shown to inform the choice

#### Scenario: Address now

- **WHEN** the user chooses "Address now"
- **THEN** the archive does not proceed; the user can fix the drift issues and re-invoke the archive command later

#### Scenario: Proceed despite critical

- **WHEN** the user chooses "Proceed with archive" despite critical findings
- **THEN** the archive proceeds normally and the archive run summary records `user_decision: proceeded_despite_critical`

#### Scenario: Abort

- **WHEN** the user chooses "Abort"
- **THEN** the archive is cancelled; no move, no sync-specs, no summary written

#### Scenario: No critical findings

- **WHEN** the pre-archive audit reports no CRITICAL findings
- **THEN** the archive proceeds without prompting; warnings (if any) are logged in the archive run summary

### Requirement: Archive run summary written

The system SHALL write a JSON summary to `openspec/changes/archive/<YYYY-MM-DD>-<change-name>/.orbit-runs/archive-<TS>.json` after a successful archive (the `<YYYY-MM-DD>-` prefix is added by upstream's archive move step).

#### Scenario: Summary contents

- **WHEN** the archive completes
- **THEN** the JSON includes: command name, timestamp, change name, audit (ran/skipped, findings summary by severity), user_decision (one of: `proceeded_with_no_critical`, `proceeded_despite_critical`, `audit_skipped_via_flag`), sync_specs (capabilities updated, counts of added/modified/removed/renamed)

### Requirement: `.orbit-runs/` directory moves with the change

The system SHALL move the change's `.orbit-runs/` directory along with the rest of the change content into the archive.

#### Scenario: Move with the change

- **WHEN** the archive moves `openspec/changes/<name>/` to `openspec/changes/archive/<YYYY-MM-DD>-<name>/`
- **THEN** the `.orbit-runs/` subdirectory is moved as part of the change content; all prior internal-run summaries and external-review findings persist in the archived location at `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/`

### Requirement: Unresolved marker warning

The system SHALL grep the change directory for unresolved `@review:` markers before completing the archive and warn the user if any are present.

#### Scenario: Unresolved markers detected

- **WHEN** `openspec/changes/<change-name>/` contains one or more `@review:` markers
- **THEN** the command warns: "N unaddressed `@review:` markers will land in archive — convert to `@todo:` or address before archiving?" and offers the user options to convert / address / proceed

#### Scenario: No markers

- **WHEN** the change directory contains no `@review:` markers
- **THEN** the archive proceeds without this warning

### Requirement: Edge cases

The system SHALL handle archive edge cases predictably.

#### Scenario: Already archived

- **WHEN** the user invokes `/opsx:archive <change-name>` for a change already in `openspec/changes/archive/<YYYY-MM-DD>-<change-name>/`
- **THEN** the command halts with a clear error: "Change <name> is already at openspec/changes/archive/<YYYY-MM-DD>-<name>/."

#### Scenario: audit-drift fails to run

- **WHEN** the pre-archive audit fails to execute (parse error, internal exception)
- **THEN** the archive proceeds with a warning; the archive run summary records `audit.ran: false` with the failure reason

### Requirement: Audit informational, not blocking by default

The system SHALL treat audit-drift findings as informational unless the user explicitly chooses to abort.

#### Scenario: Don't auto-fix

- **WHEN** audit-drift surfaces findings
- **THEN** the archive command does NOT attempt to resolve them automatically; the user's options are surface-prompt-proceed-or-abort

#### Scenario: Don't re-run reviews

- **WHEN** the user invokes archive
- **THEN** the command does NOT automatically invoke `/opsx:review --as system` even if it hasn't run; system-mode review is the user's gate (their responsibility)
