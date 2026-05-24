## Why

The upstream `openspec-onboard` skill that orbit ships in its `.claude/skills/` overlay still has the upstream-bodied guided-tour content (562 lines walking users through `openspec new change` + 8 teaching phases) with only a small `# Orbit additions` section appended. This misses orbit's entire extended workflow surface — editorial review (`/opsx:review`, `/opsx:review-external`, `/opsx:address-reviews`), drift audit (`/opsx:audit-drift`), capture (lenses), run-summary JSON emission, the three execution disciplines, and the 9-phase canonical flow that orbit actually runs.

This change replaces both the `.claude/skills/openspec-onboard/SKILL.md` body and the matching `.claude/commands/opsx/onboard.md` command body with 100% orbit-authored content — a 5-section reference-leaning hybrid walkthrough (per the design inherited from the cut `lean-overlay-and-add-orbit-onboard` change's D8-D11 and re-grounded in the just-archived `align-readme-install-with-v1-3-1`'s documented install state). Closes [openspec-orbit#23](https://github.com/las-sal/openspec-orbit/issues/23) and completes the cluster-2 wind-down.

## What Changes

**Replace `openspec-onboard` skill body** (100% orbit-authored):
- 5 sections: Setup verification / Identity statement / Canonical-flow walkthrough (9 phases) / Quick-reference table / Try-it nudge
- Setup verification hard-stops on overlay-incomplete failures (modes 1/2/5/7 from explore D1); warns on prune-step-skipped (mode 3); produces layered ✓ checks on pass (mirrors `Overlay file disposition` 4 categories per explore D3)
- Setup verification stays bundled as section 1 of `/opsx:onboard` (no separate `/opsx:verify-setup` command per explore D4)
- Setup verification emits lumped error messages for overlay-incomplete sub-modes, not distinguished per sub-mode (per explore D2)
- Identity statement uses post-pegging framing; enumerates orbit-distinctive layers
- Canonical-flow walkthrough: 9-phase ASCII diagram + 1 paragraph per phase; lenses introduced contextually in explore + review phases; external-review demoed abstractly with pointer to `openspec-review-external/SKILL.md`
- Quick-reference table lists every `/opsx:*` command in the overlay with one-line descriptions
- Try-it nudge has two closing recommendations: named-mode `/opsx:explore <name>` for users with a concrete idea; bare-mode `/opsx:explore` for orientation-only users

**Rewrite `.claude/commands/opsx/onboard.md`** to match the new SKILL.md body verbatim (duplicate-pattern preservation per explore D5d; cross-cutting body-deduplication cleanup tracked separately as [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29)).

**Update `orbit-conventions` baseline `Overlay file disposition` requirement** (3 drift items bundled per explore D5b):
- Move `openspec-onboard` from `Orbit-modified` category to `Orbit-authored` category in the disposition scenarios
- Name `openspec-sync-specs` explicitly as the concrete example in the `Upstream-required primitive` scenario
- Drop `.claude/commands/opsx/fast-forward.md` from orbit's overlay (verbatim duplicate of upstream's `ff.md` modulo whitespace) and codify the principle as a new `Verbatim upstream files not in orbit's overlay` scenario in the modified requirement

**Preserve `/opsx:onboard` non-emission semantics** (per explore D5c): the new orbit-authored SKILL asserts in a brief metadata note that `/opsx:onboard` does NOT emit run-summary JSON (composes with the existing `orbit-run-summary-emit` `Emit scope` baseline requirement that already lists onboard as a non-emit command).

## Capabilities

### New Capabilities
- `orbit-onboard`: defines the orbit-authored onboarding skill — 5-section reference-leaning hybrid walkthrough (Setup verification / Identity / Canonical-flow / Quick-reference / Try-it nudge). Slash-command surface `/opsx:onboard`. Bundled setup verification with hard-stop / warn-and-continue / pass-with-layered-checks semantics. Non-emission of run-summary JSON. Command-body duplication discipline.

### Modified Capabilities
- `orbit-conventions`: MODIFIES the `Overlay file disposition` requirement — updates `Orbit-modified` + `Upstream-required primitive` + Commands-follow-same-framework scenarios to reflect openspec-onboard moving to Orbit-authored and explicit naming of `openspec-sync-specs`; ADDS a new `Verbatim upstream files not in orbit's overlay` scenario codifying that orbit ships ONLY files it authors or modifies (the immediate trigger is dropping `.claude/commands/opsx/fast-forward.md` from orbit's overlay because it was byte-identical to upstream's `ff.md`).

## Impact

- **Skills**: `.claude/skills/openspec-onboard/SKILL.md` fully rewritten (~562 lines upstream-bodied → ~comparable size 100% orbit-authored). Upstream body is removed; the slash command's behavior changes entirely.
- **Commands**: `.claude/commands/opsx/onboard.md` fully rewritten (~548 lines → matching SKILL.md body). Duplicate-pattern preservation per [#29](https://github.com/las-sal/openspec-orbit/issues/29).
- **Specs baseline**: new `orbit-onboard` capability spec created on archive sync; `orbit-conventions` `Overlay file disposition` requirement gets MODIFIED scenarios.
- **No CLI changes**: `/opsx:onboard` slash-command name and invocation pattern unchanged; behavior body is what changes.
- **No runtime changes** outside the SKILL/command bodies and one baseline spec requirement: existing changes, archives, lenses, .orbit-runs/ files are untouched.
- **Closes [#23](https://github.com/las-sal/openspec-orbit/issues/23)** — the cluster-2 deliverable that was deferred from `lean-overlay-and-add-orbit-onboard`.
- **Surfaces [#29](https://github.com/las-sal/openspec-orbit/issues/29)** as new follow-up: cross-command body-deduplication refactor (5 command/skill pairs total, including the new onboard pair). Not bundled here.
- **Reads against**: `[[openspec-1-3-1-actual-install-surface]]` memory for Setup verification's reference data; just-archived `align-readme-install-with-v1-3-1` for documented post-install state that verification cites as authority.
