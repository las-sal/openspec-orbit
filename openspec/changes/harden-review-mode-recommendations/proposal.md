## Why

Two related issues from cluster-2 + `bootstrap-orbit-status-cli` dogfooding land together:

- **[#20](https://github.com/las-sal/openspec-orbit/issues/20)** — Empirical evidence (3 of 3 real implementation bugs missed by in-context system review, all caught by external GPT-5 Codex review on `bootstrap-orbit-status-cli`) argues that `/opsx:review --as system`'s current default ("Ready to archive" on clean in-context pass) silently encourages skipping the cross-AI second-opinion pass. The default should nudge toward external review before archive.

- **[#13](https://github.com/las-sal/openspec-orbit/issues/13)** — The decision framework for when to use in-context vs `--fresh` vs external review is fragmented across 3 files. User reported spending 4-5 conversation turns reconstructing the rough pattern. Without a consolidated framework, the recommendation from #20 reads as arbitrary policy rather than data-driven default.

The two bundle naturally: #20 is the behavior change; #13 is the framework that justifies it.

## What Changes

**System-mode review final-assessment recommendations updated (#20)**:
- `Final assessment phrasings depend on mode` requirement MODIFIED — system-mode "All clear" and "Only WARNING/SUGGESTION" cases recommend `/opsx:review-external` (best independence) or `/opsx:review --fresh` (lighter middle-ground) before `/opsx:archive` when no external system review has run for this change yet.
- Iteration-aware logic: when external system review has already run and converged clean, recommendation simplifies to "External system review converged. Ready to archive."
- Recommendation prose cites the `bootstrap-orbit-status-cli` 3-of-3 empirical evidence as rationale.
- Proposal-mode review's default-recommendation behavior is **unchanged** — the empirical evidence is specifically about system mode where anchoring risk is highest (AI reviewing code it just wrote).

**Decision-framework documentation added (#13)**:
- `orbit-conventions` baseline gains a new `Review mode decision framework` requirement (3 scenarios) codifying when each mode is appropriate.
- `README.md` gains a new "Choosing a review mode" section between the workflow + external-review sections, discoverable from the TOC.
- Both surfaces present the same framework; the spec gives normative force, the README gives discoverability.

**No UX additions**:
- The recommendation lands as updated text in the existing final-assessment stock-phrasings table. NO interactive prompt is added; review stays as a passive-emit command. The user reads the recommendation prose and invokes the next command themselves.

## Capabilities

### New Capabilities

(none — both spec changes are MODIFIED requirements + ADD scenarios under existing capabilities)

### Modified Capabilities

- `orbit-review`: MODIFIED `Final assessment phrasings depend on mode` requirement — system-mode stock phrasings updated for the recommend-external default + iteration-aware logic.
- `orbit-conventions`: ADD new `Review mode decision framework` requirement codifying when in-context vs `--fresh` vs external is appropriate (3 scenarios — first-pass default / anchoring-break case / cross-AI cross-check case).

## Impact

- **Spec deltas**: 2 capability deltas (1 MODIFIED requirement + 1 ADDED requirement). Sync applies on archive; baseline gains the new conventions requirement and updated review phrasings.
- **Skill body**: `.claude/skills/openspec-review/SKILL.md` final-assessment stock phrasings table updated to match the modified requirement. Recommendation-logic prose explains the iteration-aware behavior.
- **Command body**: `.claude/commands/opsx/review.md` updated to match SKILL body (per the orbit-modified pattern; the recommendation logic surfaces in both).
- **Documentation**: `README.md` gains a "Choosing a review mode" section. Estimated +40-60 lines.
- **No runtime changes** outside the recommendation text + spec text. Existing review machinery (passes, scorecard, run-summary emit) unchanged.
- **No CLI changes**: `/opsx:review --as system` invocation pattern unchanged; only the final-assessment line text changes.
- **Issue closure**: closes [#20](https://github.com/las-sal/openspec-orbit/issues/20) + [#13](https://github.com/las-sal/openspec-orbit/issues/13).
- **Related (unchanged)**:
  - [#12](https://github.com/las-sal/openspec-orbit/issues/12) (dynamic next-reviewer recommendation) — this change implements one specific default within #12's general framework; the broader recommendation-as-runtime-logic stays for a future change.
  - [#15](https://github.com/las-sal/openspec-orbit/issues/15) (workflow inflection points) — out of scope here; `/opsx:archive` pre-flight check for missing external review explicitly deferred.
- **Iteration trail**: this change inherits the cluster-2 review-cycle discipline (proposal-mode review iter-1 + iter-2 fresh + external + apply chunks + system-mode review). Recommend running the new external default-recommendation on this change itself as a dogfood-validation.
