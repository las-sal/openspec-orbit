# External Review: lean-overlay-and-add-orbit-onboard (proposal-mode iter-2)

**Reviewer model**: GPT-5 Codex 2026-05-23
**Findings count**: 2 CRITICAL, 1 WARNING, 1 SUGGESTION

---

## CRITICAL

### EC1' - EW1 reversal is still incomplete in proposal/design/tasks

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/proposal.md
**Line**: 50
**Dimension**: coherence

The iter-4 reversal is correct: upstream currently documents `/opsx:sync` as an "Optional command" that merges delta specs, not as deprecated. However, the proposal still says the `orbit-run-summary-emit` delta represents "primitive-only retention, user-callable surface pruned." The same false premise remains in `design.md:66` ("deprecated from direct user invocation"), `design.md:150` (sync.md example under NOT shipped / user-callable command pruned), `design.md:156` ("primitive only ... not exposed as a user-callable command (sync.md pruned)"), and `tasks.md:34` ("upstream deprecated direct invocation"). These lines directly contradict `proposal.md:26`, `design.md:164`, and `tasks.md:19`, which now say `/opsx:sync` is retained. An implementer could still remove or under-document `/opsx:sync` by following the stale task/design language, even though the reversal was supposed to settle that.

**Recommendation**: Sweep the active change artifacts, not just `.claude/`, for stale sync-deprecation/pruning language. Remove "deprecated from direct user invocation," "primitive-only," "user-callable surface pruned," and "sync.md pruned" from proposal.md, design.md, and tasks.md unless explicitly presented as a historical false claim inside the iter-4 reversal note. Update task 3.5 to say README should document `/opsx:sync` as retained optional upstream functionality while avoiding obsolete direct-use recommendations only where they are genuinely obsolete.

### EC2' - Onboard implementation tasks still contradict the corrected onboard spec

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md
**Line**: 40
**Dimension**: correctness

The orbit-onboard spec was corrected after prior reviews to use stable post-install artifacts for setup verification and to allow bare-mode `/opsx:explore` for orientation-only users. The implementation tasks still carry the old instructions: task 4.2 tells the implementer to verify "orbit-authored skill presence + at least one orbit-specific directory," even though the spec now explicitly forbids depending on `openspec/lenses/` or `openspec/explore/` because a correct fresh install may have neither. Task 4.6 still says the try-it nudge should recommend `/opsx:explore <name>` on a real project idea, omitting the spec's new bare-mode path for users who do not have an idea yet. Since tasks drive apply, this can reintroduce exactly the false-fail setup check and orientation gap that EW2/TR4 were supposed to close.

**Recommendation**: Update task 4.2 to name the stable artifacts from `orbit-onboard` (`openspec-review`, `openspec-audit-drift`, `opsx/review.md`, `opsx/address-reviews.md`, plus feedback absence and version pin). Update task 4.6 to require both closing recommendations: `/opsx:explore <name>` for concrete ideas and bare `/opsx:explore` for orientation-only users. Add a quick task-level grep/sanity note so future edits keep tasks and spec scenarios aligned.

## WARNING

### EW1' - Emit scope names the skill, not the retained slash command

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-run-summary-emit/spec.md
**Line**: 17
**Dimension**: correctness

The `Emit scope` delta is in the "following commands SHALL NOT emit" list, but it lists `/opsx:sync-specs` while also saying the retained user-callable command is `/opsx:sync`. In this repo the command file is `.claude/commands/opsx/sync.md` and its input text is `/opsx:sync`; `openspec-sync-specs` is the skill/primitive name. After the reversal, the manual `/opsx:sync` path is retained, so the spec should explicitly define whether `/opsx:sync` emits or not. As written, an implementer could read the exclusion as applying only to the primitive invoked during archive and leave manual `/opsx:sync` emit behavior ambiguous.

**Recommendation**: Rename the bullet to `/opsx:sync` (or `/opsx:sync` / `openspec-sync-specs` if both surfaces need naming) and state explicitly: manual `/opsx:sync` does not emit a standalone run-summary JSON; archive-invoked sync results are captured in `archive-<TS>.json`'s `sync_specs` field. Mirror the same naming in proposal/design/task prose.

## SUGGESTION

### ES1' - explore.md still presents the false sync deprecation claim as a current decision

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/explore.md
**Line**: 56
**Dimension**: coherence

The explore history still says `openspec-sync-specs` is "deprecated from direct user invocation in 1.3.1" and "not deprecated as a capability." Unlike a forensic `.orbit-runs/` log, `explore.md` is part of the proposal's decision history that future readers are asked to consume. Leaving D7 unamended makes the same false claim look like a standing design premise rather than a reversed one.

**Recommendation**: Add a short amendment under D7 noting the 2026-05-23 iter-4 reversal: upstream docs list `/opsx:sync` as optional, not deprecated; orbit keeps both `/opsx:sync` and `openspec-sync-specs` under the full-support-for-1.3.1 principle. Preserve the original path as historical context if useful, but mark it superseded.
