# External Review: orbit-onboard-follow-up (iteration 1, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone the `main` branch; confirm latest commit hash matches what you're reviewing).

## Project context (read first)

- `CLAUDE.md` — orientation for the openspec-orbit repo. Orbit is a workflow tool that owns the `.claude/` surface and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine.
- `README.md` — user-facing docs. The Installation section (around L898-L1040) was rewritten in the just-archived `align-readme-install-with-v1-3-1` change to match v1.3.1's actual install surface (15 skills + 16 commands post-overlay). This change will need to update parts of that section per task 2.4 (overlay disposition shift).
- `openspec/changes/orbit-onboard-follow-up/.orbit-runs/` — iteration history. Includes propose T0 (2026-05-24T16:36:37Z), review-proposal iter-1 (2026-05-24T16:55:15Z, in-context Claude with `--fresh` subagent, 2 CRITICAL + 3 WARNING + 2 SUGGESTION), address-reviews iter-1 (2026-05-24T17:21:58Z, resolved 6 + deferred 1).
- `openspec/specs/orbit-conventions/spec.md` — current baseline. The change MODIFIES one requirement here (`Overlay file disposition`) and ADDs no new requirements. Verify the MODIFIED block preserves intent and the delta composes cleanly with existing baseline.
- `openspec/specs/orbit-run-summary-emit/spec.md` — baseline. The new `orbit-onboard` capability's `Non-emission of run-summary JSON` requirement composes with this baseline's `Emit scope` requirement. Verify composition.
- `openspec/lenses/` — does NOT exist; lens-dependent passes skip gracefully.
- `openspec/project.md` — does NOT exist.

## Cycle context

- **Iteration**: 1 (first external review for this change in any mode).
- **Prior internal findings still open**: 0. iter-1's 2 CRITICAL + 3 WARNING + 2 SUGGESTION were walked via address-reviews — 6 resolved with material fixes (4 explicit + 2 auto-resolved by dependency), 1 deferred per user decision (per-phase paragraph length unconstrained at spec level; tasks.md:35 already has author-time guidance; user-validation at task 3.2 catches outcomes).
- **Prior external findings still open**: N/A (first external review).
- **Resolved since last review (iter-1 → iter-2 state)**:
  - CRIT 1: spec.md:38 layered-✓ scenario counts updated from pre-state (10+4) to post-state (9+5+1)
  - CRIT 2: tasks.md gained task 2.4 — README install section sync per baseline `Install documentation describes actual install surface` `Upgrade and overlay-change proposals include README sync` scenario
  - WARN 3: spec.md:44 lumped enumeration extended to include wrong-upstream-version; message now points to entire `## Installation` section
  - SUGG 1: spec.md:130 metadata-note placement flexibility cited explicitly
  - WARN 1 + WARN 2: auto-resolved by CRIT 1 + CRIT 2 fixes respectively
  - SUGG 2 (deferred): per-phase paragraph length unconstrained at spec — accepted per user decision

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as proposal)

Mandatory artifacts (under `openspec/changes/orbit-onboard-follow-up/`):

