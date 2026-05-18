# External Review: bootstrap-openspec-orbit (iteration 4)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-18

## CRITICAL

None.

## WARNING

### Current docs still count the two review modes as two new commands
**File**: README.md:46
**Description**: README still says the overlay adds five new opsx commands, and design.md:17 says the same. proposal.md:11-12 then preserves that old count by listing `/opsx:review --as proposal` and `/opsx:review --as system` as two separate "New command" bullets. After the merge, the proposal's own capability list has 8 capabilities and a single `orbit-review` command with two modes, so the current command count should be four new commands (`review`, `review-external`, `audit-drift`, `address-reviews`) plus three modified upstream commands. Update the count and collapse the two proposal bullets into one `/opsx:review <name> [--as proposal|system]` bullet so the top-level scope matches the merged capability model.

### README repo layout still points implementers at deleted review command files
**File**: README.md:697
**Description**: The repo-layout block still lists pending command bodies `review-proposal.md` and `review-system.md`, even though the merge deleted those and introduced `.claude/commands/opsx/review.md`. The skills block also lists `openspec-review/` twice, where it should list the unified review skill once. This is active adopter-facing install/layout documentation, not historical context, and would lead implementation toward recreating the split command files. Replace the pending command entries with `review.md, audit-drift.md, address-reviews.md, review-external.md` and deduplicate the skills list.

### orbit-conventions final-assessment contract still names old review commands
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-conventions/spec.md:224
**Description**: The cross-cutting final-assessment requirement still says `<gate>` is `/opsx:apply` for `review-proposal` and `/opsx:archive` for `review-system`. Since `orbit-conventions` defines shared reporting behavior consumed by the merged review command, this is normative current-state residue. Reword it to say `<gate>` is `/opsx:apply` for `/opsx:review --as proposal`, `/opsx:archive` for `/opsx:review --as system`, and varies by audit-drift invocation context.

### audit-drift spec still treats review-system as the caller/report owner
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-audit-drift/spec.md:120
**Description**: The audit-drift spec has current requirements for "Library call from review-system" and summaries written in a change context "e.g., from review-system"; the sketch repeats the same shape in its final-assessment table and composition diagram. After the merge there is no `/opsx:review-system` command, only system mode of `/opsx:review`. Update these active spec/sketch references to `/opsx:review --as system` or "review system-mode report" so implementers do not create a separate review-system caller in the audit-drift integration.

## SUGGESTION

### address-reviews sketch still uses split review names in its closing diagram/table
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/address-reviews.md:248
**Description**: The bottom composition diagram says "re-run review-proposal / review-system to confirm clean", and the parallels table still has columns named `review-proposal` and `review-system`. Earlier prose in the same sketch was updated to `/opsx:review --as proposal` and `/opsx:review --as system`, so this is sibling-diagram residue from the merge. Update the diagram/table labels to `review --as proposal` and `review --as system` for consistency.

### tasks.md keeps a numbering gap after merging Group 3 into Group 2
**File**: openspec/changes/bootstrap-openspec-orbit/tasks.md:25
**Description**: The task groups jump from `## 2. New skill: /opsx:review` directly to `## 4. New skill: /opsx:review-external`. The prompt calls this cosmetic, but the task file is the implementation checklist and group numbers are used in references, so either renumber subsequent groups or add a short note that Group 3 was intentionally merged into Group 2.

## Notes

`openspec validate bootstrap-openspec-orbit` passes. The merged `specs/orbit-review/spec.md` and `sketches/review.md` cover both modes coherently; I did not find leftover `orbit-review-proposal` or `orbit-review-system` capability files. `@review:` hits in change artifacts are documentation/examples, not unresolved review markers. Historical old-name mentions in `explore.md` were not flagged where later entries clearly supersede them.
