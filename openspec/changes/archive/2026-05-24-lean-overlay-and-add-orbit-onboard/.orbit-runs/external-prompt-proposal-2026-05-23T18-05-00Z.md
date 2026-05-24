# External Review: lean-overlay-and-add-orbit-onboard (iteration 2, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone fresh; the change is on `main` after commit ~`b356d97` or later — pull latest before reading; specifically need the iter-4 commit that includes the EW1-reversal)

## Project context (read first)

Same as iter-1 prompt — read CLAUDE.md, README.md, baseline orbit-conventions / orbit-run-summary-emit / other orbit-* spec files, the just-archived `emit-run-summary-jsons-from-workflow-commands` change at `openspec/changes/archive/2026-05-21-emit-run-summary-jsons-from-workflow-commands/`.

**NEW context since iter-1**: Two project memories were saved that frame orbit's strategic position:
- `orbit-pegged-to-openspec-1-3-1` — strategic pegging decision; no upstream auto-ingest; sequenced Option 1→2→3/4
- `orbit-supports-full-openspec-1-3-1` — default scope: keep upstream features unless real reason to drop; verify deprecation claims against current upstream state

These memories are in your session context (if the harness loads them) but if not, they're documented at: `/Users/sal/.claude/projects/-Users-sal-code-openspec-orbit/memory/`.

## Cycle context

