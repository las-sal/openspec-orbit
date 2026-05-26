# External Review: harden-review-mode-recommendations (iteration 2)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-26

## CRITICAL

None.

## WARNING

### Later address-reviews runs can leave the state machine with no matching state
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:34
**Description**: Path A says a clean external only converges when no later `apply-*.json` or `address-reviews-*.json` is newer than the external, and Path B says a resolved external only converges when no later `apply-*.json` or later `address-reviews-*.json` for other inputs is newer than the resolution JSON. But the stale scenario at line 49 only matches later `apply-*.json`, not later address-reviews, and the unresolved scenario at line 44 does not match a clean external or an external whose findings were already resolved. That means a clean external followed by an unrelated address-reviews run, or a resolved external followed by a later address-reviews run for another input, can fail all five convergence states. Either make later address-reviews a stale/artifact-changed trigger with matching stock phrasing, or remove later address-reviews from the Path A/Path B "artifact-changing event" guards unless the state machine can route it.

### Path A clean parsing is stricter than the external-findings parser contract
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:34
**Description**: Path A requires each severity section to contain only exact `None.`, but `.claude/skills/openspec-address-reviews/references/external-findings-format.md:67` explicitly treats `None`, `none.`, and `(none)` as equivalent empty-severity sentinels. A system-mode external review using one of those parser-supported clean forms would be ingested as zero findings by address-reviews, but this new review-mode logic would classify it as not clean and fall toward unresolved/stale behavior. Reuse the existing external-findings parser contract for Path A, or explicitly revise the parser contract and prompt template so the whole system has one clean-sentinel definition.

### Path B does not define how source_path references are matched
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:39
**Description**: Path B depends on the most recent `address-reviews-*.json` whose `source_path` "references that external file", but the address-reviews schema stores the user-supplied `--from-file` path. In practice that may be repo-relative, absolute, or otherwise normalized differently from the external file path the reviewer logic is comparing against. Two implementers could choose exact string match, basename match, or normalized-path match and produce different convergence decisions. Specify the comparison rule, ideally canonicalizing both paths to repo-relative paths before exact comparison, with absolute paths accepted only after normalization to the same repo-relative target.

## SUGGESTION

### User-validation checklist still describes the old three-state model
**File**: openspec/changes/harden-review-mode-recommendations/tasks.md:45
**Description**: Task 3.2 still asks the user to confirm variants only for "no prior external / converged clean / stale" and implementation clarity for "file-existence check + timestamp comparison". The actual proposal is now a five-state model with Path A markdown parsing, Path B source_path plus `resolution_summary` inspection, unresolved findings, stale post-apply, and precedence rules. Update the user-validation checklist so the human review explicitly covers all five states and the new parsing/path-matching assumptions, not just the old three-state shape.

## Notes

The iter-1 CRITICAL appears materially fixed: the revised proposal no longer treats external file presence plus timestamp ordering as convergence. The remaining findings are about making the new five-state model exhaustive and implementable without local inference.
