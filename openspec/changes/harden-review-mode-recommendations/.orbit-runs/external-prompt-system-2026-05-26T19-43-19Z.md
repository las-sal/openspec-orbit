# External Review: harden-review-mode-recommendations (iteration 1, system mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

This is a **system-mode** review (post-apply): the change has been implemented and you are verifying the whole product state — code-vs-spec coherence, baseline compliance, surface walk, drift residue. Distinct from proposal-mode (pre-apply, artifacts only).

## Repo

https://github.com/las-sal/openspec-orbit

## Project context (read first)

- `CLAUDE.md` — handoff orientation for openspec-orbit
- `openspec/project.md` — project goals + stack (if present)
- `*_convention.md` at repo root (if any — orbit currently has none)
- `openspec/lenses/perspectives.md` — named callers worth validating from (note: this file currently does NOT exist for orbit; Pass 4 is N/A)
- `openspec/lenses/critical-paths.md` — user flows worth walking end-to-end (note: this file currently does NOT exist for orbit; Pass 5 is N/A)
- `openspec/changes/harden-review-mode-recommendations/.orbit-runs/` — full iteration history; see what's already been addressed in prior cycles

## Cycle context

- **Iteration**: 1 (this is the FIRST external system-mode review for this change).
- **Prior internal review findings still open**: 0 — the internal system-mode review (`review-system-2026-05-26T14-44-15Z.json`) returned all-clear across 5 passes (Passes 4 + 5 skipped per lens absence).
- **Prior external review findings still open**: 0 — no prior system-mode external. Proposal-mode had 2 prior externals (both by GPT-5 Codex on 2026-05-26); all 5 findings (1 CRITICAL + 4 WARNINGs across both iterations) were resolved via `/opsx:address-reviews` walks. See `address-reviews-2026-05-26T12-45-38Z.json`, `address-reviews-2026-05-26T13-41-20Z.json`, `address-reviews-2026-05-26T13-57-50Z.json` for the resolution audit trail.
- **Resolved since last review (proposal-mode iter-2 external)**: per `address-reviews-2026-05-26T13-57-50Z.json` — (1) state-machine coverage gap closed via per-external scoping; (2) Path A clean-sentinel parser aligned with `openspec-address-reviews/references/external-findings-format.md`; (3) Path B `source_path` matching specified as repo-relative normalization; (4) task 3.2 user-validation checklist expanded from 3-state to 5-state coverage.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What this change does (one-paragraph)

Bundle closing GitHub #20 + #13. Two surfaces change:

- **`/opsx:review --as system`'s final-assessment recommendation logic** — the spec MODIFIED requirement `Final assessment phrasings depend on mode` expands from 3 system-mode stock phrasings (CRITICAL / Only W/S / all clear) to 11 (1 CRITICAL + 10 across 5 convergence states × 2 sub-cases). The 5 states are: (1) no prior external, (2) Path A convergence (external content clean), (3) Path B convergence (external findings resolved via address-reviews), (4) external present with unresolved findings, (5) external stale relative to artifact changes. Default-flip: when no external system review exists for a change (State 1), the recommendation nudges toward `/opsx:review-external` (or `/opsx:review --fresh` as lighter alternative) before archive, citing the bootstrap-orbit-status-cli 3-of-3 empirical evidence as justification. Proposal-mode phrasings unchanged.
- **orbit-conventions baseline** — new ADDED requirement `Review mode decision framework` codifies decision criteria for choosing among orbit's three review modes (in-context / `--fresh` / external). 5 scenarios: 3 decision-criteria + 2 governance (drift-audit catches README↔spec divergence; framework is guidance not enforcement). New README `## Choosing a review mode` section provides discoverable user-facing guidance with the same framework.

The change has been fully applied: SKILL.md + opsx/review.md updated with the 14-row stock-phrasings table + iteration-aware logic prose + 3-of-3 empirical citation; README has the new section between Command reference + The external review cycle; all 14 tasks marked complete; `openspec validate --strict` passes.

## What to read for THIS review (`--as system`)

