# External Review: harden-review-mode-recommendations (iteration 2, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone `main`; confirm the latest commit matches your review subject before you start).

## Iter-2 context (special — substantive design revision since iter-1)

Iter-1 external review (GPT-5 Codex) found a CRITICAL: the iteration-aware convergence logic could mark unresolved external findings as "converged" because it only checked file presence + timestamp ordering, never the external findings' actual content or resolution status.

The address-reviews iter-2 walk responded with a **substantive design revision**: expanded the spec from 3 convergence states to 5, with two-path convergence detection (Path A: parse external markdown for clean content; Path B: inspect address-reviews JSON for resolution). orbit-review now inspects 3 file types beyond `review-system-*.json`: external markdown content, address-reviews JSON `resolution_summary`, and `apply-*.json` filename tokens for stale detection.

**This iter-2 review specifically validates the design revision.** Two recommended session paths:

- **Same-AI continuation (you, GPT-5 Codex)** — verifies your iter-1 CRITICAL was actually fixed correctly + you can spot if the 5-state model has any new structural flaws you caught flavors of in iter-1.
- **Fresh different AI (Claude in a new session)** — orthogonal eyes; better at catching net-new issues introduced by the design revision, weaker on verifying iter-1 fix integrity.

For this iter, either is valid; the user has chosen to send to whichever AI they're addressing this prompt to.

## Meta-context (carried from iter-1)

This change modifies orbit's review skill itself. Be alert to self-referential coherence — do the new convergence-state scenarios cohere with the rest of orbit-review's existing baseline?

## Project context (read first)

- `CLAUDE.md` — orbit's project orientation. Pegging strategy + execution disciplines.
- `README.md` — user-facing docs. The change will ADD a "Choosing a review mode" section between `## Command reference` and `## The external review cycle`.
- `openspec/specs/orbit-review/spec.md` — current baseline. The MODIFIED requirement `Final assessment phrasings depend on mode` lives at L130; the change now rewrites it with **14 scenarios** (was 5 in baseline; iter-1 internal review had 10; iter-1 external CRITICAL drove the expansion to 14).
- `openspec/specs/orbit-conventions/spec.md` — current baseline. The ADDED requirement `Review mode decision framework` is new with 5 scenarios.
- `openspec/specs/orbit-run-summary-emit/spec.md` — relevant because the iteration-aware logic depends on `external-system-*.md`, `address-reviews-*.json`, `review-system-*.json`, and `apply-*.json` filename conventions; the filename `<TS>` token format is defined in `orbit-conventions` `Internal-run JSON summary format`.
- `openspec/changes/harden-review-mode-recommendations/.orbit-runs/` — iteration history. Includes propose T0, review-proposal iter-1 internal (3 warnings + 2 suggestions resolved/deferred), external-proposal iter-1 from GPT-5 Codex (1 critical + 2 warnings + 1 suggestion), address-reviews iter-1 internal walk (3 resolved + 1 deferred + 1 stale-suppressed), address-reviews iter-2 external walk (4 resolved with 1 design revision).

## Cycle context

- **Iteration**: 2 (one prior external review for this change in proposal mode)
- **Prior internal findings still open**: 0
- **Prior external findings still open (iter-1 GPT-5 Codex)**: 0 — all 4 resolved in address-reviews iter-2.
- **Resolved since last external review (iter-1 → iter-2 state)**:
  - **iter-1 CRITICAL** (iteration-aware logic can mark unresolved external findings as converged): addressed via substantive design revision. Spec expanded from 3 convergence states to 5, with two-path convergence detection. New scenarios: `Path A clean content convergence`, `Path B address-reviews resolution convergence`, `external present with unresolved findings`, `external stale relative to artifact changes` (proper definition against `apply-*.json`), `convergence-state precedence rules`, `does NOT inspect apply-chunk emit chain`. Scenario count in orbit-review delta: 10 → 14.
  - **iter-1 WARN 1** (stale-branch compared wrong timestamps): folded into CRITICAL fix; stale is now defined against `apply-*.json` filename tokens (not internal-review timestamps).
  - **iter-1 WARN 2** (stale W/S phrasing missing + tasks.md row count off): all 10 sub-cases now have explicit stock phrasings (5 states × 2 severity sub-cases); tasks.md 1.1 updated to "10 new rows" + expanded recommendation-logic prose.
  - **iter-1 SUGGESTION** (scenario count drift): proposal.md + design.md updated to "5 scenarios (3 decision-criteria + 2 governance)" for the orbit-conventions delta.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as proposal)

Mandatory artifacts (under `openspec/changes/harden-review-mode-recommendations/`):

