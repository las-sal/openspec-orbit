## MODIFIED Requirements

### Requirement: Overlay file disposition

The system SHALL classify every file orbit ships in `.claude/` into one of four disposition categories. Each category dictates the file's lifecycle in orbit's overlay.

#### Scenario: Orbit-authored — full ownership

- **WHEN** orbit ships a file with no upstream-derived content (e.g., `openspec-review`, `openspec-audit-drift`, `openspec-review-external`, `openspec-address-reviews`, `openspec-onboard` — the orbit-authored onboarding skill replaces upstream's guided-tour body with 100% orbit-authored content)
- **THEN** the file is fully owned by orbit; orbit's contributors edit it freely; the file MAY use the `openspec-*` directory naming convention for consistency with siblings even when the body is 100% orbit-authored

#### Scenario: Orbit-modified — upstream body with `# Orbit additions`

- **WHEN** orbit ships a file that bundles upstream's body with an appended `# Orbit additions` section (e.g., `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`)
- **THEN** the upstream body is the pinned-version snapshot; orbit's additions append behavior or constraints; this pattern is transitional and a subject of future Option-2 work (dropping the additions pattern in favor of fully orbit-authored versions of these skills)

#### Scenario: Upstream-required primitive — kept verbatim

- **WHEN** orbit depends on an upstream skill as a callable primitive (invoked at the orchestration layer, not modified at the skill-body layer) — the concrete example is `openspec-sync-specs`, which orbit's archive flow invokes as a callable primitive even though upstream `init --tools claude` does not materialize the skill directory
- **THEN** orbit ships the upstream skill unchanged; orbit's use of the primitive matches upstream's internal use pattern; no `# Orbit additions` are needed because orbit's interaction is at the invocation layer, not the skill body

#### Scenario: Not shipped — pruned from overlay

- **WHEN** an upstream skill is truly unmodified, not used by orbit, and provides no orbit-mission value (e.g., `feedback`, which sends user feedback to Fission-AI and has no role in orbit's pegged workflow)
- **THEN** orbit does NOT ship the file in its overlay; the skill is simply absent from orbit's `.claude/skills/` directory. NOTE: because the overlay install model is `cp -r` (which does not delete target-only files), absence-in-overlay is necessary but not sufficient to ensure absence-in-user-project. The README install/update sections SHALL document explicit `rm` commands for pruned files so user-project state matches overlay intent.

#### Scenario: Per-skill disposition is documented

- **WHEN** orbit's contributors evaluate whether to ship a new upstream skill that lands in a future pinned-version upgrade
- **THEN** the disposition decision is recorded in the change proposal that performs the upgrade; the new skill is assigned to one of the four categories with rationale

#### Scenario: Commands follow the same 4-category framework

- **WHEN** orbit ships (or chooses not to ship) a slash-command file in `.claude/commands/opsx/`
- **THEN** the file's disposition is one of the four categories on the same criteria as skills: orbit-authored (e.g., `opsx/review.md`, `opsx/audit-drift.md`, `opsx/review-external.md`, `opsx/address-reviews.md`, `opsx/onboard.md`); orbit-modified (mirrors an upstream-bound capability with orbit-specific behavior, e.g., `opsx/propose.md`, `opsx/explore.md`, `opsx/archive.md`, `opsx/apply.md`, `opsx/verify.md`, `opsx/continue.md`, `opsx/new.md`, `opsx/bulk-archive.md`, `opsx/sync.md`); upstream-required primitive (none currently — orbit does not depend on any upstream command file as a callable primitive); not shipped (rare — applied per-file when an upstream command provides no orbit-mission value)

#### Scenario: `ff.md` / `fast-forward.md` naming divergence — known overlay disposition oddity

- **WHEN** orbit ships `.claude/commands/opsx/fast-forward.md` (orbit-authored, in the Orbit-authored category) alongside upstream's `.claude/commands/opsx/ff.md` (left untouched by the overlay because the filenames differ)
- **THEN** both files coexist in user projects after `cp -r` overlay; `fast-forward.md` is the canonical orbit-shipping for the fast-forward workflow; `ff.md` is upstream's untouched copy. This is a known overlay-disposition oddity tracked for resolution in a future overlay-cleanup change; it is acknowledged in baseline rather than silently tolerated

#### Scenario: Command-file disposition documented in the same change as a removal or addition

- **WHEN** orbit decides a command should be added to the overlay or pruned from it
- **THEN** the decision is recorded in the change proposal that performs the addition/removal with the disposition category cited; pruned commands must include `rm` step in README install/update documentation (analogous to pruned-skill handling)
