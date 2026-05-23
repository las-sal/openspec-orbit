## RENAMED Requirements

- FROM: `### Requirement: Distribution model — overlay, not CLI fork`
- TO: `### Requirement: Distribution model — pegged engine + orbit-owned surface`

(Rename rationale per design D-conventions-1 item 1 and address-reviews iter-3 FF2 resolution: the "overlay, not CLI fork" framing is the exact source of confusion that this change argues against. The MODIFIED requirement body describes pegging; the title was kept in the MODIFIED block to satisfy the MODIFY-by-name match rule, but the RENAMED operation completes the alignment at archive sync. Cross-references throughout this change's deltas reference the new name.)

## MODIFIED Requirements

### Requirement: Distribution model — pegged engine + orbit-owned surface

The system SHALL be distributed as a `.claude/` overlay pegged to a specific upstream `@fission-ai/openspec` version, not as a fork of the upstream OpenSpec CLI. Orbit owns the `.claude/` surface (skills, commands, supporting docs); upstream supplies the CLI binary as a pinned engine.

#### Scenario: Overlay shape

- **WHEN** orbit ships
- **THEN** the deliverable is markdown content under `.claude/commands/opsx/` and `.claude/skills/openspec-*/`, plus supporting docs; the upstream `@fission-ai/openspec` CLI binary is unchanged

#### Scenario: Consumers install after openspec init at pinned version

- **WHEN** a user adopts orbit in a project
- **THEN** they first install upstream `@fission-ai/openspec` at orbit's pinned version (see `Upstream version pinning` requirement), run upstream `openspec init` to scaffold the project's `.claude/` and `openspec/` directories, then drop orbit's overrides + new files into the same locations; orbit does not include or replace upstream's CLI

#### Scenario: Pinned upstream version is contractual

- **WHEN** orbit is in use against an upstream version other than the pinned one
- **THEN** orbit's behavior is undefined; users are responsible for verifying their installed upstream version matches orbit's pin (per `Upstream version pinning` requirement)

#### Scenario: Overlay-overwrite behavior is intentional under pegging

- **WHEN** orbit's overlay copies files into `.claude/skills/` or `.claude/commands/opsx/` that share names with upstream-`init`-installed files
- **THEN** the overwrite is the intended distribution mechanism; under pegging there is no "fresh upstream copy to preserve" — the pinned upstream version produces deterministic files, and orbit's overrides are the orbit-owned versions for that pinned upstream

## ADDED Requirements

### Requirement: Upstream version pinning

The system SHALL declare a specific pinned upstream `@fission-ai/openspec` version that orbit is tested against and contractually depends on. Orbit's behavior outside the pinned version is undefined. Version upgrade is a deliberate change-proposal event, not an automatic ingest.

#### Scenario: Pinned version declared in CLAUDE.md

- **WHEN** a user or AI reads `CLAUDE.md` at the orbit repo root
- **THEN** the pinned upstream version is clearly stated in the introductory or installation context

#### Scenario: Pinned version declared in README

- **WHEN** a user reads the orbit `README.md`
- **THEN** the install section names the exact pinned version (e.g., `@fission-ai/openspec@1.3.1`) as a required prerequisite

#### Scenario: Doc-only enforcement is acceptable

- **WHEN** orbit ships without an install script (current state)
- **THEN** documentation-only declaration is sufficient enforcement; install-time version-check is tracked as a separate future enhancement (orbit issue #26)

#### Scenario: Version upgrade is a deliberate change

- **WHEN** orbit's contributors consider upgrading the pinned upstream version
- **THEN** the upgrade is proposed as its own OpenSpec change (with explicit upgrade-and-port scope), not performed as silent maintenance; the new pinned version replaces the prior one in CLAUDE.md + README + this requirement's documented value

#### Scenario: Upstream improvements do not auto-propagate

- **WHEN** upstream releases `@fission-ai/openspec` v1.3.2, v1.4.0, or later while orbit is pinned to an earlier version
- **THEN** orbit users do not receive those improvements until a deliberate upgrade change lands in orbit; this is an accepted trade-off (per design.md D-arch-1)

### Requirement: Overlay file disposition

The system SHALL classify every file orbit ships in `.claude/` into one of four disposition categories. Each category dictates the file's lifecycle in orbit's overlay.

#### Scenario: Orbit-authored — full ownership

- **WHEN** orbit ships a file with no upstream-derived content (e.g., `openspec-review`, `openspec-audit-drift`, `openspec-review-external`, `openspec-address-reviews`, the orbit-authored `openspec-onboard`)
- **THEN** the file is fully owned by orbit; orbit's contributors edit it freely; the file MAY use the `openspec-*` directory naming convention for consistency with siblings even when the body is 100% orbit-authored

#### Scenario: Orbit-modified — upstream body with `# Orbit additions`

- **WHEN** orbit ships a file that bundles upstream's body with an appended `# Orbit additions` section (e.g., `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`)
- **THEN** the upstream body is the pinned-version snapshot; orbit's additions append behavior or constraints; this pattern is transitional and a subject of future Option-2 work (dropping the additions pattern in favor of fully orbit-authored versions of these skills)

#### Scenario: Upstream-required primitive — kept verbatim

- **WHEN** orbit depends on an upstream skill as a callable primitive (invoked at the orchestration layer, not modified at the skill-body layer)
- **THEN** orbit ships the upstream skill unchanged; orbit's use of the primitive matches upstream's internal use pattern; no `# Orbit additions` are needed because orbit's interaction is at the invocation layer, not the skill body

#### Scenario: Not shipped — pruned from overlay

- **WHEN** an upstream skill is truly unmodified, not used by orbit, and provides no orbit-mission value (e.g., `feedback`, which sends user feedback to Fission-AI and has no role in orbit's pegged workflow)
- **THEN** orbit does NOT ship the file in its overlay; the skill is simply absent from orbit's `.claude/skills/` directory. NOTE: because the overlay install model is `cp -r` (which does not delete target-only files), absence-in-overlay is necessary but not sufficient to ensure absence-in-user-project. The README install/update sections SHALL document explicit `rm` commands for pruned files so user-project state matches overlay intent. See orbit-onboard `Setup verification section` for the verify-absence scenarios users invoke to check.

#### Scenario: Per-skill disposition is documented

- **WHEN** orbit's contributors evaluate whether to ship a new upstream skill that lands in a future pinned-version upgrade
- **THEN** the disposition decision is recorded in the change proposal that performs the upgrade; the new skill is assigned to one of the four categories with rationale

#### Scenario: Commands follow the same 4-category framework

- **WHEN** orbit ships (or chooses not to ship) a slash-command file in `.claude/commands/opsx/`
- **THEN** the file's disposition is one of the four categories on the same criteria as skills: orbit-authored (e.g., `opsx/review.md`, `opsx/audit-drift.md`, `opsx/review-external.md`, `opsx/address-reviews.md`); orbit-modified (mirrors an upstream-bound capability with orbit-specific behavior, e.g., `opsx/propose.md`, `opsx/explore.md`, `opsx/archive.md`, `opsx/apply.md`, `opsx/verify.md`, `opsx/continue.md`, `opsx/fast-forward.md`, `opsx/new.md`, `opsx/bulk-archive.md`, `opsx/sync.md`); upstream-required primitive (none currently — orbit does not depend on any upstream command file as a callable primitive); not shipped (rare — applied per-file when an upstream command provides no orbit-mission value)

#### Scenario: Command-file disposition documented in the same change as a removal or addition

- **WHEN** orbit decides a command should be added to the overlay or pruned from it
- **THEN** the decision is recorded in the change proposal that performs the addition/removal with the disposition category cited; pruned commands must include `rm` step in README install/update documentation (analogous to pruned-skill handling)
