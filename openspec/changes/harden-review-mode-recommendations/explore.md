# Explore: harden-review-mode-recommendations

## Premise

Bundle two related issues that emerged from cluster-2 + bootstrap-orbit-status-cli dogfooding:

- **[#20](https://github.com/las-sal/openspec-orbit/issues/20)** — System-mode review default-recommend external/fresh-context, not in-context. Empirical evidence from `bootstrap-orbit-status-cli`: 3 of 3 real implementation bugs missed by in-context system review, all caught by external GPT-5 Codex review. The current default ("Ready to archive" after in-context system-mode review with no findings) silently encourages skipping the cross-AI second-opinion pass.

- **[#13](https://github.com/las-sal/openspec-orbit/issues/13)** — Document the review-mode decision framework (in-context vs `--fresh` vs external). The mechanisms exist but the decision framework is scattered across 3 different files. User-reported pain: spent 4-5 conversation turns reconstructing the rough pattern.

The two bundle naturally: #20 is the behavior change that #13's documentation would justify; #13 is the framework that #20's recommendation logic instantiates.

This explore designs the bundle. Implementation will be a normal openspec change (`/opsx:propose` → review → apply → archive).

## Decisions

### D1: Scope #20's behavior change to system-mode review ONLY (not proposal-mode)

**Decision**: The default-flip applies to `/opsx:review --as system` only. Proposal-mode review's default-recommendation behavior is unchanged by this bundle.

**Why** (per #20 issue body):
- Empirical evidence is specifically about system-mode (where the AI reviewing has just written the code; anchoring risk is highest).
- Proposal-mode review's anchoring story is different: the AI is reviewing artifacts the user might have authored or that came from a different earlier session; less self-confirmation risk.
- Bundling proposal-mode changes here would expand scope and weaken the empirical justification.

### D2: Cite the `bootstrap-orbit-status-cli` 3-of-3 evidence in the recommendation prose

**Decision**: Both the SKILL body's prompt UX and the documentation include the bootstrap-orbit-status-cli evidence as the rationale (e.g., "In this project's own evidence, 3 of 3 real bugs were missed by in-context review and caught by external review.").

**Why**: The evidence IS the argument. Without citing it, the recommendation reads as an arbitrary policy rather than data-driven default.

### D3: Bundle #20 (behavior) + #13 (docs) into one change

**Decision**: One change closes both issues. Spec change + doc change land together.

**Why**:
- The behavior change without docs is mysterious ("why does it recommend external?"); the docs without behavior change is aspirational ("here's the framework, but the tool doesn't enforce it").
- Both touch the same baseline area (`orbit-review` final-assessment phrasings and/or a new "Review mode decision" requirement).
- Net change shape is small enough to fit one change.

### D4: Iteration-aware recommendation logic (not unconditional)

**Decision**: System-mode review's final-assessment recommendation conditions on prior-reviewer history. If no external system review has run for this change yet, recommend `/opsx:review-external` next. If external has run and converged clean (iter-N+1 with no new findings against the same scope), recommend `/opsx:archive` directly.

**Why over unconditional always-recommend-external**:
- Composes with [#12](https://github.com/las-sal/openspec-orbit/issues/12) (dynamic next-reviewer recommendation) which the #20 issue body explicitly calls out as related-but-separate. Iteration-aware logic here is one specific default within #12's general framework.
- Avoids nagging on iter-2+ where the user has already done the external pass.
- The decision logic is small: read `.orbit-runs/` for prior `external-system-*.md` files; if any exist with no findings later than the most-recent `review-system-*.json` clean run, recommend archive; otherwise recommend external.

**Trade-off accepted**: a user who DOES want to re-run external on iter-N can do so manually; the recommendation just stops nudging once convergence is reached.

### D5: Text-only final-assessment update (no interactive prompt)

**Decision**: The recommendation lands as updated text in the `Final assessment phrasings depend on mode` requirement's stock phrasings table. No interactive [E]/[F]/[Q] prompt is added; the review skill stays as a passive-emit command.

**Why over interactive prompt**:
- Minimal scope: 1-2 cells of the stock-phrasings table change, plus matching SKILL prose.
- Doesn't break orbit's existing pattern of review emitting a final-assessment LINE that the user reads + decides next-action from.
- An interactive prompt would require a new dispatcher step + spec-side scenario coverage + UX flow specification — substantial scope expansion for marginal benefit (the text recommendation is itself the [E]/[F]/[Q] choice presented as prose).

**Concrete shape** (sketch — refined in propose):
```
Old:  All checks passed. Ready to archive.
New:  All checks passed. Recommend /opsx:review-external for a fresh-context
      cross-check before archive (per cluster-2 + bootstrap-orbit-status-cli
      empirical evidence: in-context system review missed 3 of 3 real bugs in
      one canonical case). Or /opsx:review --fresh for a lighter pass (same
      model, fresh context). Proceed to /opsx:archive if accepting that risk.
```

Iteration-aware adjustment (per D4): when prior external has already converged clean, the recommendation simplifies to `All checks passed. External system review converged. Ready to archive.`

### D6: Decision framework documentation lands in BOTH README and orbit-conventions baseline

**Decision**: Add a "Choosing a review mode" section to README + ADD a new requirement to `orbit-conventions` baseline codifying the decision criteria.

**Why both** (per #13 issue body's recommendation):
- README section: discoverability for new users + handoff readers; gets indexed in TOC.
- orbit-conventions spec requirement: normative force; `audit-drift` Cat 3 would catch drift if README and spec contradict; downstream tools (orbit-status per [#10](https://github.com/las-sal/openspec-orbit/issues/10)) can query the spec for "you've done N in-context iterations; consider --fresh next" prompts.

**Scope discipline**: keep the spec requirement tight — just the decision criteria (when each mode is appropriate), NOT exhaustive cycle patterns per change size (those go in README as guidance, not as normative scenarios). Three scenarios sketch:
1. In-context default for first-pass review with low anchoring
2. `--fresh` for multi-iteration cases where in-context anchoring is suspected
3. External for substantive changes / code-vs-spec verification / before-archive system-mode gate

### D7: Recommendation prose mentions both external AND `--fresh` as options

**Decision**: The updated final-assessment prose mentions both `/opsx:review-external` (best independence; cross-AI) and `/opsx:review --fresh` (lighter; fresh subagent, same model) as recommended next steps. External is the stronger recommendation; `--fresh` is the middle-ground option.

**Why mention both**:
- `--fresh` is a real orbit option that exists exactly for the anchoring-break use case.
- Recommending only external ignores it; the framework is richer than just in-context vs external.
- The lighter middle-ground is appropriate for cases where cross-AI round-trip cost feels disproportionate.

**Trade-off accepted**: slightly longer recommendation prose; cognitive load on the user to choose between external and --fresh. Mitigated by the framework documentation (D6) explaining when each is appropriate.

## Open questions

(All explore-phase open questions resolved during the initial batch walk — see D4-D7 + Considered & out for the resolutions.)

## Considered & out

- **Q4 / `/opsx:archive` pre-flight check for missing external review** — Rejected from this bundle. Bleeds into [#15](https://github.com/las-sal/openspec-orbit/issues/15) (workflow inflection points) and conflates "review recommendation" with "archive gating." The recommendation in system-mode review's final-assessment is enough nudge; archive doesn't need to also gate. Defer to #15 or a separate small change.

- **Interactive [E]/[F]/[Q] prompt at end of system-mode review** — Rejected per D5. The text-only approach is minimal-scope and doesn't break the passive-emit pattern. The user reads the recommendation prose and invokes the next command themselves; no new dispatcher logic required.

- **Always-recommend-external unconditionally** — Rejected per D4. Iteration-aware logic respects convergence: once external has run clean, stop nudging.

- **Change proposal-mode review's default-recommendation behavior too** — Out of scope per D1. Empirical evidence in #20 is system-mode specific. Cluster-2 also showed external catching things in proposal mode, but that's a separate empirical case worth its own issue + decision rather than expanding this bundle's scope.

## References

### Closes (intended)

- [#20](https://github.com/las-sal/openspec-orbit/issues/20) — System-mode review default-recommend external/fresh-context
- [#13](https://github.com/las-sal/openspec-orbit/issues/13) — Document the review-mode decision framework

### Related (open)

- [#12](https://github.com/las-sal/openspec-orbit/issues/12) — dynamic next-reviewer recommendation based on iteration history. This bundle implements one specific default within #12's general framework; #12's broader recommendation-as-runtime-logic stays for a future change.
- [#15](https://github.com/las-sal/openspec-orbit/issues/15) — workflow inflection points list next-step options. Generalizes the recommendation-at-checkpoint idea; out of scope here.
- [#16](https://github.com/las-sal/openspec-orbit/issues/16) — review skip unchanged artifacts; adjacent inflection-point question.

### Archived precedents

- `openspec/changes/archive/2026-05-24-orbit-onboard-follow-up/` — final-assessment recommendation logic of that change's address-reviews iter-2 (user pushback that surfaced the verbatim-duplicates principle) is precedent for "user pushback during review caught what in-context AI missed."
- `openspec/changes/archive/2026-05-24-align-readme-install-with-v1-3-1/` — external GPT-5 Codex review caught CRIT 1 (README sync task violated baseline SHALL) + CRIT 2 (verification spec promised more than task implemented) in iter-1 system mode. Two more bugs in the "in-context misses, external catches" pattern.
- `las-sal/orbit-status` → `openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/` — the source of the 3-of-3 empirical evidence. External review found 3 real implementation bugs (timestamp sort, mtime-vs-filename freshness, missing render segment) all missed by in-context system review.

### Baseline specs (relevant)

- `openspec/specs/orbit-review/spec.md` `Final assessment phrasings depend on mode` (L130) — the requirement this change MODIFIES.
- `openspec/specs/orbit-conventions/spec.md` — possible new requirement codifying the review-mode decision framework (per Q3).
- `README.md` — possible new "Choosing a review mode" section.
