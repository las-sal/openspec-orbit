# External Review: address-reviews-defaults-and-decision-forks (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-27

## CRITICAL

None.

## WARNING

### Primary address-reviews discipline still says v1 does not auto-cascade
**File**: .claude/skills/openspec-address-reviews/SKILL.md:32
**Description**: The command frontmatter and Step 3d now correctly describe cascade-by-default, but the self-contained "Change completeness" discipline still tells the agent to "surface" ripple files, says "Step 5 below", and explicitly says "v1 does NOT auto-cascade — the user gets a list of files to check, not silent edits." That block is near the top of the primary skill and says the discipline applies throughout the command, so it can override the later implementation prose for exactly the behavior this change flips. Replace this paragraph with the new contract: derive ripple-flagged files, auto-apply IN-set ripples by default during Step 3d, record OUT-set and `--no-cascade` suppressed paths in `ripple_cascade.flagged_not_applied[]`, and keep the post-edit residue sweep.

### Stale and unresolvable classifications still jump to Step 3d after 3d became cascade
**File**: .claude/skills/openspec-address-reviews/SKILL.md:133
**Description**: The classifier still says stale findings "Skip directly to 3d (remove + log as stale)", and the unresolvable default says to proceed to 3d after filing a task. After this change's reorder, Step 3d is ripple cascade and marker removal is Step 3e. A fresh implementer following this text could run cascade for a stale finding or skip the actual marker-removal step. Update stale to skip to Step 3e/no-op removal plus stale logging, and update unresolvable to apply its task/TODO/escalation action in Step 3c before normal Step 3e removal/transform behavior.

### CLAUDE.md still teaches the old ripple-flag lifecycle
**File**: CLAUDE.md:27
**Description**: The repo-level context that auto-loads for future agents still summarizes `/opsx:address-reviews` as "lean v1: discover -> triage -> walk -> ripple flag -> report" and only says markers are removed on resolution. That conflicts with the newly landed public behavior: walk-mode is the default, cascade auto-applies IN-set ripples by default, and marker removal happens after cascade. Since this is a governing doc future implementation/review sessions read before SKILL.md details, update the lifecycle sentence to match the new pushback -> classify -> fix -> ripple-cascade -> remove-marker flow and mention `--batch` / `--no-cascade` as the opt-outs.

## SUGGESTION

None.

## Notes

`openspec validate address-reviews-defaults-and-decision-forks --strict` and `openspec validate --all --strict` both pass. The task-state gate matches the prompt's expected final state: 35 checked AI-doable tasks and 8 unchecked user-validation handoff tasks. `openspec/lenses/` is absent, so Passes 4 and 5 were skipped per prompt. The `@review:` sweep only found documented marker syntax/examples, not unresolved inline markers.
