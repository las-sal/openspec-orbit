# External Review: orbit-onboard-follow-up (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-24

## CRITICAL

### README sync task contradicts required sandbox verification
**File**: openspec/changes/orbit-onboard-follow-up/tasks.md:77
**Description**: Task 2.4 explicitly says "No fresh-sandbox verify required" even though the same task modifies the README Installation section, and the archived baseline requirement `Install documentation describes actual install surface` says README install/update/uninstall changes SHALL include an apply-time fresh-sandbox verification task. That baseline scenario is unconditional at `openspec/specs/orbit-conventions/spec.md:722-725`; it does not exempt "derived count only" README edits. Either add a lightweight fresh-sandbox verification task for the rewritten install surface after task 2.4, or formally modify the baseline requirement to permit this exception. As written, the proposal asks apply to violate an archived SHALL.

### Setup verification omits required command checks
**File**: openspec/changes/orbit-onboard-follow-up/tasks.md:22
**Description**: The new `orbit-onboard` spec requires Section 1 to hard-stop when the user's `.claude/` state is missing any required orbit-shipped skill or command (`specs/orbit-onboard/spec.md:26-29`), and the pass scenario references the documented post-install surface of 15 skills + 16 commands (`specs/orbit-onboard/spec.md:36-39`). The implementation task, however, only tells the author to check skills: five orbit-authored skill files, `openspec-sync-specs`, `# Orbit additions` count, and `feedback/` absence. A partial overlay with all skills present but missing `.claude/commands/opsx/review.md`, `review-external.md`, `audit-drift.md`, `address-reviews.md`, `onboard.md`, `sync.md`, or `fast-forward.md` would pass the specified checks while the walkthrough proceeds to commands the user cannot run. Add command presence/count checks to task 1.2 (at minimum the orbit-shipped command files, and preferably the expected 16-file post-install command surface including upstream `ff.md`), or narrow the spec so it no longer promises command verification.

## WARNING

### README sync task misses stale count sites and uses too-narrow residue grep
**File**: openspec/changes/orbit-onboard-follow-up/tasks.md:76
**Description**: Task 2.4 covers the main install bullets and table rows, but several current README install-section sites will become stale when `openspec-onboard` moves to Orbit-authored and are not explicitly named: partial adoption still says "four new orbit-authored commands" and "four new SKILL.md directories" plus "All 10 upstream skills" (`README.md:985`), customization warnings still say "any of the 10 orbit-modified upstream skills" and enumerate `openspec-onboard` (`README.md:994`), the skipped-step gotcha still says "All 10 orbit-modified skills" (`README.md:999`), and uninstall comments still say 4 orbit-authored commands/skills and restore 10 orbit-modified skills (`README.md:1036`, `README.md:1040`, `README.md:1048`). The proposed residue grep only catches numeric `4 orbit-authored` / `10 upstream-modified` forms, so it will miss spelled-out "four new", "All 10", and "any of the 10" residues. Add explicit task bullets for these sites and broaden the final grep so the README synchronization is actually review-proof.

### `fast-forward.md` command disposition is internally inconsistent
**File**: openspec/changes/orbit-onboard-follow-up/specs/orbit-conventions/spec.md:35
**Description**: The modified command-disposition scenario omits `opsx/fast-forward.md` from both the orbit-authored examples and the orbit-modified examples, but the new naming-divergence scenario immediately below classifies `.claude/commands/opsx/fast-forward.md` as "orbit-authored, in the Orbit-authored category" (`specs/orbit-conventions/spec.md:39`). That also conflicts with task 2.4's README plan, which expects 5 orbit-authored commands after adding `onboard.md` while leaving `fast-forward.md` in the separate "2 additional orbit commands" bucket. Pick one disposition for `fast-forward.md` and make the spec delta plus README-sync task use the same category/counts; otherwise archive sync will bake a category contradiction into the baseline.

## SUGGESTION

### Clarify whether quick-reference excludes upstream `ff.md`
**File**: openspec/changes/orbit-onboard-follow-up/specs/orbit-onboard/spec.md:96
**Description**: The quick-reference scenario says it lists commands "per the verified-at-install-time state" but enumerates 15 commands and omits upstream `ff.md`, while the README post-install state has 16 command files because upstream `ff.md` coexists with orbit's `fast-forward.md`. If the onboard quick-reference intentionally lists only orbit-shipped command files from this repo, say that instead of "verified-at-install-time state"; if it is meant to describe the user's installed slash-command surface, include `/opsx:ff` or explain why the upstream alias is omitted.

## Notes

Reviewed `main` at `af33191c2617d11e0039e8eae182a5526acbb98f`; `git ls-remote origin refs/heads/main` returned the same commit. `openspec validate orbit-onboard-follow-up --strict` passes. `openspec/lenses/` does not exist, so lens-dependent proposal review context has no project-specific lens files to consult.