- `proposal.md` — Why + What Changes + 1 new capability (`orbit-onboard`) + 1 modified capability (`orbit-conventions`) + Impact
- `design.md` — 7 decisions (D-onboard-1 through D-onboard-6 + D-conventions-1) + alternatives + risks
- `tasks.md` — 3 chunks (16 tasks total post-address-reviews-iter-1; chunk 2 has 4 tasks now including the added 2.4 README-sync task)
- `specs/orbit-onboard/spec.md` — NEW capability: 10 ADDED requirements + 28 scenarios
- `specs/orbit-conventions/spec.md` — MODIFIED `Overlay file disposition` requirement (8 scenarios; added a `ff.md` / `fast-forward.md` naming-divergence scenario; updated `Orbit-authored` and `Upstream-required primitive` scenarios)
- `explore.md` — historical record from explore phase (5 decisions D1-D5; 7 resolved questions; the explore established 80% of the design carries forward from the cut `lean-overlay-and-add-orbit-onboard`'s archived `explore.md` D8-D11)

Baseline + archive context:

- `openspec/specs/orbit-conventions/spec.md` — current baseline. Specifically: `Distribution model — pegged engine + orbit-owned surface` (L602), `Upstream version pinning` (L626), `Overlay file disposition` (L655 — this is the one MODIFIED), `Install documentation describes actual install surface` (L708 — the one CRIT 2 fix complies with).
- `openspec/specs/orbit-run-summary-emit/spec.md` — `Emit scope` requirement lists `/opsx:onboard` as non-emit; new spec's `Non-emission of run-summary JSON` requirement composes.
- `openspec/changes/archive/2026-05-24-lean-overlay-and-add-orbit-onboard/` — the change whose chunks 4-5 (orbit-onboard SKILL body) deferred to this change. Its `explore.md` D8-D11 carry forward as inherited design (referenced by this change's design.md and explore.md).
- `openspec/changes/archive/2026-05-24-align-readme-install-with-v1-3-1/` — the immediately prior archived change. Its archived design + spec ADDED the `Install documentation describes actual install surface` requirement that THIS change's task 2.4 complies with. Worth reading for context on why README sync is required for disposition-shifting changes.
- `openspec/changes/archive/2026-05-21-emit-run-summary-jsons-from-workflow-commands/` — the change that established `orbit-run-summary-emit` `Emit scope` (which lists `/opsx:onboard` as non-emit).

What the change actually does (one-paragraph summary):

Replaces the upstream-bodied `openspec-onboard/SKILL.md` + matching `opsx/onboard.md` with 100% orbit-authored 5-section reference-leaning hybrid content (Setup verification / Identity / Canonical-flow walkthrough / Quick-reference / Try-it nudge). Adds a new `orbit-onboard` capability spec defining the SKILL behavior. Modifies `orbit-conventions` `Overlay file disposition` to (a) add openspec-onboard to Orbit-authored category, (b) name openspec-sync-specs as Upstream-required-primitive's concrete example, (c) acknowledge `ff.md`/`fast-forward.md` naming divergence. Task 2.4 (added during address-reviews iter-1) syncs README to match. Closes [#23](https://github.com/las-sal/openspec-orbit/issues/23). Files [#29](https://github.com/las-sal/openspec-orbit/issues/29) for future cross-command body-deduplication refactor (NOT bundled here per design D-onboard-5).

## What to look for (9 proposal-mode passes)

1. **Structure & Delta Integrity** — required artifacts present; ADDED/MODIFIED/REMOVED/RENAMED valid; `openspec validate --strict` would pass.
2. **Internal Coherence** — proposal aligns with design aligns with specs aligns with tasks. Count/label consistency. No scope creep. Every spec requirement has a corresponding task; every task ties to a spec requirement; no decision lacks spec representation.
3. **Cross-Doc Coherence** — `CLAUDE.md` post-apply state still accurate; baseline cross-references resolve (`Upstream version pinning`, `Distribution model — pegged engine + orbit-owned surface`, `Overlay file disposition`, `Emit scope`); the README sync task 2.4 covers all sites that will become stale.
4. **Archive Consistency** — ADDED requirements don't contradict baseline; MODIFIED `Overlay file disposition` preserves the existing 7 scenarios' intent while updating per the disposition-shift; no name collisions.
5. **Codegen Readiness** — no implicit requirements; no ambiguity. Two implementers writing the SKILL body should produce equivalent content. Watch for hand-waving like "as appropriate" / "clearly" / "obviously" / unconstrained ranges.
6. **Gap Hunt** — could a fresh AI implement this from these specs alone? Unstated assumptions? Error paths the verification logic misses? State transitions (hard-stop ↔ warn ↔ pass) all specified?
7. **Drift Hunt** — old vocabulary lingering? Specifically: the new spec forbids augmentation language ("augments cleanly", "overlay on top of", "layered on top of", "extends upstream") in the Identity section per the `Identity section MUST NOT contain augmentation language` requirement. Verify the change's OWN artifacts (proposal, design, tasks, explore) don't use that vocabulary — meta-drift would be embarrassing.
8. **Inline Review Marker Residue** — any `@review:` markers still present in change artifacts? (Should be 0 — address-reviews iter-1 resolved all.)
9. **Pre-Handoff Sweep** — anything else before shipping? Awkward wording, count inconsistencies, references that don't resolve, examples that don't match prose.

## Areas worth extra scrutiny

This is a large change (10 new spec requirements + 28 scenarios + 16 tasks + 1 modified baseline requirement). Spend extra rigor on:

- **CRIT 2 fix verification (Pass 6 + Pass 3)**: task 2.4 was added during address-reviews iter-1 to fix a CRITICAL finding. Verify it's complete: does it cover all README sites that will become stale? The original review identified L955, L956, L957, L972, L973 + Prerequisites + a residue sweep. Spot-check the README at those line ranges yourself; flag any sites the task missed.
- **The 9-vs-10 count shift everywhere it appears**: openspec-onboard moves Orbit-modified → Orbit-authored. Modified-skill count drops from 10 to 9; orbit-authored count rises from 4 to 5. Verify every place these counts appear in change artifacts uses the post-state numbers (9, 5, 14 — wait, total skills stays 15; total commands stays 16). Also verify task 2.4's enumeration of README lines actually matches current README content.
- **Composition with baseline (Pass 4)**: the new spec's `Non-emission of run-summary JSON` requirement composes with baseline `orbit-run-summary-emit` `Emit scope`. The new `Slash-command surface /opsx:onboard` requirement composes with baseline `orbit-conventions` `Verb-prefix naming taxonomy preserved`. Verify these compositions don't create contradictions or duplicate-spec issues.
- **Lumped messaging spec (Pass 5)**: spec.md:43-44 was extended during address-reviews to lump "wrong upstream version" with overlay-incomplete sub-modes. The lumped message now points to the README's entire `## Installation` section. Is this lumping coherent with the failure-mode taxonomy in design.md D-onboard-3? Or does it muddy the user-facing remediation message?

## Output format — write to:

`openspec/changes/orbit-onboard-follow-up/.orbit-runs/external-proposal-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format — pick a fresh one so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: orbit-onboard-follow-up (iteration 1)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(For each additional CRITICAL finding, repeat the `### <Title>` + `**File**:` + `**Description**:` triple. Use `None.` as a single-word body if there are no findings at this severity.)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(Same shape as CRITICAL. Use `None.` if no findings.)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(Same shape. Use `None.` if no findings.)

## Notes

<Optional: overall impression, broader concerns. Omit the whole `## Notes` section if you have nothing to add.>
```

If your environment doesn't support file writes (chat-only interface), output the markdown directly and the user will save it.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize — the chat output is the immediately-visible read for the user (they should be able to evaluate every finding without opening the file). The file remains the canonical record for `--from-file` parsing.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/orbit-onboard-follow-up/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 1): orbit-onboard-follow-up

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
