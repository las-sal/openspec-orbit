# External Review: lean-overlay-and-add-orbit-onboard (iteration 1, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone fresh; the change is on `main` after commit `29c0f2d` or later — pull latest before reading)

## Project context (read first)

- `CLAUDE.md` — orbit's authoring conventions (three execution disciplines: read-before-reference / change-completeness / pushback). NOTE: this file is CURRENTLY pre-pegging-framed; this change rewrites it (per tasks chunk 2). Read for current state.
- `README.md` — orbit's user-facing surface and install model (CURRENTLY pre-pegging-framed; this change rewrites it). The install section L907-1012 area is what this change overhauls.
- `openspec/specs/orbit-conventions/spec.md` — baseline cross-cutting orbit conventions. This change MODIFIES the `Distribution model — overlay, not CLI fork` requirement (currently at line 602) and ADDS two new requirements (`Upstream version pinning`, `Overlay file disposition`).
- `openspec/specs/orbit-*/spec.md` — other established orbit capability specs (8 capabilities total; orbit-onboard is brand-new in this change).
- `openspec/changes/archive/2026-05-21-emit-run-summary-jsons-from-workflow-commands/` — the prior change (just archived 2026-05-21) that added `# Orbit additions` sections to 7 upstream-bodied skills, which is the proximate cause of THIS change's premise-invalidation finding (see Cycle context below). Read its `proposal.md` + `design.md` for the universal-spine + pegging context that informs this change's "everything is fine to bundle now" decisions.
- `openspec/changes/lean-overlay-and-add-orbit-onboard/explore.md` — decision history with 11 captured decisions including the strategic pivot to pegging.

## Cycle context

- **Iteration**: 1 (first external review for this change)
- **Authoring/internal-reviewer context (for cross-AI anchoring awareness)**: The proposal/design/specs/tasks artifacts AND the internal review iter-1 (11 findings) AND the address-reviews iter-1 walk were ALL authored by Claude Opus 4.7 in a single in-context session. The orbit-status retrospective evidence (`openspec-orbit#20`) documented that in-context system review missed 3/3 real bugs that external review caught. While this is proposal mode (less anchored than system mode), the same pattern of in-context anchoring may have suppressed findings. Your value as a fresh-context reviewer is exactly that — flag what the in-context AI might have anchored away from. Pay particular attention to the strategic decisions (D-arch-1 pegging, D-arch-3 narrative reframe) that the authoring AI may have over-committed to.
- **Prior internal findings still open**: 0 (all 11 from internal review iter-1 were resolved during the address-reviews iter-1 walk; see `address-reviews-2026-05-23T13-53-10Z.json` for the per-finding resolutions). Among those 11: 3 CRITICAL (F1 non-emission preservation, F2 README pre-pegging sweep, F3 cross-chunk coupling), 4 WARNING, 3 SUGGESTION, 1 stale-suppressed (F8).
- **Prior external findings still open**: 0 (this is the first external pass).
- **Resolved since last review**: 11 internal findings. Notable resolutions:
  - F3 (CRITICAL coherence) → user chose to reorder chunks 3→1→2 so feedback skill deletion happens FIRST, before chunks that update README references to it.
  - F5 (WARNING coherence) → user chose to split the canonical-flow walkthrough task into 4.4a/b/c (diagram + phases 1-4 / phases 5-9 / lens + external-review weave).
  - F7 (WARNING correctness) → user chose to DROP the origin column from the orbit-onboard quick-reference table spec — single source of truth is orbit-conventions `Overlay file disposition`.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as proposal)

- `openspec/changes/lean-overlay-and-add-orbit-onboard/proposal.md`
- `openspec/changes/lean-overlay-and-add-orbit-onboard/design.md`
- `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md`
- `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-conventions/spec.md` (MODIFIED + ADDED)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md` (ADDED — new capability)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/explore.md` (decision history with 11 captured decisions)
- `openspec/specs/orbit-conventions/spec.md` (archived baseline — to check the MODIFIED delta against)
- `openspec/specs/orbit-run-summary-emit/spec.md` (just-archived capability with the `Emit scope` requirement that scope-enforces /opsx:onboard non-emission)
- `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/review-proposal-2026-05-23T13-42-44Z.json` and `.orbit-runs/address-reviews-2026-05-23T13-53-10Z.json` — internal review + resolution log
- `.claude/skills/openspec-onboard/SKILL.md` — current state of the skill (upstream body + `# Orbit additions` scope-enforcement note); this is what gets replaced in chunk 4