```
- openspec/changes/harden-review-mode-recommendations/proposal.md
- openspec/changes/harden-review-mode-recommendations/design.md
- openspec/changes/harden-review-mode-recommendations/tasks.md
- openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md
- openspec/changes/harden-review-mode-recommendations/specs/orbit-conventions/spec.md
- The change's commit range: git diff against main from when the change directory was created
- openspec/specs/orbit-review/spec.md (archived baseline — for baseline-compliance check; note current baseline still has the OLD 3-state framing for system mode, MODIFIED applies on archive via sync-specs)
- openspec/specs/orbit-conventions/spec.md (archived baseline — for cross-reference with new Review mode decision framework ADDED)
- openspec/specs/orbit-run-summary-emit/spec.md (the spec the iteration-aware logic references — filename `<TS>` token convention)
- .claude/skills/openspec-review/SKILL.md (the modified SKILL — Step 9 fully rewritten)
- .claude/commands/opsx/review.md (the mirrored command file — new Final assessment section)
- README.md (the new Choosing a review mode section at L483)
- .claude/skills/openspec-address-reviews/references/external-findings-format.md (referenced by the new Path A parser contract)
```

## What to look for (system-mode passes)

0. **verify-change-style structural check** — tasks done (14/14), spec coverage, design adhered. The external AI runs its own equivalent of upstream verify-change.
1. **Baseline Compliance** — does this change break any archived `openspec/specs/` requirement? Note: orbit-review baseline still describes the OLD 3-state system-mode framing pre-sync; the MODIFIED in the change delta resolves this on archive. Is the in-flight divergence acceptable, or is there a baseline contract being violated?
2. **Cohesion** — the change touched `.claude/skills/openspec-review/SKILL.md`, `.claude/commands/opsx/review.md`, `README.md`. Are there callers/dependents outside the touched files that should be updated? (e.g., other SKILL files that reference the old phrasings; other commands that recommend or echo the final-assessment line; orbit-status code that parses run-summary JSONs and might need updating for the new convergence_state field).
3. **Surface Walk** — `/opsx:review --as system` CLI surface: same flags, same name, same behavior contract? The output format changed (longer phrasings) — is this a breaking change for any consumer (e.g., orbit-status's tier-1 best-effort parse of `next_recommended`)? Are the new 10 phrasings parseable by orbit-status's recommendation parser?
4. **Perspective Reviews** — N/A (no lenses defined).
5. **Critical-Path Scan** — N/A (no lenses defined).
6. **Drift / Residue** — vocabulary residue, stale references, archive-coherence misses. Specifically: does any file still reference the OLD "All checks passed. Ready to archive." system-mode all-clear phrasing? Are the proposal-mode "Ready to apply." phrasings preserved (they should be)? Is the 3-of-3 bootstrap-orbit-status-cli citation used consistently across SKILL + command + README? Are there any in-flight `@review:` markers left in the change directory?

**Specific things worth scrutiny on this change** (areas I want second-opinion validation):
- **Iteration-aware logic correctness** — the 5-state convergence model with two-path detection. Are the WHEN clauses + precedence rules + edge-case assumptions actually exhaustive? Could a real change-state fall through all 5 states? (This is a recurring concern: external proposal-mode iter-1 caught the original 3-state version had a CRITICAL coverage gap; iter-2 caught a WARN-level coverage gap that we fixed via per-external scoping.)
- **Path A parser contract reuse** — the new spec text references `openspec-address-reviews/references/external-findings-format.md` for the accepted empty-severity sentinels (`None.`, `None`, `none.`, `(none)`). Does the actual external-findings-format file define these consistently? Are there other places in orbit where a clean-sentinel definition exists that should also be unified?
- **Path B `source_path` normalization** — the spec specifies "repo-relative path normalization" for Path B comparison. Is this rule implementable without ambiguity (e.g., what about symlinks, what about case-sensitivity on case-insensitive filesystems)?
- **Cross-doc coherence between README + spec + SKILL** — the size-tier table in README ("Recommended cycle patterns by change size") is guidance-not-enforcement per spec scenario `Framework guidance does NOT enforce cycle patterns`. But does the README's guidance match the spec scenarios' normative criteria? Are there subtle divergences that would surface on a future `/opsx:audit-drift` Category 3 run?
- **Empirical citation health** — the recommendation prose hard-codes `bootstrap-orbit-status-cli` (a sibling repo). The design.md flags this as a risk (citation rot). Is the citation specific enough to verify (date + finding-summary), or so specific it's fragile to upstream repo changes?

## Output format — write to:

`openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format with colons replaced — `YYYY-MM-DDTHH-MM-SSZ`. Pick a fresh timestamp so this file doesn't overwrite any prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: harden-review-mode-recommendations (iteration 1, system mode)

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
   git add openspec/changes/harden-review-mode-recommendations/.orbit-runs/external-system-<TS>.md
   git commit -m "External review (system, iter 1): harden-review-mode-recommendations

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
