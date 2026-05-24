## Context

This is the last change in the v0.1.0 cluster-2 wind-down sequence. The chain:

```
lean-overlay-and-add-orbit-onboard     →  cut to chunks 1+2 (2026-05-24); orbit-onboard SKILL body deferred
align-readme-install-with-v1-3-1       →  archived (2026-05-24); rewrote README install/update/uninstall + ADDED `Install documentation describes actual install surface` to orbit-conventions
orbit-onboard-follow-up    (this)      →  closes #23; final cluster-2 deliverable
```

The cut `lean-overlay-and-add-orbit-onboard` change's `explore.md` already developed the orbit-onboard skill design through decisions D8-D11 (replace body in-place at openspec-onboard; 5-section reference-leaning hybrid; lenses contextual; external-review abstract). The associated spec content had to be cut because its Setup verification scenarios referenced files that aren't installed by upstream init at v1.3.1 — a finding that triggered the cut. The just-archived `align-readme-install-with-v1-3-1` established documented post-install state (15 skills + 16 commands, sandbox-verified) AND added the `Install documentation describes actual install surface` baseline requirement to `orbit-conventions`, so the orbit-onboard Setup verification scenarios can now be re-grounded against the documented reality.

This change inherits 80% of the prior design and re-grounds the 20% (primarily Setup verification logic). Memory artifacts loaded by this explore: `[[openspec-1-3-1-actual-install-surface]]` (verified 2026-05-23) and `[[readme-install-section-staleness]]` (3 drift items the explore decided to bundle here).

## Goals / Non-Goals

**Goals:**

- Replace `.claude/skills/openspec-onboard/SKILL.md` body with 100% orbit-authored content (5-section reference-leaning hybrid walkthrough)
- Rewrite `.claude/commands/opsx/onboard.md` matching the new SKILL body verbatim (duplicate-pattern preservation per D-onboard-5)
- Add `orbit-onboard` capability spec defining the required SKILL structure + behavior
- Move `openspec-onboard` from `Orbit-modified` to `Orbit-authored` category in `orbit-conventions` `Overlay file disposition` baseline
- Bundle 3 baseline-drift items (per D-onboard-3) into this change's `orbit-conventions` modifications
- Preserve `/opsx:onboard` non-emission semantics (composes with existing `orbit-run-summary-emit` Emit scope)
- Close #23

**Non-Goals:**