## Cluster 2 origin context

This change addresses GH issues [#6](https://github.com/las-sal/openspec-orbit/issues/6) and [#23](https://github.com/las-sal/openspec-orbit/issues/23). #6 was filed PRE-pegging-decision; its original framing ("delete 9 unmodified upstream files to preserve upstream-update flow") was invalidated mid-explore because the just-archived emit change added `# Orbit additions` to 7 of those 9 skills. The user's strategic pivot to peg upstream `@fission-ai/openspec@1.3.1` collapsed #6's complexity — see design D-arch-1 and D-arch-2.

The pegging strategy is foundational: orbit becomes "a workflow tool that owns the .claude/ surface and uses openspec's CLI as a pinned engine" rather than "an overlay that augments cleanly". Future Option 2 work (deferred to a separate explore cycle) will drop the `# Orbit additions` pattern entirely. Option 3/4 (fork or replace upstream CLI) are deferred indefinitely.

## What to look for

Apply ALL 9 proposal-mode passes. For each finding you raise, cite the file + line + a specific recommendation. Don't be charitable.

1. **Structure & Delta Integrity** — required artifacts present; ADDED/MODIFIED/REMOVED/RENAMED operations used correctly; `openspec validate --strict` would pass. Specifically: does the orbit-conventions MODIFIED delta correctly copy the full original `Distribution model` requirement before editing (per the MODIFIED workflow rule)? Are the orbit-conventions ADDED requirements (`Upstream version pinning`, `Overlay file disposition`) properly named and uniquely scoped against the existing 27 baseline requirements?

2. **Internal Coherence** — proposal aligns with design aligns with specs aligns with tasks. Count/label consistency. Watch for: D-arch-1 / D-arch-3 strategic decisions actually reflected in spec deltas; the 11 explore decisions all covered somewhere; capability lists in proposal vs. spec dirs; chunk reorder (post-F3 resolution) consistent across the chunking comment + task numbering + cross-references in design.

3. **Cross-Doc Coherence** — `CLAUDE.md`, README, baseline specs. Watch for: design.md references to "post-D-arch-3 framing" — is the framing description in design.md actually concrete enough that an AI implementing chunk 2.1 (CLAUDE.md rewrite) could produce coherent output? Are the spec requirements' references to other requirements (cross-refs) all resolvable?

4. **Archive Consistency** — the MODIFIED orbit-conventions `Distribution model` requirement: does it preserve the existing baseline semantics (still describes "overlay, not CLI fork") while ADDING pegging-specific scenarios? Does it break any prior orbit conventions? Check for any orbit-conventions requirement that the new MODIFIED + ADDED might silently invalidate.

5. **Codegen Readiness** — could a fresh AI implement this change from these specs alone? Specifically check:
   - The orbit-onboard `Setup verification section` requirement — are the verification scenarios concrete enough that an implementing AI can write working setup-check logic? (e.g., "check for orbit-authored skill presence" — which skill? `openspec-review` is named in tasks.md but not in the spec.)
   - The `Canonical-flow walkthrough section` requirement — does it specify ENOUGH about each phase paragraph (length expectation, what must be covered) that the 4.4a/b/c sub-tasks can produce coherent output? Or is the spec too abstract for codegen?
   - The `Identity statement section` requirement — does it actually constrain the writing enough that two implementing AIs would produce comparable output?
   - The `Overlay file disposition` requirement in orbit-conventions — is the 4-category test crisp enough? Specifically the "no orbit-mission value" criterion (in the Not-shipped scenario for `feedback`) — is that decideable for future upstream skills?

6. **Gap Hunt** — what's missing?
   - The new `openspec-onboard` skill replaces the existing one. Is the migration path crisp? What about projects with prior `.claude/` overlays already applied?
   - Tasks 5.3-5.5 are user-validation tasks. Are they specific enough to actually validate the change, or do they just punt to "user reads the SKILL.md cold and confirms"?
   - The pegging strategy says upstream improvements don't auto-propagate. What's the documented process for the eventual upgrade-and-port change? Spec mentions "deliberate change-proposal event" but doesn't enumerate criteria for when to upgrade.
   - The Option 2 / Option 3-4 future-work mentions in design.md — are they tracked anywhere (issue, TODO, decision)? Or just narrative aspirations?
   - Multi-skill rename concern: D8 said "if a global openspec-* → orbit-* rename happens later, do it all at once" — is there a tracked issue for that future work?

7. **Drift Hunt** — old vocabulary lingering:
   - "overlay" terminology — tasks 2.3 says to sweep CLAUDE.md + README for it, but is the spec itself consistent? Check orbit-conventions MODIFIED `Distribution model` requirement title — it STILL says "overlay, not CLI fork" (kept for MODIFY-by-name match). Is that contradictory with the new body content?
   - "upstream-update" / "openspec update" references in the change — are any still implying users should run it for fresh upstream skills?
   - References to specific upstream version numbers — all `1.3.1`? Any drift?
   - References to issue numbers (#6, #23, #26) — accurate?

8. **Inline Review Marker Residue** — any `@review:` markers in the change directory? (Should be zero — this is post-address-reviews state.)

9. **Pre-Handoff Sweep** — anything else before this ships to apply?
   - Scope check: 11 deltas (1 MODIFIED + 2 ADDED in orbit-conventions; 8 ADDED in new orbit-onboard). Below OpenSpec's "consider splitting at >10" soft threshold barely. Is the bundling (pegging + onboard) justified? Or should pegging ship first as foundation, with onboard as follow-up?
   - 25 tasks across 5 chunks. Is the chunking sized appropriately for apply-chunk-end emit cadence? Any chunk that's too small (chunk 1 only has 2 tasks) or too large?
   - Tasks 4.4a/b/c (canonical-flow walkthrough) are the meatiest. Is the per-sub-task scope right, or should 4.4c (lens + external-review weave) be its own quality-checked task?
   - The change introduces a `bridge` markdown findings file (`review-proposal-...-as-findings.md`) committed in `.orbit-runs/` because `address-reviews --from-file` doesn't accept JSON yet (openspec-orbit#4). Is that bridge file appropriately scoped?
   - Has the user-validation pattern (tasks 5.3-5.5) been validated as effective in prior changes? (Reference: the just-archived emit change had 4 user-validation tasks that were closed "observationally" — same pattern here.)

## Output format

Write your findings to: `openspec/changes/lean-overlay-and-add-orbit-onboard/.orbit-runs/external-proposal-<TS>.md` (where `<TS>` is the timestamp YOU set when you write the file, in `YYYY-MM-DDTHH-MM-SSZ` format).

**Critical instructions for the output file**:

1. **Echo findings to chat as well** — write the same findings markdown to your chat output in addition to the file. This lets the user see your output without having to read the file separately.

2. **Commit and push the output file when done** — after writing the findings to the file, `git add` the file, commit with a message like `external review: lean-overlay-and-add-orbit-onboard iter-1 (proposal mode, <your model name>)`, and `git push`. This makes the findings ingestible by `/opsx:address-reviews --from-file` in the orbit repo.

**File format** (orbit external-review markdown):

```markdown
# External Review: lean-overlay-and-add-orbit-onboard (proposal-mode iter-1)

**Reviewer model**: <e.g., GPT-5 Codex 2025-12-01> | <e.g., Claude Opus 4.7>
**Findings count**: <N CRITICAL, M WARNING, K SUGGESTION>

---

## CRITICAL

### <ID1> — <short title>

**File**: <path>
**Line**: <line ref or N/A>
**Dimension**: <completeness | correctness | coherence>

<description of issue>

**Recommendation**: <specific actionable recommendation>

### <ID2> — <short title>
...

## WARNING

### <ID3> — ...

## SUGGESTION

### <ID4> — ...
```

Use IDs like `EC1, EC2, ...` (external CRITICAL), `EW1, EW2, ...` (external WARNING), `ES1, ES2, ...` (external SUGGESTION). One per finding.

Do NOT apply pushback. Report what you see; the orbit address-reviews walk handles pushback on ingest.
