# External Review: harden-review-mode-recommendations (iteration 1, system mode)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-26

## CRITICAL

None.

## WARNING

### Design state machine still documents pre-resolution Path A/Path B rules
**File**: openspec/changes/harden-review-mode-recommendations/design.md:56
**Description**: The applied spec and review-command prose now resolve the proposal iter-2 findings by accepting the external-findings empty sentinels (`None.`, `None`, `none.`, `(none)`), treating only later `apply-*.json` files as stale-making artifact changes, ignoring unrelated `address-reviews-*.json` files for this external's convergence, and repo-relative-normalizing Path B `source_path` comparisons. The design rationale still says Path A is clean only when sections contain `"None."`, still disqualifies Path A when any later `address-reviews-*.json` exists, and still describes Path B as a loose `source_path referencing this external` check. That leaves the change's design document contradicting the final requirement/SKILL contract and could reintroduce the exact gaps fixed by `address-reviews-2026-05-26T13-57-50Z.json`. Update D-logic-1 in `design.md` to mirror the current `specs/orbit-review/spec.md` state model, or explicitly mark the old sketch as pre-iter-2 historical context and add the corrected final model immediately after it.

### README small-change guidance contradicts the system-mode default flip
**File**: README.md:499
**Description**: The small-change row says the suggested system-mode cycle is `In-context once; accept default proceed-to-archive`, but this change's primary behavior is that when no external system review exists, the default final assessment recommends `/opsx:review-external` or `/opsx:review --fresh` before archive, with direct archive only as an explicit risk acceptance path. A cold reader can reasonably infer from this row that the old all-clear system-mode default still proceeds to archive. Rewrite the cell to preserve the low-stakes guidance without calling direct archive the default, for example: `In-context once; proceed directly only if accepting the noted in-context-only risk`.

## SUGGESTION

None.

## Notes

The core implemented surfaces line up well: `openspec validate harden-review-mode-recommendations --strict` and `openspec validate --all --strict` passed, the SKILL/command/spec Path A and Path B wording now match the iter-2 resolutions, the external-findings parser contract includes the accepted empty-sentinel variants, and I did not find unresolved `@review:` markers in the change directory.