- **Cross-command body-deduplication refactor** — affects 5 command/skill pairs across all orbit-authored commands. Tracked as [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29), filed during this explore. This change preserves the duplicate-body pattern; cleanup is its own scope.
- **Interactive guided tour for orbit-onboard** — rejected during explore D-onboard-1 (audience is AI cold-load + future-self + collaborators; reference-leaning fits).
- **Separate `/opsx:verify-setup` command** — rejected during explore D-onboard-2 (bundle as section 1 of onboard; mild re-verify-after-fix friction acceptable).
- **Emit run-summary JSON on `/opsx:onboard` invocation** — rejected during explore D5c (teaching session, not workflow advancement).
- **Option 2 work** — dropping the `# Orbit additions` pattern (orbit-modified skills becoming fully orbit-authored). Tracked as [#27](https://github.com/las-sal/openspec-orbit/issues/27); independent of this change.
- **Global `openspec-*` → `orbit-*` rename** — tracked as [#28](https://github.com/las-sal/openspec-orbit/issues/28); independent. The new orbit-authored SKILL stays at `openspec-onboard` to match sibling convention (per inherited prior D8).
- **`fast-forward.md` / `ff.md` consolidation** — this change acknowledges the divergence in baseline `Overlay file disposition` but does not resolve it (consolidating would touch multiple files + the fast-forward workflow surface; out of scope for the orbit-onboard rewrite).

## Decisions

### D-onboard-1: Reference-leaning hybrid 5-section walkthrough (inherits cut `lean-overlay`'s D9)

**Decision**: The orbit-authored SKILL body has 5 sections in this exact order:

1. **Setup verification** — bundled gate (D-onboard-2); hard-stop / warn / pass-with-layered-checks per D-onboard-4
2. **Identity statement** — post-pegging framing; enumerates orbit-distinctive layers (editorial review, drift audit, capture/lenses, JSON emission, three execution disciplines)
3. **Canonical-flow walkthrough** — 9-phase ASCII diagram + 1 paragraph per phase. Phases: explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive. Lenses introduced contextually in explore + review phases (inherits cut `lean-overlay`'s D10). External-review demoed abstractly with pointer to `openspec-review-external/SKILL.md` (inherits D11).
4. **Quick-reference command table** — every `/opsx:*` command in the overlay with one-line descriptions
5. **Try-it nudge** — two closing recommendations: named-mode `/opsx:explore <name>` for concrete-idea users, bare-mode `/opsx:explore` for orientation-only users

**Why this style** (inherited rationale, validated in current explore):
- **Audience**: AI sessions loading orbit context cold (primary) + future-self after context break + eventual collaborators / handoff readers. AI sessions benefit from parseable workflow docs, not interactive demos.
- **No demo change creation**: avoids cluttering the user's project with sandbox artifacts; the Try-it nudge invites real engagement (or bare-mode orientation) instead of synthetic.
- **Effort**: ~2-3 days of writing vs ~1-2 weeks for an interactive guided tour. Audience doesn't justify the heavier investment.

**Alternatives considered**:
- **Interactive guided tour** — rejected: over-built for AI-cold-load primary audience; future enhancement if reference-leaning proves insufficient.
- **Brutally minimal** (quick-reference table only) — rejected: loses the narrative flow that helps new users understand orbit's workflow shape.

### D-onboard-2: Setup verification bundled as section 1 of `/opsx:onboard` (not a separate command)

**Decision**: Setup verification stays as section 1 of the onboard SKILL body. No separate `/opsx:verify-setup` command is added.

**Why** (per explore D4):
- Lower surface area: no new command file + SKILL pair; no extra `Overlay file disposition` entry; no extra README install-section enumeration.
- Re-verify-after-troubleshooting use case is rare; users wanting to re-verify run `/opsx:onboard` and exit after section 1 — mild friction acceptable.

**Alternatives considered**:
- **Separate `/opsx:verify-setup` command** — rejected: surface cost outweighs re-verify-use-case benefit.
- **Documented re-invocation pattern in onboard prose** — rejected: adds documentation for marginal benefit; users can already re-invoke `/opsx:onboard` without explicit instructions.

### D-onboard-3: Setup verification hard-stops on overlay-incomplete; warns on prune-step-skipped; emits layered checks on pass

**Decision**: Three verification outcomes:

| Outcome | Triggers | Behavior |
|---|---|---|
| **Hard-stop** | Failure modes 1/2/4/5/7 from the explore taxonomy: overlay step skipped, partial overlay, wrong upstream version, wrong --tools profile, sync-specs missing | Halts the onboard session; emits a lumped "overlay incomplete — see README install section" message (per D-onboard-6 lumping decision); does NOT proceed to canonical-flow walkthrough |
| **Warn-continue** | Failure mode 3: prune step not run (`.claude/skills/feedback/` present) | Warns inline with `⚠`; emits remediation hint (`rm -rf .claude/skills/feedback`); proceeds to canonical-flow walkthrough — the warning is post-walkthrough cleanup, not a blocker |
| **Pass** | All checks succeed | Emits a layered ✓ output mirroring `orbit-conventions` `Overlay file disposition` 4 categories (post-this-change baseline: 9 upstream-modified + 5 orbit-authored + 1 upstream-required primitive + feedback/ absent); proceeds to canonical-flow walkthrough |

**Why hard-stop over warn-and-continue** (per explore D1):
- Lower author effort: walkthrough sections assume all commands runnable; no defensive "if missing skip" branching throughout sections 2-5.
- Better UX: a walkthrough teaching commands the user can't run materially misleads. Halting with "your setup is incomplete; here's what's missing" is honest.

**Why layered ✓ checks** (per explore D3):
- Pedagogical fit: `/opsx:onboard` is a teaching session. Layered output makes orbit's overlay-categorization structure visible at verification time, before the walkthrough.
- Mirrors baseline: the 4 check categories map 1:1 to `Overlay file disposition` 4 scenarios. Verification output IS the spec made executable.

### D-onboard-4: Lumped messaging for overlay-incomplete sub-modes (not distinguished per sub-mode)

**Decision**: When any overlay-incomplete sub-mode triggers (modes 1, 2, 5, 7 from the explore taxonomy), the SKILL emits a single "overlay incomplete; see README install section #2 (Overlay orbit)" message. Verification does NOT distinguish between "fully-skipped overlay" / "partial overlay" / "wrong --tools profile" / "sync-specs missing."

**Why** (per explore D2):
- Lower author effort: one detection branch + one prose message vs four sub-variant messages.
- Remediation is the same regardless: re-run the documented overlay sequence + prune step. The README install instructions are authoritative; verification points users there.
- Trade-off accepted: a user with sync-specs-only-missing gets the same message as a user with fully-skipped overlay. Both re-run the overlay step; partial-completion user re-runs slightly more than strictly necessary. Acceptable cost vs the prose maintenance.

### D-onboard-5: Command-body duplication preserved; cleanup tracked as #29

**Decision**: The new `.claude/commands/opsx/onboard.md` duplicates the `.claude/skills/openspec-onboard/SKILL.md` body verbatim (differing only in frontmatter — `category` + `tags` vs `license` + `compatibility` + `metadata`). Edit-time discipline: when revising either, update both. The spec for `orbit-onboard` codifies this discipline.

**Why preserve duplication for this change** (per explore D5d):
- Follows the existing pattern used by all 4 other orbit-authored command/skill pairs (`review`, `review-external`, `audit-drift`, `address-reviews`). Inconsistency with siblings is a worse signal than the drift risk.
- Refactoring to a single-source pattern is cross-cutting — affects 5 pairs (4 existing + onboard) and possibly the slash-command-loading behavior of Claude Code. Bundling that scope here would expand this change beyond #23.

**Filed for systematic cleanup**: [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29) — deduplicate orbit-authored command files and SKILL.md bodies (filed during this explore; 3 proposal options A/B/C captured in issue body).

### D-onboard-6: Non-emission of run-summary JSON preserved

**Decision**: `/opsx:onboard` does NOT emit a run-summary JSON when invoked. The new orbit-authored SKILL body includes a brief metadata note asserting this (mirroring the assertion currently in the upstream skill's `# Orbit additions` section).

**Why** (per explore D5c):
- `/opsx:onboard` is a single-pass reference read, not a workflow advancement. No workflow state changes; no artifacts produced in user project.
- Already codified in `orbit-run-summary-emit` baseline `Emit scope` requirement, which lists `/opsx:onboard` as a non-emit command. This change preserves that assertion via SKILL-body metadata note.
- Avoids creating new emit-scope work, new schema docs, new JSON write paths.

### D-conventions-1: `orbit-conventions` `Overlay file disposition` requirement MODIFIED with 3 drift fixes

**Decision**: Bundle 3 baseline-drift items into this change's `orbit-conventions` modification. The `Overlay file disposition` requirement's relevant scenarios get MODIFIED content:

1. **`Orbit-modified` scenario** — current list of 9 orbit-modified upstream skills stays at 9. `openspec-onboard` no longer in this category after this change archives (moves to `Orbit-authored`).
2. **`Orbit-authored` scenario** — current list expands by 1: `openspec-onboard` is added (alongside the existing 4: `openspec-review`, `openspec-review-external`, `openspec-audit-drift`, `openspec-address-reviews`).
3. **`Upstream-required primitive` scenario** — gets `openspec-sync-specs` named explicitly as the concrete example (replaces or supplements the current abstract phrasing).
4. **Commands-follow-same-framework scenario** — gets the `ff.md` (upstream-installed, untouched by overlay) / `fast-forward.md` (orbit-shipped, parallel to ff.md) naming-divergence acknowledged inline, with a note that consolidation is out of scope for now.

**Why bundle** (per explore D5b):
- All 4 modifications touch the same baseline requirement. One MODIFIED block edits all scenarios at once.
- Item 1 + 2 are intrinsic to this change (openspec-onboard category move). Items 3 + 4 are independent doc-hygiene but cost ~0 to bundle vs another change-cycle later.

## Risks / Trade-offs

- **Large SKILL body rewrite** — replacing 562 lines of upstream-bodied content with ~comparable orbit-authored content is the largest single-file edit in cluster-2. Risk: prose-quality issues across 5 sections that proposal review might catch but won't always. Mitigation: the SKILL body MUST be sanity-checked for clarity by re-reading cold after writing; user-validation handoff is a required apply task.

- **Verification false-positives** — Setup verification's hard-stop tripping in some legitimate state would lock users out of onboard for no good reason. Risk: user has a custom skill named `openspec-review-2.0` (or similar) but is missing `openspec-review` — partial-overlay false positive. Mitigation: verification logic is specific (checks file presence by exact name, not pattern); the README install steps document the exact names users should see; documented gotchas section in onboard's verification message points to install steps.

- **Verification false-negatives** — Setup verification incorrectly passes when something IS wrong. Risk: orbit-authored SKILL.md is present but corrupted (e.g., empty file). Mitigation: out of scope for v1; verification checks existence, not content correctness. Future enhancement: hash-based or frontmatter-presence check.

- **Audience drift** — the inherited audience framing (AI cold-load + future-self + collaborators) is what the design optimizes for. Risk: if the actual primary audience becomes "users who want a tutorial," the reference-leaning style is wrong-tool. Mitigation: filed as known limitation in design; if collaborator usage proves insufficient, an interactive-tour follow-up can extend the Try-it nudge section.

- **Duplicate-body drift** — until #29 is resolved, the SKILL and command files can drift. Risk: a future edit to `.claude/commands/opsx/onboard.md` doesn't propagate to `.claude/skills/openspec-onboard/SKILL.md` (or vice versa). Mitigation: spec for `orbit-onboard` codifies the discipline ("when revising either, update both"); #29 tracks systematic resolution.

- **Three drift items bundling** — bundling cluster-2 items 1-4 into this change's `orbit-conventions` modifications expands the diff. Risk: a focused #23 review gets bogged down in the drift-item details. Mitigation: drift items are scoped to one baseline requirement (`Overlay file disposition`); review can pass-1 the SKILL body and pass-2 the orbit-conventions delta separately if needed.
