## ADDED Requirements

### Requirement: Install documentation describes actual install surface

The README's installation, update, and uninstall sections SHALL describe the actual install surface produced by the pinned `@fission-ai/openspec` version (per `Upstream version pinning`). Skill counts, command counts, file lists, CLI invocations, and behavior warnings SHALL match what a fresh sandbox install produces. This requirement exists because the README-vs-install-surface drift has caused two cluster-2 change cuts (issue #6's original delete-9-files framing and the original `lean-overlay-and-add-orbit-onboard` chunks 3-5 cut on 2026-05-23); discipline alone was not sufficient to prevent recurrence.

#### Scenario: README skill and command counts match install output

- **WHEN** a reader follows the README's documented install steps in a fresh sandbox (`mktemp -d` + same Node version as the pinned upstream supports)
- **THEN** the resulting `.claude/skills/` and `.claude/commands/opsx/` directories contain exactly the files the README's "What you should see after install" section enumerates, with counts matching exactly; no file is documented that isn't installed, and no installed file is omitted from the documentation

#### Scenario: CLI invocations in README are non-interactively executable

- **WHEN** a reader executes the CLI commands documented in the README's install, update, or uninstall sections in a non-interactive shell (stdin closed)
- **THEN** each command succeeds (or exits with the specific error/warning the README documents); no command produces an unexpected error from missing required flags, no command hangs on a prompt the README didn't warn about, no command's exit behavior contradicts the prose

#### Scenario: README-modifying changes pair with sandbox verification

- **WHEN** a change proposal modifies README install/update/uninstall sections
- **THEN** the change SHALL include an apply-time fresh-sandbox verification task that runs the documented CLI sequence end-to-end and confirms the file-system state matches the documentation; verification failures escalate via `@review(escalated):` rather than ship undocumented behavior

#### Scenario: Upgrade and overlay-change proposals include README sync

- **WHEN** a change proposal upgrades the pinned upstream version (per `Upstream version pinning`) OR modifies overlay file disposition (per `Overlay file disposition` — adding to or removing from the four categories)
- **THEN** the proposal SHALL include a README install-section synchronization task that updates counts, file lists, and CLI invocations to match the new install surface; the synchronization task is required before the upgrade/overlay-change is considered apply-complete

#### Scenario: User-facing behavior warnings are accurate, not pessimistic

- **WHEN** the README documents the behavior of a CLI command (e.g., `init --force` overwrite scope, `cp -r` overlay non-deletion behavior)
- **THEN** the warning text SHALL accurately describe what the command does in the pinned version, neither understating nor overstating risk; overly-cautious warnings that don't match actual behavior are themselves drift and SHALL be corrected to match sandbox-verified facts
