## MODIFIED Requirements

### Requirement: Emit scope

The following orbit commands SHALL emit `openspec/changes/<name>/.orbit-runs/<command>-<TS>.json` (or `openspec/explore/<name>/.orbit-runs/<command>-<TS>.json` for explore) when invoked. This requirement covers two categories:

- **New emit behavior** (commands that don't emit any run-summary JSON today):
  - Workflow commands: `/opsx:explore` (named mode only — bare mode does NOT emit), `/opsx:propose`, `/opsx:new`, `/opsx:continue`, `/opsx:ff`, `/opsx:apply`, `/opsx:verify`
  - `/opsx:review-external` at T0 (when the prompt is packaged, before findings return — today it writes only the `.md` prompt file)
- **Refined existing emit** (already emits today; this change refines the recommendation logic and aligns with the universal spine from orbit-conventions):
  - Standalone `/opsx:audit-drift` (already documented at `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`)

The following commands SHALL NOT emit additional JSONs as part of this change:

- `/opsx:bulk-archive` — wrapper; each inner `/opsx:archive` invocation emits separately
- `/opsx:onboard` — meta walkthrough; no change-state transition
- `/opsx:sync-specs` — primitive only under pegging strategy (per orbit-conventions `Distribution model — overlay, not CLI fork`); used by orbit's archive flow via subagent; not exposed as a user-callable command in orbit's overlay (the `.claude/commands/opsx/sync.md` user-callable command is pruned)

Existing emit-producing commands (`/opsx:review`, `/opsx:address-reviews`, `/opsx:archive`, inline `/opsx:audit-drift` during archive) continue emitting as today; this change does not modify their emit behavior.

#### Scenario: Named-mode /opsx:explore writes explore JSON
- **WHEN** the user invokes `/opsx:explore foo` (named mode) and the session ends
- **THEN** the emit-layer writes `openspec/explore/foo/.orbit-runs/explore-<TS>.json`

#### Scenario: Bare-mode /opsx:explore writes no JSON
- **WHEN** the user invokes `/opsx:explore` without a name argument
- **THEN** no `.orbit-runs/` JSON is written for that conversation

#### Scenario: /opsx:onboard does not emit
- **WHEN** the user invokes `/opsx:onboard` and completes the walkthrough
- **THEN** no `.orbit-runs/onboard-<TS>.json` is written anywhere in the project

#### Scenario: /opsx:bulk-archive relies on inner /opsx:archive emits
- **WHEN** the user invokes `/opsx:bulk-archive` to archive 3 changes
- **THEN** 3 `archive-<TS>.json` files are written (one per inner archive), and no `bulk-archive-<TS>.json` file is written
