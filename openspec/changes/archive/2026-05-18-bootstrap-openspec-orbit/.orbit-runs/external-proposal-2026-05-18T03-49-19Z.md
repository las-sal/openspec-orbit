# External Review: bootstrap-openspec-orbit (iteration 3)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-18

## CRITICAL

None.

## WARNING

### proposal.md still describes review-external as emitting the full prompt to chat
**File**: openspec/changes/bootstrap-openspec-orbit/proposal.md:13
**Description**: The proposal's active What Changes list says `/opsx:review-external` "emits a self-contained prompt to chat", but the resolved contract is file-backed: write `external-prompt-<as>-<TS>.md` and emit a short invocation snippet to chat. The primary sketch still has the same residue in its purpose line (`sketches/review-external.md:8`, "Generates a ready-to-paste prompt"). README/design/spec now describe the file-backed flow, so update proposal.md and the sketch purpose to match. Otherwise a fresh implementer could treat the proposal as permission to resurrect the old chat-paste handoff.

### Chat-output exactness omits the required inferred-mode note
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-external/spec.md:53
**Description**: The chat invocation scenario now enumerates the exact chat output items and includes the dirty-tree warning plus recommended-session note, but it still omits the mode-inference note required at lines 23-29 and echoed in the sketch heuristic at `sketches/review-external.md:321`. When `--as` is omitted, the command must both show "Generating external-review prompt as `proposal`/`system` (inferred from tasks state)" and obey the "exactly" chat-output contract. Add the inferred-mode note to the allowed output list (with ordering relative to `Recommended session:` and dirty-tree warnings), or relax the exactness wording so all required diagnostics can coexist.

### README external cycle omits the new full-chat-output and commit/push completion contract
**File**: README.md:501
**Description**: The concrete external-review walkthrough still says the external AI writes findings to the file, and only mentions chat output as a fallback when the external AI cannot write files. The current `orbit-review-external` spec requires every generated prompt to tell the external AI to output the complete findings markdown in chat in addition to writing the file, and to commit/push the findings file when git access is available. Update the README walkthrough around steps 7 and 8 so adopter-facing docs match the new "full findings markdown in chat + file remains canonical + commit/push if possible" contract.

## SUGGESTION

### Workflow diagram still uses approximate Mode-C threshold wording
**File**: README.md:84
**Description**: The README prose and tasks now use "2+ substantive decisions", but the workflow diagram still says crystallized mode prompts after "~2 decisions emerge". Since iteration 2 explicitly standardized the phrasing, change the diagram text to "2+ substantive decisions emerge" so the quick-skim workflow does not preserve the old approximate threshold.

## Notes

`openspec validate bootstrap-openspec-orbit` passes. I treated older `.orbit-runs/external-prompt-*.md` files as historical artifacts, per the prompt, and did not flag stale wording inside them. Pass 8 now has a usable operational scenario, and the `@review:` hits I saw in active artifacts were documentation/examples rather than unresolved review markers.
