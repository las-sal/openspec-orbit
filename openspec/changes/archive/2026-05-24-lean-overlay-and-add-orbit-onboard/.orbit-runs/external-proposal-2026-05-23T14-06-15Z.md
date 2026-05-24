# External Review: lean-overlay-and-add-orbit-onboard (proposal-mode iter-1)

**Reviewer model**: GPT-5 Codex 2026-05-23
**Findings count**: 2 CRITICAL, 3 WARNING, 2 SUGGESTION

---

## CRITICAL

### EC1 - Feedback pruning will not take effect in installed projects

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/proposal.md
**Line**: 56
**Dimension**: correctness

The change keeps the manual `cp -r` overlay install model unchanged, while task 1.1 only deletes `.claude/skills/feedback/` from the orbit repository. That does not make `feedback` absent in a user project. The documented install flow starts from upstream `openspec init`, which creates `.claude/skills/feedback/`; copying orbit's `.claude/skills/.` over that tree will overwrite same-name files but will not delete a target-only `feedback/` directory. Existing orbit installs have the same problem when they re-run the overlay. This contradicts the new `Not shipped - pruned from overlay` scenario, which says the skill is absent from orbit's `.claude/skills/` directory, and it means #6's visible cleanup will not actually occur for fresh or existing users.

**Recommendation**: Add an explicit fresh-install and update/migration step that removes `feedback` from the target project, e.g. `rm -rf .claude/skills/feedback`, after upstream init/update and before or after the orbit copy. Track it in `tasks.md`, update README install/update/uninstall guidance, and make the fresh-sandbox verification in task 3.4 assert that `.claude/skills/feedback/` is absent after the full documented install. If the intent is only "orbit repo does not ship feedback" while upstream init leaves it present, revise the spec and proposal so they do not claim the skill is absent from installed orbit.

### EC2 - /opsx:onboard command body would keep the old guided-tour behavior

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md
**Line**: 46
**Dimension**: correctness

Task 4.7 says `.claude/commands/opsx/onboard.md` most likely needs no change because "the command file binds to skill name not skill content." Current state does not support that assumption: `.claude/commands/opsx/onboard.md` is a full upstream-authored guided-tour prompt. It tells the AI to "do real work in their codebase" and explicitly runs `openspec new change "<derived-name>"`. If the implementer rewrites only `.claude/skills/openspec-onboard/SKILL.md`, invoking `/opsx:onboard` can still follow the old command body, create a change, and violate the new orbit-onboard requirement that onboarding is a reference-leaning 5-section body that does not orchestrate workflows or create artifacts.

**Recommendation**: Make rewriting `.claude/commands/opsx/onboard.md` an explicit task, not a maybe. Either replace the command body with a thin dispatcher that invokes the orbit-authored `openspec-onboard` skill, or duplicate the same 5-section reference content in the command file if slash commands are the executable surface. Add a verification step that greps the command file for old guided-tour residue such as `openspec new change`, `Phase 4: Create the Change`, and "do real work in their codebase".

## WARNING

### EW1 - sync-specs disposition contradicts existing specs and the direct command surface

**File**: openspec/specs/orbit-run-summary-emit/spec.md
**Line**: 23
**Dimension**: coherence

The just-archived `orbit-run-summary-emit` baseline still says `/opsx:sync-specs` is "deprecated upstream; slated for removal by openspec-orbit#6." `orbit-conventions` also says the `sync_specs` archive field is transitional until #6 removes `/opsx:sync-specs`, and the archive summary schema repeats the same future-removal statement. This proposal closes #6 under the pegging strategy and keeps `openspec-sync-specs` as an upstream-required primitive, but it does not modify those baseline claims. It also leaves `.claude/commands/opsx/sync.md` in the command surface, while tasks only remove README direct-use mentions. After archive, the repo would disagree about whether sync is removed, retained as a primitive only, or still user-invokable.

**Recommendation**: Decide the final sync surface in this change. If sync is primitive-only, add tasks to remove or stop shipping `.claude/commands/opsx/sync.md`, and update `orbit-run-summary-emit`, `orbit-conventions`'s internal-run JSON summary text, and `archive-summary-schema.md` so they no longer say #6 will remove it later. If `/opsx:sync` remains a user command, update the proposal/specs/README/onboard quick reference to classify and document it explicitly instead of only removing direct-use prose.

### EW2 - Setup verification can fail a correct fresh install

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md
**Line**: 43
**Dimension**: correctness

The setup verification scenario requires an orbit-authored skill plus "at least one orbit-specific directory," giving `openspec/lenses/` or `openspec/explore/` as examples. Those are not stable install artifacts. The baseline conventions allow `openspec/lenses/` to be empty on a fresh project, and the README currently says `openspec/lenses/` "doesn't exist yet" until first capture; `openspec/explore/` is likewise created by later workflow use. The new onboard spec also forbids the skill from creating persistent artifacts, so onboarding cannot repair this by creating those directories. A fresh, correctly installed project can therefore be reported as incomplete.

**Recommendation**: Base setup verification on artifacts that exist immediately after install, such as `.claude/commands/opsx/review.md`, `.claude/commands/opsx/address-reviews.md`, `.claude/skills/openspec-review/SKILL.md`, `.claude/skills/openspec-audit-drift/SKILL.md`, and the pinned `openspec --version` result. Treat `openspec/lenses/` and `openspec/explore/` as optional workflow-state evidence, or change the install instructions to create empty directories and update baseline conventions accordingly.

### EW3 - Overlay file disposition says "every file" but only classifies skills

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-conventions/spec.md
**Line**: 60
**Dimension**: completeness

The `Overlay file disposition` requirement says every file orbit ships in `.claude/` is classified into four categories, but the scenarios and examples only classify skill directories (`openspec-review`, `openspec-explore`, `openspec-sync-specs`, `feedback`). The distribution model includes `.claude/commands/opsx/` as part of the deliverable, and command files are where important user-visible behavior lives (`onboard.md`, `sync.md`, `archive.md`, etc.). Without command-file disposition, the spec can pass while the slash-command surface remains stale or inconsistent with the skill disposition.

**Recommendation**: Either narrow the requirement to "every skill orbit ships in `.claude/skills/`" or add command-file disposition scenarios/manifest entries. At minimum, classify `onboard.md` and `sync.md` in this change because they are directly implicated by the orbit-onboard rewrite and the sync-specs retention decision.

## SUGGESTION

### ES1 - Option 2 and global rename follow-ups need durable tracking handles

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/design.md
**Line**: 24
**Dimension**: completeness

The design leans on future Option 2 work (dropping the `# Orbit additions` pattern) and a possible global `openspec-*` to `orbit-*` rename, but it only says these are "tracked as the next explore cycle" or "in its own change." Unlike #26, there is no issue number or other durable handle. Because this change deliberately writes forward-looking identity language meant to survive Option 2, future contributors need a concrete place to find or continue that deferred work.

**Recommendation**: File issues for Option 2 cleanup and any global skill rename, or add explicit TODO/tracking references in `design.md` and README deferred-work sections before archive.

### ES2 - Design says three disposition categories but lists four

**File**: openspec/changes/lean-overlay-and-add-orbit-onboard/design.md
**Line**: 146
**Dimension**: coherence

The design introduces `Overlay file disposition` as "Three categories" and then lists four bullets: orbit-authored, orbit-modified, upstream-required primitive, and NOT shipped. The spec correctly says four categories, so this is small residue rather than a design problem.

**Recommendation**: Change "Three categories" to "Four categories" in `design.md`.