- `proposal.md` — Why + What Changes + Capabilities + Impact
- `design.md` — 6 decisions (D-scope-1, D-logic-1, D-ux-1, D-docs-1, D-mention-fresh, D-empirical-citation). **D-logic-1 was substantially rewritten in iter-2** to document the 5-state state machine; read it carefully against the spec.
- `tasks.md` — 14 tasks (one row-count + recommendation-logic-prose entry got expanded in iter-2)
- `specs/orbit-review/spec.md` — MODIFIED `Final assessment phrasings depend on mode` with **14 scenarios** post-iter-2-revision (3 proposal-mode unchanged + 6 system-mode convergence variants + 1 precedence + 1 non-gate + 1 edge-case-assumptions + 1 apply-chunk-emit-chain note + 1 audit-friendly)
- `specs/orbit-conventions/spec.md` — ADDED `Review mode decision framework` with 5 scenarios
- `explore.md` — historical record from explore phase (unchanged since iter-1)

Baseline + iter-1 history (for delta comparison):

- `openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-proposal-2026-05-26T13-00-53Z.md` — iter-1 external findings (your CRITICAL is here)
- `openspec/changes/harden-review-mode-recommendations/.orbit-runs/address-reviews-2026-05-26T13-41-20Z.json` — iter-2 address-reviews walk; documents the design revision

What the change does (one-paragraph summary):

Updates `/opsx:review --as system`'s final-assessment recommendations to reflect a 5-state convergence model based on external-review state (no prior external / Path A clean content / Path B resolved via address-reviews / present with unresolved findings / stale relative to apply). orbit-review's recommendation logic inspects 4 file types (external-system-*.md content + address-reviews-*.json source_path + apply-*.json filename tokens + review-system-*.json filename tokens). Cites bootstrap-orbit-status-cli's 3-of-3 empirical evidence as rationale for the system-mode default-flip toward external review. Adds a new `Review mode decision framework` requirement to orbit-conventions + a "Choosing a review mode" section to README. Proposal-mode default-recommendation behavior is unchanged. Closes [#20](https://github.com/las-sal/openspec-orbit/issues/20) + [#13](https://github.com/las-sal/openspec-orbit/issues/13).

## What to look for (9 proposal-mode passes)

1. **Structure & Delta Integrity** — required artifacts present; ADDED/MODIFIED valid; `openspec validate --strict` would pass.
2. **Internal Coherence** — proposal/design/specs/tasks aligned. **Verify D-logic-1 rewrite covers the 5-state model accurately + matches the spec scenarios**.
3. **Cross-Doc Coherence** — `CLAUDE.md`, README, and existing orbit-review SKILL/command remain consistent after this change applies.
4. **Archive Consistency** — ADDED requirement name doesn't collide; MODIFIED block includes the FULL updated content.
5. **Codegen Readiness** — the new 5-state convergence logic is implementable. **Specifically verify**:
   - Path A markdown parsing is precise (counting `### <title>` entries under each `## SEVERITY` heading)
   - Path B address-reviews JSON inspection is precise (`source_path` matching + `resolution_summary.deferred == 0 AND resolution_summary.escalated == 0`)
   - Stale detection compares against `apply-*.json` filename token (not internal-review file)
   - Convergence-state precedence rules resolve ambiguous cases unambiguously
6. **Gap Hunt** — what failure modes does the 5-state model still miss?
   - Does the precedence-rules scenario handle ALL combinations correctly?
   - What about address-reviews JSON whose source_path is a DIFFERENT external (i.e., references an external other than the most-recent one)?
   - What about clean external (Path A) followed by an apply, then NO new external — is this correctly "stale" per state 5, or could Path A still apply?
7. **Drift Hunt** — recommendation prose in spec scenarios should propagate cleanly to SKILL + command + README per tasks.
8. **Inline Review Marker Residue** — any `@review:` markers still present? (Should be 0.)
9. **Pre-Handoff Sweep** — anything else before shipping? Awkward wording, contradictions, examples that don't match prose.

## Areas worth extra scrutiny (iter-2 specifically)

This iter-2 specifically validates the iter-1 design revision. Spend extra rigor on:

1. **The 5-state convergence model's correctness** — walk through several iteration scenarios (clean external, findings external + address-reviews, post-apply-state, multiple-apply-chunks state) and verify each falls into exactly one state without ambiguity.

2. **Path A markdown parsing precision** — the spec describes detecting clean external via "no `### <title>` entries under each `## SEVERITY` heading; severity sections contain only `None.`". Is this unambiguously parseable? What if the external AI writes "(none)" instead of "None." (case variations)? What if it omits the severity section entirely?

3. **Path B JSON inspection precision** — the spec requires `address-reviews-*.json` whose `source_path` references the external file. What's the exact match — full path? Basename? What if multiple address-reviews JSONs reference the same external (iterative address-reviews walks)?

4. **The precedence-rules scenario** — is it complete? Are there state combinations not covered?

5. **The iter-1 CRITICAL is actually fixed** — re-read the CRITICAL's "Description" in `external-proposal-2026-05-26T13-00-53Z.md` and verify the fix specifically addresses it (not just the symptom).

## Output format — write to:

`openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-proposal-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format — pick a fresh one so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: harden-review-mode-recommendations (iteration 2)

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

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 2): harden-review-mode-recommendations

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
