# External Review: address-reviews-defaults-and-decision-forks (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-26

## CRITICAL

None.

## WARNING

### Archived change paths are only specified in a scenario, not the normative OUT list
**File**: openspec/changes/address-reviews-defaults-and-decision-forks/specs/orbit-address-reviews/spec.md:40
**Description**: The normative cross-change OUT bullet only names `openspec/changes/<other-name>/*` and parenthetically scopes that to "other active changes' directories." The later scenario says archive paths must also be skipped, but an implementer following the four bullets can treat `openspec/changes/archive/<date-name>/...` as IN unless they special-case the scenario or accidentally interpret `archive` as an active change name. Add an explicit archive prefix/category, such as `openspec/changes/archive/*`, or revise the cross-change bullet to cover both other active changes and archived changes.

### Safe-exclusion scenario reopens an otherwise fixed OUT list
**File**: openspec/changes/address-reviews-defaults-and-decision-forks/specs/orbit-address-reviews/spec.md:78
**Description**: The Option D framing depends on a small structural OUT list, but this scenario says cascade skips `.git/`, `node_modules/`, `dist/`, `build/`, "or another safe-exclusion path." That undefined phrase lets implementers invent extra excluded paths during codegen, which can reintroduce the ad hoc file/category exclusions the design rejected. Either remove the open-ended phrase and keep the four paths exact, or define the mechanism and evidence threshold for adding more safe-exclusion paths.

## SUGGESTION

### Accept both bold Options-prefix spellings in the heuristic
**File**: openspec/changes/address-reviews-defaults-and-decision-forks/specs/orbit-address-reviews/spec.md:154
**Description**: The proposal and prompt describe the strict heuristic signal as `**Options:**`, while this normative bullet lists `"**Options**:"` plus plain `"Options:"`. A literal implementation could miss external markdown where the colon is bolded with the word. List both bold variants, or define the detector after Markdown normalization so `**Options:**` and `**Options**:` are equivalent.

## Notes

`openspec validate address-reviews-defaults-and-decision-forks --strict` and `openspec validate --all --strict` both pass. I did not find unresolved inline `@review:` marker residue in the change artifacts; the remaining findings are codegen-readiness gaps around the Option D path-prefix contract and the decision-fork heuristic.
