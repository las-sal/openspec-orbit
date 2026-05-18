# External Review: bootstrap-openspec-orbit (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-18

## CRITICAL

None.

## WARNING

### Review-external prompt delivery contract is still split between chat-paste and file-backed modes
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-external/spec.md:10
**Description**: The first scenario still says `/opsx:review-external` "generates a self-contained markdown prompt and emits it to chat for the user to copy", while the later normative requirement says the full prompt is written to `.orbit-runs/external-prompt-<as>-<TS>.md` and chat emits only a short invocation snippet. The same stale chat-paste wording remains in `design.md:146` and `sketches/review-external.md:82`. This is exactly the kind of self-referential workflow drift that can make codegen implement the wrong handoff surface. Pick the file-backed contract, revise the earlier scenario/design/sketch language to match it, and keep "full prompt in file; short snippet in chat" as the single source of truth.

### Snippet-only chat output conflicts with the required dirty-tree warning
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-external/spec.md:52
**Description**: The chat invocation scenario says chat output contains three things "and nothing else", but the repo-state validation requirement later requires chat output to include an uncommitted-changes warning. Those requirements cannot both be true when the repo is dirty. Make the warning an explicitly allowed fourth item, or define that the warning precedes the three-item snippet, so the generated skill has one deterministic output contract.

### address-reviews sketch contradicts v1 `--from-file` ingest
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/address-reviews.md:23
**Description**: The sketch says paste/file input sources are deferred to v2, even though the spec and README make `--from-file` a v1 requirement. The same stale model returns at line 255, saying external findings flow through the user manually dropping `@review:` markers. This undercuts the core cross-AI loop described by the proposal. Update the sketch so lean v1 includes `--from-file` throughout and only `--from-paste` remains deferred.

### review-external sketch omits the Inline Review Marker Residue pass from proposal-mode prompts
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/review-external.md:177
**Description**: The proposal-mode appendix lists only eight review-proposal checks and jumps from Drift Hunt to Pre-Handoff Sweep, omitting the required Inline Review Marker Residue pass. The actual generated prompt for this review includes all nine passes, and `orbit-review-external` requires proposal prompts to enumerate the nine review-proposal passes. Update the sketch's reference appendix so future prompt generation does not regress to the eight-pass version.

### README still contains an unresolved review note outside the canonical marker syntax
**File**: README.md:212
**Description**: The README includes `@review - not sure this makes sense? ...` in active user-facing content. Because it is not `@review:`, `/opsx:address-reviews` will not discover it, but humans will read it as unresolved review residue. Either resolve the question in prose or convert it to a canonical `@review:` marker and run the resolution flow before apply.

### README links still point to the pre-propose explore staging path
**File**: README.md:55
**Description**: The README points readers to `openspec/explore/bootstrap-openspec-orbit/explore.md`, and the Design Records section repeats the same `openspec/explore/...` paths. In the current repo, the exploration has already been promoted under `openspec/changes/bootstrap-openspec-orbit/`; there is no `openspec/explore/` directory. These broken links make the dogfooded example look inconsistent with its own "propose moves staging to changes" rule. Update the current-state README links/status to `openspec/changes/bootstrap-openspec-orbit/...`, optionally noting that `openspec/explore/<name>/` is the pre-propose staging location.

## SUGGESTION

### Archive skip-audit scenario has a typo in normative text
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-archive-modifications/spec.md:24
**Description**: The scenario says "Pass-archive audit is skipped"; this appears to mean "Pre-archive audit is skipped." Fixing the typo avoids giving codegen or future reviewers a phantom term to reconcile.

## Notes

`openspec validate bootstrap-openspec-orbit` passes. I did not flag historical rename vocabulary when it appeared only in clearly rejected or superseded exploration context; the findings above are current-state contradictions in README, design, specs, or command sketches that an implementer is expected to read.
