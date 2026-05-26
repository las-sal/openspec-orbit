## Context

Two issues land together because they're behavior + documentation of the same workflow concern: **which review mode is appropriate when**.

**The empirical evidence** (from #20 issue body): on `bootstrap-orbit-status-cli`, internal in-context `/opsx:review --as system` (iter-1) found 2 non-bug warnings; external GPT-5 Codex review on the same code found **3 real implementation bugs** (timestamp sort bug, mtime-vs-filename-timestamp freshness bug, missing required render segment). The archive JSON's `meta_validation` recorded: *"3 of 3 external system-review findings were missed by the in-context system review."* This is the strongest single piece of dogfood evidence orbit has accumulated about review-mode trade-offs.

**The user-facing pain** (from #13 issue body): the user (during orbit-status dogfooding) asked *"the AI suggested a pattern for when to use in-context vs external vs spin-clean-internal — did that make it into the repo?"* Answer: the mechanisms shipped; the framework didn't. 4-5 conversation turns to reconstruct it from scattered sources (README L184, explore.md rejected-decisions, review SKILL.md L58).

This change lands the framework + flips the system-mode default to reflect what the empirical evidence already justifies.

## Goals / Non-Goals

**Goals:**

- Update `/opsx:review --as system`'s final-assessment recommendations to nudge toward external review (or `--fresh`) before archive, when no external system review has run yet for the change.
- Add iteration-aware logic so the recommendation stops nudging once external has converged clean.
- Cite the `bootstrap-orbit-status-cli` 3-of-3 evidence in the recommendation prose so the policy reads as data-driven, not arbitrary.
- Document the in-context vs `--fresh` vs external decision framework in BOTH `orbit-conventions` baseline (normative) and README (discoverable).
- Close [#20](https://github.com/las-sal/openspec-orbit/issues/20) + [#13](https://github.com/las-sal/openspec-orbit/issues/13).

**Non-Goals:**

- **Change proposal-mode review's default-recommendation behavior** — the empirical evidence is specifically about system-mode. Cluster-2 also showed external catching things in proposal mode, but that's a separate empirical case worthy of its own decision rather than scope expansion here.
- **Interactive [E]/[F]/[Q] prompt at end of system-mode review** — the #20 issue body sketches one; rejected here per D-ux-1 (minimal scope, doesn't break passive-emit pattern).
- **`/opsx:archive` pre-flight check for missing external review** — bleeds into [#15](https://github.com/las-sal/openspec-orbit/issues/15) (workflow inflection points). The recommendation in system-mode review's final-assessment is enough nudge; archive doesn't also need to gate.
- **Dynamic next-reviewer recommendation logic generally** — tracked as [#12](https://github.com/las-sal/openspec-orbit/issues/12). This change is the specific default for one inflection point (system-mode review final-assessment); the broader runtime-logic framework stays for a future change.
- **Cycle-pattern enforcement** — the framework documentation includes recommended cycle patterns by change size (small / medium / substantial), but as guidance, NOT as normative spec scenarios. orbit shouldn't refuse to archive a "substantial" change without external review; the recommendation is sufficient.

## Decisions

### D-scope-1: Default-flip applies to system-mode review only

**Decision**: The recommendation text change in `Final assessment phrasings depend on mode` applies to system-mode "All clear" and "Only WARNING/SUGGESTION" cases. Proposal-mode phrasings are unchanged.

**Why**:
- Empirical evidence (3-of-3 missed) is system-mode specific.
- Proposal-mode review's anchoring story differs: the AI reviewing artifacts often didn't author them (or authored them in a session ago); self-confirmation risk is lower.
- Bundling proposal-mode changes would expand scope + weaken the empirical justification.

**Alternative considered**:
- Apply default-flip to both modes. Rejected — separate empirical case needed for proposal-mode; the data we have is system-specific.

### D-logic-1: Iteration-aware recommendation logic

**Decision**: The recommendation conditions on whether external system review has run for this change yet. If no `external-system-*.md` file exists in `.orbit-runs/`, recommend external next. If external has run and the most recent `external-system-*.md` is older than the current clean `review-system-*.json` (i.e., external ran, then internal re-ran clean), recommend archive directly.

**Concrete logic**:
```
1. Look for openspec/changes/<name>/.orbit-runs/external-system-*.md
2. If absent → recommend /opsx:review-external (or /opsx:review --fresh)
3. If present:
   - Find the most recent internal review-system-*.json
   - If it's clean (0 critical) and timestamped later than the external file → recommend archive
   - Otherwise → external is stale, recommend re-running external
```

**Why over unconditional always-recommend-external**:
- Composes with [#12](https://github.com/las-sal/openspec-orbit/issues/12) (general dynamic recommendation framework). This change is the specific default within #12's general logic.
- Avoids nagging on iter-N+1 where external already converged.
- Acknowledges that some changes (small, low-stakes) may legitimately archive without external — the framework documentation (per D-docs-1) makes this explicit.

### D-ux-1: Text-only final-assessment update (no interactive prompt)

**Decision**: Recommendation lands as updated text in the existing stock-phrasings table. NO interactive prompt is added; the review skill continues to passively emit a final-assessment line.

**Concrete shape** (sketch — exact wording in spec scenarios):

Current system-mode "All clear":
```
All checks passed. Ready to archive.
```

New system-mode "All clear" (no prior external):
```
All checks passed. Recommend /opsx:review-external for fresh-context cross-check
before archive (per `Review mode decision framework`; in-context system review
missed 3 of 3 real bugs in bootstrap-orbit-status-cli). Or /opsx:review --fresh
for a lighter pass. Proceed to /opsx:archive if accepting that risk.
```

New system-mode "All clear" (external already converged):
```
All checks passed. External system review converged clean. Ready to archive.
```

**Why over interactive prompt**:
- Minimal scope: 2-3 cells of the stock-phrasings table change, plus matching SKILL prose.
- Doesn't break orbit's passive-emit review pattern.
- The text recommendation IS the [E]/[F]/[Q] choice presented as prose; user invokes the next command themselves.

**Trade-off accepted**: slightly longer final-assessment lines. Acceptable given the cognitive benefit of inline rationale.

### D-docs-1: Documentation in both `orbit-conventions` baseline and README

**Decision**: Add a new `Review mode decision framework` requirement to `orbit-conventions` (3 scenarios codifying the framework as testable statements) + add a new "Choosing a review mode" section to README.

**Spec scenarios** (tight; just decision criteria, NOT exhaustive cycle patterns):

1. **In-context default for first-pass review with low anchoring** — when no prior review of the artifact has run + no design ambiguity flagged, in-context is appropriate.
2. **`--fresh` for anchoring-break in multi-iteration cases** — when in-context review has run N times in the same session against the same scope and the user wants to verify convergence without cross-AI cost, `--fresh` is the middle-ground option.
3. **External for system-mode + substantive cross-AI verification** — when system-mode review is verifying code the AI just wrote (high anchoring risk per bootstrap-orbit-status-cli evidence) OR the change is substantial enough that cross-AI cross-check pays for itself.

**README section content** (lighter guidance, optional cycle patterns):
- Three-mode framework summary (in-context / `--fresh` / external)
- Recommended cycle patterns by change size: small / medium / substantial / high-stakes
- Cite the bootstrap-orbit-status-cli evidence as the empirical basis for the system-mode default
- Cross-reference the orbit-conventions baseline requirement for normative criteria

**Why both** (per #13 explicit recommendation):
- README section for discoverability (TOC-indexed; new users + handoff readers find it).
- Spec requirement for normative force + `audit-drift` Cat 3 coverage (drift between README and spec gets caught).
- Downstream tools (orbit-status per [#10](https://github.com/las-sal/openspec-orbit/issues/10)) can query the spec scenarios for runtime recommendation prompts.

**Alternative considered**:
- README only — rejected; loses normative force; tools can't query.
- Spec only — rejected; loses discoverability; new users hit the framework via spec which is high-friction.

### D-mention-fresh: Recommendation prose mentions both external AND `--fresh`

**Decision**: The updated final-assessment text mentions both `/opsx:review-external` (primary recommendation; best independence) and `/opsx:review --fresh` (secondary; lighter middle-ground) as next-step options.

**Why mention both**:
- `--fresh` exists exactly for the anchoring-break use case. Recommending only external ignores it.
- The middle-ground is appropriate when cross-AI round-trip cost feels disproportionate.

**Trade-off**: slightly longer recommendation prose; cognitive load on the user. Mitigated by the `Review mode decision framework` requirement explaining when each is appropriate.

### D-empirical-citation: Recommendation prose cites the bootstrap-orbit-status-cli evidence

**Decision**: Both the spec-text and the SKILL-prose recommendation include a brief reference to the bootstrap-orbit-status-cli 3-of-3 evidence (e.g., "per `Review mode decision framework`; in-context system review missed 3 of 3 real bugs in bootstrap-orbit-status-cli").

**Why**:
- The evidence IS the policy's justification. Without citation, the default-flip reads as arbitrary.
- Users can follow the citation back to the archive JSON for verification.
- Aligns with orbit's read-before-reference discipline applied to its own docs.

**Trade-off**: external-name reference (`bootstrap-orbit-status-cli` lives in a sibling repo). Mitigated by the citation being illustrative-not-load-bearing; the recommendation logic doesn't require the user to read the cited evidence.

## Risks / Trade-offs

- **Over-nudging users with substantial recommendations** — every system-mode review now produces a longer final-assessment line. Risk: alert fatigue; users start ignoring the recommendation. Mitigation: iteration-aware logic per D-logic-1 stops nudging once external converges; the recommendation only appears on the first system-mode pass.

- **Citation rot** — the bootstrap-orbit-status-cli evidence lives in a sibling repo. If that repo's archive structure changes or the linked JSON moves, the citation breaks. Mitigation: cite the archive-date + finding-summary rather than the literal file path; archive structures are stable but if the citation does rot, drift-audit Cat 3 (cross-doc consistency) would catch it.

- **Framework documentation may get stale** — the README "Choosing a review mode" section duplicates content from the orbit-conventions baseline scenarios. Mitigation: drift-audit Cat 3 already catches README-vs-spec drift; the new `Review mode decision framework` requirement is a clean target for that check.

- **`--fresh` is mentioned but not deeply explored** — the framework documents `--fresh` as an option, but orbit hasn't accumulated empirical evidence about how `--fresh` performs vs external on substantive changes. Risk: users follow the documented framework but discover `--fresh` doesn't quite break anchoring enough on certain change types. Mitigation: acceptable as a v1 framework; future empirical data can refine.

- **Default-flip + proposal-mode unchanged is asymmetric** — system-mode recommends external, proposal-mode doesn't. Risk: users wonder why. Mitigation: the framework documentation explains the asymmetry (system mode = AI reviewing code it just wrote; higher anchoring risk; proposal mode = artifacts review with different anchoring story).
