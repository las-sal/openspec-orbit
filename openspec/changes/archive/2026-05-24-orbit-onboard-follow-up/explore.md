# Explore: orbit-onboard-follow-up

## Premise

Closes the cluster-2 wind-down by replacing the upstream-bodied `openspec-onboard` SKILL.md + matching `.claude/commands/opsx/onboard.md` with 100% orbit-authored content. Closes [openspec-orbit#23](https://github.com/las-sal/openspec-orbit/issues/23).

This is the follow-up to the just-archived `align-readme-install-with-v1-3-1` (2026-05-24), which itself was the follow-up to the just-archived `lean-overlay-and-add-orbit-onboard` (2026-05-24) after that change was cut to chunks 1+2.

**~80% of the design was already explored** in the cut lean-overlay change (see archived `explore.md` D8-D11): replace body in-place at `openspec-onboard`, 5-section reference-leaning hybrid walkthrough (Setup verification / Identity / Canonical-flow / Quick-reference / Try-it nudge), lenses introduced contextually, external-review demoed abstractly. Most of this transfers cleanly.

**~20% needs fresh thinking** — primarily the Setup verification section, which the original specced against incorrect install-surface assumptions (referenced files that aren't installed; would false-fail on any fresh install). Now that [[align-readme-install-with-v1-3-1]] established documented post-install state (15 skills + 16 commands, sandbox-verified), verification can be re-grounded against the real surface.

Memory grounding this explore:
- [[openspec-1-3-1-actual-install-surface]] — what `init --tools claude` actually produces
- [[readme-install-section-staleness]] — 3 baseline-drift items for this follow-up to fix in passing

## Decisions

### D1: Setup verification hard-stops on failure (not warn-and-continue)

If the post-install state shows missing orbit-authored skills (modes 1/2/5/7 from the failure-mode taxonomy below), verification halts the onboard session with an actionable remediation message. The canonical-flow walkthrough does NOT run in degraded mode.

**Why hard-stop over warn-and-continue**:
- Less effort to author: one decision tree, one prose branch in SKILL body. Walkthrough sections assume all commands are runnable (no defensive "if /opsx:X is missing, skip" branching).
- Better UX: a canonical-flow walkthrough teaching commands the user can't actually run materially misleads. Halting with "your setup is incomplete; here's what's missing" is more honest.
- Verification IS a gate, not a report — that framing simplifies all downstream SKILL body content.

**Soft failures** (e.g., `feedback/` present but everything else correct — prune step skipped) DO continue with a warning, not a hard-stop. The user can still run the walkthrough; the warning gets surfaced for post-walkthrough cleanup.

**Failure-mode taxonomy** (informs verification logic + remediation messages):

| Mode | What's wrong | Detection | Severity |
|---|---|---|---|
| 1 | Overlay step skipped entirely | All 4 orbit-authored skills missing | hard-stop |
| 2 | Overlay partially applied | Some but not all orbit-authored skills present | hard-stop |
| 3 | Prune step not run | `feedback/` present | warn-continue |
| 4 | Wrong upstream version | `openspec --version` ≠ 1.3.1 | hard-stop |
| 5 | Wrong upstream install profile | 10 expected upstream skills missing | hard-stop |
| 6 | Stale AI client cache | Undetectable from skill body | mention as gotcha |
| 7 | openspec-sync-specs missing | `.claude/skills/openspec-sync-specs/` absent | hard-stop |

### D2: Verification messaging is lumped, not distinguished per sub-mode

When overlay-incomplete failure modes (1/2/5/7 from D1's taxonomy) trigger, verification emits a single "overlay incomplete; see README install section #2" message rather than 4 sub-variant messages. The user re-runs the overlay step in all cases.

**Why lumped over distinguished**:
- Lower effort: one detection branch + one prose message vs four.
- Remediation is the same regardless of sub-mode (re-run the documented `cp -r` overlay sequence + prune step).
- Distinguished messages only add value when overlay was *almost* complete (e.g., sync-specs alone missing) — that's a narrow edge case; the README's install steps work either way.

**Trade-off accepted**: a user with sync-specs-only-missing gets the same "overlay incomplete" message as a user with fully-skipped overlay. Both re-run the overlay step; the partial-completion user re-runs slightly more than strictly necessary. Acceptable cost.

## Open questions

### D3: Layered positive checks (option B) — show the overlay structure on pass

Setup verification's pass output enumerates the layered structure rather than emitting a terse "all good":

```
✓ openspec CLI @ 1.3.1 (pin match)
✓ 10 upstream-modified skills present (`# Orbit additions` count = 10)
✓ 4 orbit-authored skills present (openspec-review, openspec-review-external,
   openspec-audit-drift, openspec-address-reviews)
✓ 1 upstream-required primitive present (openspec-sync-specs)
✓ feedback/ absent (prune step verified)
→ Proceeding to canonical-flow walkthrough.
```

**Why layered (B) over terse (A) or hybrid (C)**:
- Pedagogical fit: `/opsx:onboard` is specifically a teaching session. Showing the layered structure at verification makes orbit's `Overlay file disposition` category framework (10 modified + 4 authored + 1 primitive) visible before the walkthrough even starts. Reinforces categories the user will encounter in README install docs and baseline specs.
- Mirrors baseline structure: the 4 check categories map 1:1 to `orbit-conventions` `Overlay file disposition` requirement's 4 scenarios. Verification output IS the spec made executable.
- Effort delta vs A is small: ~7 lines of structured output vs ~2 lines. Both have the same detection logic underneath; only the prose differs.
- Inherited from prior design D9 ("quick check") but adapted for the teaching-session audience — quick was vague; layered is concrete.

**Soft-fail warning case** (feedback/ present): the layered output naturally surfaces the issue inline (`⚠ feedback/ present — run \`rm -rf .claude/skills/feedback\` to align with current overlay disposition`) rather than needing a separate warn-mode prose branch.

### D4: Verification stays bundled as section-1 of `/opsx:onboard` (no separate `/opsx:verify-setup` command)

One entry point: `/opsx:onboard`. Verification runs first; canonical-flow walkthrough follows on pass. Re-verifying standalone requires re-invoking `/opsx:onboard` and exiting after section 1.

**Why bundled (A) over separate command (B) or documented re-invocation (C)**:
- Lower surface area: no new `/opsx:verify-setup` command file + SKILL.md; no extra entry in `Overlay file disposition`; no extra README install enumeration.
- Re-verify use case is rare in practice — after troubleshooting an install issue, the standalone-verify path saves a few seconds; not worth the surface cost.
- "Documented re-invocation" (C) adds prose for marginal benefit; users can already re-invoke `/opsx:onboard` without instructions.

**Trade-off accepted**: users wanting to re-verify after a fix run `/opsx:onboard`, see the verification output, exit. Mild friction; documented friction is option-C-equivalent without the explicit prose.

### D5: Inherit prior design decisions D9 (walkthrough style), D10 (lenses contextual), D11 (external-review abstract); preserve non-emission semantics; follow existing command-body duplication pattern + file #29 for future cleanup

Four sub-decisions, batched here because they each accept the inherited-from-prior-design or follow-existing-convention default:

**D5a — Walkthrough style: reference-leaning hybrid (inherits prior D9)**. 5 sections: Setup verification / Identity / Canonical-flow walkthrough (9-phase, 1 paragraph per phase) / Quick-reference table / Try-it nudge. No interactive demo, no auto-generated sample change. Audience: AI sessions cold-loading orbit + future-self after a context break + eventual collaborators / handoff readers. Audience framing hasn't changed since prior design; this stays.

**D5b — Drift items bundled with this change (3 items)**. All three drift items from `[[readme-install-section-staleness]]` touch the same `orbit-conventions` baseline requirement (`Overlay file disposition`):
- (1) `Orbit-modified` scenario list adjustment when openspec-onboard moves to Orbit-authored category (list stays at 9: `openspec-explore, openspec-propose, openspec-archive-change, openspec-apply-change, openspec-verify-change, openspec-continue-change, openspec-ff-change, openspec-new-change, openspec-bulk-archive-change`)
- (2) `Upstream-required primitive` scenario gets `openspec-sync-specs` named explicitly as the concrete example
- (3) `ff.md` / `fast-forward.md` naming divergence acknowledged in baseline (either enumerated in `Orbit-modified` for commands, or flagged as known limitation)

Item (1) is intrinsic to this change (the openspec-onboard category move). Items (2) and (3) are independent but touch the same baseline requirement — bundling = one-pass edit. Tighter scope than splitting into a separate doc-hygiene change cycle.

**D5c — `/opsx:onboard` does NOT emit run-summary JSON (preserves current non-emission semantics)**. Onboard is a single-pass reference read, not a workflow advancement. The new orbit-authored SKILL body asserts non-emission via a brief metadata note (mirroring the assertion currently in upstream SKILL's `# Orbit additions` section). This decision composes with the `orbit-run-summary-emit` `Emit scope` baseline requirement, which already lists `/opsx:onboard` as a non-emit command.

**D5d — Command-file duplication pattern preserved; cleanup tracked as [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29)**. The new `.claude/commands/opsx/onboard.md` duplicates the `.claude/skills/openspec-onboard/SKILL.md` body (differing only in frontmatter), following the existing pattern used by all 4 other orbit-authored command/skill pairs (`review`, `review-external`, `audit-drift`, `address-reviews`). Why: refactoring to a single-source pattern is a cross-command concern, not an orbit-onboard concern — bundling it here would expand scope inappropriately. Filed [#29](https://github.com/las-sal/openspec-orbit/issues/29) (2026-05-24) for the systematic cleanup across all 5 pairs (4 existing + onboard) when capacity allows. Until then: edit-time discipline is "update both files when revising onboard" — call this out explicitly in the spec scenarios for the new orbit-onboard capability.

## Considered & out

(none yet — populated as alternatives get rejected)

## References

### Closes (intended)

- [openspec-orbit#23](https://github.com/las-sal/openspec-orbit/issues/23) — `/opsx:onboard` walks through orbit's extended workflow.

### Related (open)

- [openspec-orbit#26](https://github.com/las-sal/openspec-orbit/issues/26) — install-script with version-check enforcement. Still doc-only post-this-change.
- [openspec-orbit#27](https://github.com/las-sal/openspec-orbit/issues/27) — Option 2 (drop `# Orbit additions` pattern). Tracked for future cycle.
- [openspec-orbit#28](https://github.com/las-sal/openspec-orbit/issues/28) — global `openspec-*` → `orbit-*` rename. Tracked for future cycle.
- [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29) — deduplicate orbit-authored command files and SKILL.md bodies (single-source pattern). Filed during this explore (D5d). Affects 5 command/skill pairs (4 existing + onboard); cross-cutting refactor, not bundled here.

### Archived precedents

- `openspec/changes/archive/2026-05-24-align-readme-install-with-v1-3-1/` — established documented post-install state that this change's Setup verification references as authority.
- `openspec/changes/archive/2026-05-24-lean-overlay-and-add-orbit-onboard/` — the original cluster-2 change whose chunks 4-5 (orbit-onboard SKILL body) deferral led to this explore. Its `explore.md` D8-D11 carry forward as inherited design.

### Memory

- [[openspec-1-3-1-actual-install-surface]] — verified sandbox findings (2026-05-23) on what `init --tools claude` actually produces.
- [[readme-install-section-staleness]] — remaining cluster-2 scope after `align-readme-install-with-v1-3-1` landed: skill-body work + 3 baseline-drift items.

### Baseline specs (relevant)

- `openspec/specs/orbit-conventions/spec.md` — pegged-engine framing, Upstream version pinning, Overlay file disposition, Install documentation describes actual install surface. The new orbit-onboard capability will compose with these.