- **Iteration**: 2 (second external review for this change; iter-1 was you (Codex) on 2026-05-23T14:06:15Z)
- **You found in iter-1**: 2 CRITICAL (EC1 feedback pruning install-vs-repo gap, EC2 onboard.md command body keeps guided tour), 3 WARNING (EW1 sync-specs disposition contradicts existing specs, EW2 Setup verification non-stable artifacts, EW3 disposition spec only classifies skills), 2 SUGGESTION (ES1 Option 2/global rename need tracking handles, ES2 Three categories listed as four)
- **Since iter-1, three more cycles ran**:
  - **address-reviews iter-2 (in-context Claude resolved your 7 findings)**: EC1 resolved with README rm steps; EC2 resolved with explicit task 4.7 rewrite; **EW1 resolved with /opsx:sync command-file deletion + sweep of "deprecated" framing across baseline specs**; EW2 resolved with stable post-install artifacts; EW3 resolved with command-disposition scenarios; ES1 filed openspec-orbit#27 + #28; ES2 fixed "Three → Four"
  - **review iter-2 (fresh-context Claude subagent)**: 1 CRITICAL (FF1: third stale #6 reference at archive-summary-schema.md:18 — you implicitly named 3 locations; iter-2 missed 1) + 3 WARNING + 3 SUGGESTION
  - **address-reviews iter-3**: 4 decisions resolved including FF1 fix, FF2 RENAMED for Distribution model title, FF4 mandatory sandbox install, FF5/6/7 trivial fixes
  - **review iter-3 (--thorough in-context Claude)**: 0 CRITICAL + 2 WARNING (TR1 sandbox undefined, TR2 archive→sync-specs dependency not anchored) + 2 SUGGESTION (TR3 subjective Identity framing, TR4 Try-it nudge assumes idea)
  - **address-reviews iter-4 (THIS IS THE BIG ONE)**: User pushback caught a propagated false claim — the original "deprecated upstream" framing for /opsx:sync was VERIFIED FALSE against current upstream docs (https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md describes /opsx:sync as "Optional command" not deprecated). **EW1 was REVERSED**: /opsx:sync command-file retained; all spec language about "primitive only" / "deprecation" / "pruning sync.md" simplified or removed. PLUS resolved iter-3's TR1/TR3/TR4 as trivial fixes; TR2 accepted-as-is per "don't write requirements around it".
- **Prior internal findings still open**: 0
- **Prior external findings still open**: 0 (your iter-1 7 findings all resolved, though EW1's resolution was reversed in iter-4)

**Authoring/internal-reviewer context (for cross-AI anchoring awareness)**: 4 prior review cycles all anchored on the propagated false "deprecated upstream" claim. Only user-initiated pushback against upstream docs caught it. The reversal in iter-4 is structural — `/opsx:sync` survives; multiple spec language simplifications applied. **Your specific value this iter-2**: cross-check whether the reversal is correct AND whether iter-1 → iter-4 net work introduced any new drift or removed too much in the simplification pass.

## What to read for THIS review (--as proposal iter-2)

Same files as iter-1, plus:
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/external-proposal-2026-05-23T14-06-15Z.md` (your iter-1 findings)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/address-reviews-2026-05-23T14-31-46Z.json` (in-context Claude's resolution of your iter-1)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/review-proposal-2026-05-23T14-38-05Z.json` (fresh-context Claude iter-2 findings)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/address-reviews-2026-05-23T13-53-10Z.json` (resolution of in-context iter-1)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/address-reviews-2026-05-23T14-48-08Z.json` (resolution of fresh-context iter-2)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/review-proposal-2026-05-23T14-54-09Z.json` (--thorough iter-3 findings)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/address-reviews-2026-05-23T18-03-25Z.json` (**THIS is the iter-4 reversal — read carefully**)

## What to look for (iter-2 specific)

**Primary concern (validate iter-4 reversal)**:

1. **Is the EW1 reversal correct?** Check that:
   - The user's pushback claim is verifiable (`/opsx:sync` IS listed in upstream docs as "Optional command")
   - The reversal didn't go too far (did anything legitimately need to be pruned that's now retained?)
   - The reversal didn't miss anything that should still be reversed (any remaining "deprecated"/"primitive only"/"pruned" language in spec deltas or design.md or proposal.md that should also be cleaned up?)
   - Cross-references between spec deltas remain coherent after the language simplification (e.g., orbit-conventions Internal-run JSON summary format sync_specs parenthetical → does it still tell readers anything useful?)

2. **Spec language simplification pass**: iter-4 simplified language in 4 places (orbit-conventions Internal-run JSON summary format, orbit-conventions Overlay file disposition Commands scenario, orbit-run-summary-emit Emit scope, orbit-onboard Verify pruned upstream files are absent). Verify none of these simplifications dropped load-bearing content.

3. **Apply all 9 proposal-mode passes again on the now-substantially-different proposal** (same pass list as iter-1):

   1. Structure & Delta Integrity (does the RENAMED requirement from FF2 still work cleanly? both MODIFIED requirements still match their baselines exactly?)
   2. Internal Coherence (do design.md items 7-11 from iter-4 align with the actual spec/tasks changes?)
   3. Cross-Doc Coherence (post-reversal, does anything still reference "sync.md is pruned" or "primitive only" in stale ways?)
   4. Archive Consistency (will iter-4's spec simplifications archive cleanly?)
   5. Codegen Readiness (specifically: is task 3.4's "fresh sandbox" definition now concrete enough?)
   6. Gap Hunt (did iter-4's TR2 "accepted as-is" decision leave a real gap, or is the dependency truly implicit-enough under pegging?)
   7. Drift Hunt (post-reversal: any remaining "deprecated" / "removal" / "transitional" language that should have been swept?)
   8. Inline Marker Residue (should still be 0)
   9. Pre-Handoff Sweep (scope check: 28 tasks across 5 chunks, 14 deltas — still bundling-justified?)

**Specific anchor-break questions**:

- The iter-4 reversal was triggered by user pushback, not by review. Are there OTHER propagated false claims in the proposal that no review cycle has caught? Apply pushback to specific claims: "deprecated", "transitional", "primitive only", "obsolete", "slated for removal" — verify each against current upstream state if you can.
- The 14 deltas now include 1 RENAMED + 2 MODIFIED + 2 ADDED in orbit-conventions, 8 ADDED in orbit-onboard, 1 MODIFIED in orbit-run-summary-emit. Is the orbit-run-summary-emit delta still earning its keep after iter-4 simplifications, or could it shrink further?
- design.md is now 11 items in D-conventions-1 (items 1-11). Heavy decision section. Does it still read coherently as one design narrative, or has accretion made it confusing?

## Output format

Same as iter-1: write findings to `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/external-proposal-<TS>.md` (timestamp YOU set when writing). Echo findings to chat. Commit + push.

Use IDs like `EC1', EC2', ...` (iter-2 critical), `EW1', ...` (iter-2 warning), `ES1', ...` (iter-2 suggestion) — prime suffix marks iter-2 to distinguish from your iter-1 finding IDs.

Don't apply pushback discipline. Report what you see; orbit address-reviews handles pushback on ingest.
