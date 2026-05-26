# External Review: harden-review-mode-recommendations (iteration 1, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone `main`; confirm the latest commit matches your review subject before you start).

## Meta-context (special framing for this change)

**This change modifies orbit's review skill itself.** The change updates `/opsx:review --as system`'s final-assessment recommendations (the very mechanism you'd use to review this change in system mode) and adds a new `Review mode decision framework` requirement to `orbit-conventions`. Be alert to:

- **Self-referential correctness**: do the modified spec scenarios cohere with the rest of orbit-review's existing baseline? Look for places the new logic might contradict unchanged scenarios.
- **Dogfood validity**: this is also the first cycle where the user is invoking external review against a change about to make external review the default. The recommendation logic this change introduces is the very pattern the user is following.

## Project context (read first)

- `CLAUDE.md` — orbit's project orientation. Pegging strategy + execution disciplines.
- `README.md` — user-facing docs. The change will ADD a "Choosing a review mode" section here; verify the proposed placement (per task 2.1) makes sense relative to existing structure (currently `## Command reference` L238-481 immediately before `## The external review cycle` L482; the new section would slot between).
- `openspec/specs/orbit-review/spec.md` — current baseline. The MODIFIED requirement `Final assessment phrasings depend on mode` lives at L130; the change rewrites it with iteration-aware logic for system-mode.
- `openspec/specs/orbit-conventions/spec.md` — current baseline. The ADDED requirement `Review mode decision framework` is new (no name collision should exist; verify).
- `openspec/specs/orbit-run-summary-emit/spec.md` — relevant because the iteration-aware logic depends on `external-system-*.md` and `review-system-*.json` filename conventions; the filename `<TS>` token format is defined in `orbit-conventions` `Internal-run JSON summary format`.
- `openspec/changes/harden-review-mode-recommendations/.orbit-runs/` — iteration history. Includes propose T0, review-proposal iter-1 (in-context Claude --fresh subagent: 0 critical + 3 warning + 2 suggestion), and address-reviews iter-1 (3 resolved + 1 deferred + 1 stale-suppressed).

## Cycle context

- **Iteration**: 1 (first external review for this change in any mode)
- **Prior internal findings still open**: 0. iter-1 internal review's 3 warnings + 2 suggestions were walked via address-reviews iter-1 — 3 resolved with material fixes (timestamp source pinned to filename `<TS>` token; README placement named exactly; new edge-case-assumptions scenario added); 1 SUGGESTION deferred per reviewer's own recommendation (orbit-conventions baseline meta-convention asymmetry); 1 SUGGESTION stale-suppressed per reviewer's own recommendation (T0 emit off-by-one count, immutable history).
- **Prior external findings still open**: N/A (first external review)
- **Resolved since last review (iter-1 → iter-2 state)**:
  - WARN 1: spec.md scenarios for "converged clean" + "external stale" — pinned timestamp comparison to filename `<TS>` token (per orbit-conventions `Internal-run JSON summary format`). Affects 2 scenarios.
  - WARN 2: tasks.md task 2.1 README placement — named exact placement (after `## Command reference`, before `## The external review cycle`) instead of "End-to-end workflow" (which doesn't exist as a section name).
  - WARN 3: new `Edge-case assumptions for the iteration-aware logic` scenario added to specs/orbit-review/spec.md — documents 3 v1 assumptions (presence regardless of content; multiple-files tie-break via filename token lexical sort; unparseable filename token treated as absent). Scenario count in orbit-review delta: 9 → 10.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as proposal)

Mandatory artifacts (under `openspec/changes/harden-review-mode-recommendations/`):

- `proposal.md` — Why + What Changes + Capabilities + Impact
- `design.md` — 6 decisions (D-scope-1, D-logic-1, D-ux-1, D-docs-1, D-mention-fresh, D-empirical-citation) + 5 risks/trade-offs
- `tasks.md` — 3 chunks (14 tasks total — note: I'd accept 13 or 14 depending on whether you count 1.x as 5 sub-tasks; verify)
- `specs/orbit-review/spec.md` — MODIFIED `Final assessment phrasings depend on mode` with 10 scenarios (3 proposal-mode + 6 system-mode + 1 edge-case + 1 audit-friendly + 1 non-gate)
- `specs/orbit-conventions/spec.md` — ADDED `Review mode decision framework` with 5 scenarios
- `explore.md` — historical record from explore phase (7 decisions D1-D7, all resolved via batch user-agreement on the 5 open questions)

Baseline + archive context:

- `openspec/specs/orbit-review/spec.md` — `Final assessment phrasings depend on mode` (L130 — the MODIFIED target). Verify the MODIFY-by-name match: the change's MODIFIED block must include the FULL updated requirement content per OpenSpec MODIFY semantics; the current baseline's 5 scenarios are replaced by the change's 10 scenarios.
- `openspec/specs/orbit-conventions/spec.md` — `Internal-run JSON summary format` (L185-area; defines the filename `<TS>` token format); `Final-assessment phrasings` (L358-area; lists 3 base stock-phrasing templates — note: this is a different requirement than orbit-review's `Final assessment phrasings depend on mode`. Reviewer iter-1 flagged this as a soft drift in SUGG 1, deferred. Confirm the deferral is reasonable).

What the change actually does (one-paragraph summary):

Updates `/opsx:review --as system`'s final-assessment recommendations to nudge toward external review (or `--fresh`) before archive when no external system review has run yet for the change. Iteration-aware logic stops nudging once external converges clean (compared via filename `<TS>` token). Cites bootstrap-orbit-status-cli's 3-of-3 empirical evidence in the recommendation prose. Adds a new `Review mode decision framework` requirement to orbit-conventions codifying when in-context vs `--fresh` vs external is appropriate. Also adds a "Choosing a review mode" section to README. Proposal-mode review's default-recommendation is **unchanged** (the empirical evidence is system-mode specific). Closes [#20](https://github.com/las-sal/openspec-orbit/issues/20) + [#13](https://github.com/las-sal/openspec-orbit/issues/13).

## What to look for (9 proposal-mode passes)

1. **Structure & Delta Integrity** — required artifacts present; ADDED/MODIFIED valid; `openspec validate --strict` would pass.
2. **Internal Coherence** — proposal aligns with design aligns with specs aligns with tasks. Decision-to-scenario coverage; no scope creep.
3. **Cross-Doc Coherence** — `CLAUDE.md`, README, and the existing orbit-review SKILL/command files remain consistent after this change applies. Spot-check that the new system-mode phrasings don't contradict existing review-skill behavior described in other places.
4. **Archive Consistency** — ADDED requirement name (`Review mode decision framework`) doesn't collide with any baseline requirement; MODIFIED block includes the FULL updated content.
5. **Codegen Readiness** — the iteration-aware logic in spec scenarios is implementable by two different reviewers without divergence. The edge-case-assumptions scenario should resolve most ambiguity; flag any remaining underspecification.
6. **Gap Hunt** — what failure modes does the iteration-aware logic still miss after the WARN 3 fix? E.g., does it cover the case where `external-system-*.md` exists but the matching `address-reviews-*.json` shows the external findings weren't actually resolved?
7. **Drift Hunt** — recommendation prose appears in spec scenarios; will appear in SKILL body + README section per tasks 1.1/2.1. Verify the propagation tasks correctly cite the spec as canonical.
8. **Inline Review Marker Residue** — any `@review:` markers still present? (Should be 0 — pushback verified in address-reviews iter-1.)
9. **Pre-Handoff Sweep** — anything else before shipping? Awkward wording, contradictions, examples that don't match prose.

## Areas worth extra scrutiny

This change has self-referential complexity. Spend extra rigor on:

1. **The cited empirical evidence** — the spec scenarios + recommendation prose cite "in-context system review missed 3 of 3 real bugs in bootstrap-orbit-status-cli's first archived cycle." This is in a sibling repo (`las-sal/orbit-status`). You can't verify this from inside this repo; treat it as illustrative-not-load-bearing per design.md's `Risks / Trade-offs` section (which acknowledges citation-rot risk). Don't re-flag as a critical issue.

2. **The new edge-case-assumptions scenario** — added during address-reviews iter-1 to address WARN 3. Verify it actually covers the iteration-aware logic's edge cases (corrupted files, multiple files, unparseable tokens) without introducing new ambiguity.

3. **The recommendation prose specifics** — spec.md scenarios for the new system-mode rows have exact stock phrasings in the THEN clauses (long quoted strings). Verify these phrasings are well-formed (no broken markdown, no contradictions, no internal references that would rot).

4. **The asymmetry between proposal-mode and system-mode** — the change explicitly does NOT modify proposal-mode behavior. Is this asymmetry well-justified by the empirical evidence + design.md's reasoning, or does it create user-facing confusion?

5. **The README section that doesn't exist yet** — task 2.1 prescribes adding "Choosing a review mode" between `## Command reference` and `## The external review cycle`. Verify the prescribed content (per task 2.2) would coherently land in that location given the existing README structure.

## Output format — write to:

`openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-proposal-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format — pick a fresh one so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: harden-review-mode-recommendations (iteration 1)

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
   git add openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 1): harden-review-mode-recommendations

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
