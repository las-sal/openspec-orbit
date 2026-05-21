# Internal Review iter-3 (in-context, --thorough; rendered as findings for `--from-file`): emit-run-summary-jsons-from-workflow-commands

**Reviewer**: Claude Opus 4.7 (in-context, iter-3 proposal-mode, --thorough depth, no --fresh flag)
**Date**: 2026-05-21

> **Note**: Markdown rendering of `.orbit-runs/review-proposal-2026-05-21T02-43-15Z.json` for `--from-file` parser compatibility (per openspec-orbit#4).
>
> Iter-3 with `--thorough` adversarial framing surfaced 3 net-new findings after 33 prior findings resolved across 3 prior modes (iter-1 in-context internal, iter-1 external Codex, iter-2 fresh-context). The --thorough lever (constraint probes on Pass 5, second-pass reset on Pass 9) was effective at pushing past anchoring even in-context.

## CRITICAL

None.

## WARNING

### T1: Task 9.5 'backward-compatibility check' directly contradicts task 1.4 + 1.5 (which intentionally MODIFY existing emitters' shapes)
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:88
**Description**: Task 9.5 reads: "Backward-compatibility check: run /opsx:review, /opsx:address-reviews, /opsx:archive (existing emitters); verify their existing JSON shapes are unchanged (no regressions from this change)." But task 1.4 (after F1's expansion) intentionally modifies those exact emitters' SKILL.md instructions: review and address-reviews gain `kind`; archive gains `kind` + `final_assessment` + `next_recommended`. Task 1.5 then verifies archive emits include all 3 new fields. So task 9.5's "verify shapes are unchanged" check would falsely FAIL after task 1.4 runs — because the shapes ARE intentionally changed. **Recommendation**: rewrite task 9.5 to verify the INTENDED changes are in place (e.g., "verify /opsx:review and /opsx:address-reviews emits now include `kind: 'editorial'`; archive emits now include `kind: 'lifecycle'` + `final_assessment` + `next_recommended` per task 1.4"). The original intent of 9.5 — guard against unintended regressions in OTHER fields — should be preserved as a separate phrasing: "no fields beyond the spine-additions in task 1.4 should be removed or repurposed."

### T2: Named-mode explore maturity rules have an unhandled state — 4+ decisions captured AND >1 open question falls through all 3 rules
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:138
**Description**: The `Named-mode explore recommendation by maturity` requirement defines three rules: Early (0-1 decisions), Mid (2-3 decisions), Mature (4+ decisions AND ≤1 open question). The case of `decisions_captured >= 4 AND open_questions_count > 1` matches NONE of these: not Early (decision count too high), not Mid (decision count too high), not Mature (open-question count too high). An implementer faced with this state would have no rule to apply. **Recommendation**: either (a) broaden Mid to cover this case ("2-3 decisions OR 4+ decisions with >1 open question"), (b) broaden Mature to just "4+ decisions" (dropping the open-Q condition; let open questions live in design.md or be addressed before propose), or (c) add a 4th rule explicitly for high-decisions-high-questions case. Lean: (a) — the maturity ladder is meant to push toward propose only when ready; high open questions means NOT ready regardless of decision count, so Mid behavior (continue or propose if ready) is the right semantics. Add a scenario for this case once decided.

## SUGGESTION

### T3: explore.md retains 3 "emit JSON" instances that should have been swept during W7's terminology standardization
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/explore.md:145
**Description**: W7's fix swept `emit JSON` → `run-summary JSON` across proposal/design/spec.md. But explore.md was treated as historical record and not swept. 3 instances remain at lines 145 (D9 description: "the emit JSON's next_recommended reads"), 226 (D14 description: "the emit JSON's next_recommended is multi-step prose"), 244 (D10 description: "the emit JSON's next_recommended is copied"). The historical-fidelity argument for preserving terminology is weak here because D9/D10/D14 are STILL the canonical decisions (not deprecated) — the terminology drift now propagates from explore.md into anyone re-reading the explore for context. **Recommendation**: sweep the 3 instances in explore.md too. Trivial (3-line edit). Marginal because explore.md is historical-context-only; readers should know the canonical terminology lives in the specs.

## Notes

Iter-3 in-context with --thorough caught 2 WARNINGs that all 3 prior modes (in-context iter-1, external iter-1 Codex, fresh-context iter-2) missed. Distinct value-add mechanism: --thorough's adversarial constraint-probe framing on Pass 5 surfaced T2 (the maturity-rule fall-through), and Pass 9's second-pass-with-reset surfaced T1 (the within-tasks.md contradiction my F1 expansion introduced). All 33 prior findings verified resolved; zero stale findings re-surfaced.

`openspec validate emit-run-summary-jsons-from-workflow-commands --strict` passes.
