# Explore: address-reviews-defaults-and-decision-forks

## Premise

Bundle `#14` (auto-cascade ripple-flagged files by default, P0) + `#11` (per-finding walk default + opt-in batch mode, P1) + `#18` (treat disjunctive `recommendation` fields as explicit user decision forks, P2). All three flip v1 conservatism in `/opsx:address-reviews` that's hurting the workflow loop in dogfooding. They compose along the Step 3 walk axis:

- **#11 → walk granularity.** Per-finding walk is the default; batch-mode is opt-in. Provides the natural per-finding checkpoint surface the other two issues plug into.
- **#18 → decision-handling within each walk step.** When a finding's `recommendation` field is disjunctive ("either A or B" / "options: A vs B" / numbered alternatives), surface as A/B/discuss fork rather than collapsing to a passive line.
- **#14 → cascade after resolution.** Once the finding's resolution lands, ripple-flagged markdown in the change dir auto-cascades (no separate confirmation).

Same skill (`orbit-address-reviews`); same lifecycle (Step 3 walk + Step 3a–3e sub-steps); same v1-default-flip pattern.

**Empirical evidence**:
- #14: bootstrap-orbit-status-cli iter-N+1 review caught 5 WARNINGs already in iter-N's `ripple_flagged_files_aggregate` — ignored.
- #11: every iter of every recent change asked "walk them?" verbally; the v1 "batch by default" never fired in practice.
- #18: bootstrap-orbit-status-cli W1 (test coverage gap) had a two-option recommendation; the AI flattened it to a one-liner until user pushed back. Decision was made AFTER explicit user prompting — not surfaced.

## Decisions

### D1 — Change name

`address-reviews-defaults-and-decision-forks` (43 chars). Captures both axes of the bundle — the "active defaults" flip (#14 + #11) and the decision-fork surface (#18) — without overcommitting to internal terminology like "walk" or "cascade" that future reframings might shed.

### D2 — Cascade scope rules (#14)

**IN**:
- Change-dir markdown: `proposal.md`, `design.md`, `tasks.md`, and spec delta files at `openspec/changes/<name>/specs/<capability>/spec.md`
- Project-level governing docs: `CLAUDE.md`, `openspec/project.md`, root `*_convention.md`

**OUT**:
- `.orbit-runs/*` — frozen audit trail, never edited by cascade
- Baseline `openspec/specs/<capability>/spec.md` — sync-specs territory
- Code files (any extension other than `.md`) — opt-in only; future flag
- Cross-change ripples — limit to current change-dir + project-level surface

Rationale: spec-delta `.md` files in the change-dir are already mutated by Step 3's primary-finding fix; treating them differently for ripple is incoherent.

### D3 — Walk-mode / batch-mode UX trigger (#11)

`--batch` flag is the canonical signal for batch-mode. Verbal batch-intent is honored only when expressed in the user's **invocation message** ("/opsx:address-reviews … fix them all"); the AI treats this as a verbal `--batch`. Mid-walk step responses do NOT shift mode through phrase-detection — the only exception is a clearly command-shaped interruption like "go batch" or "switch to batch", which is treated as an explicit verbal `--batch` for the remainder of the walk.

Rationale: heuristic phrase-detection during the walk produces non-determinism that violates #11's "predictable defaults" goal. The invocation-line phrase and explicit command-shape interruptions are well-bounded; loose mid-walk phrases ("just fix them all") do NOT shift mode without explicit acknowledgment from the user.

### D4 — Spec delta shape (#14 / #11 / #18 across two capabilities)

Five operations across two capability deltas:

**`orbit-address-reviews`**:
- **MODIFIED** `Discover → triage → walk → ripple flag → report lifecycle` — body updated to codify walk-mode as default + cascade-on-resolve as the lifecycle's natural ending.
- **MODIFIED-with-rename** `Ripple flag without auto-cascade` → `Ripple cascade by default` — full body rewrite; scenarios refreshed; `--no-cascade` opt-out documented.
- **ADDED** `Walk-mode by default with --batch opt-in` — codifies #11's per-finding walk default + flag semantics + invocation-line verbal trigger.
- **ADDED** `Disjunctive recommendation fields surface as decision forks` — codifies #18's hybrid detection + fork-prompt UX + `[discuss]` escape.

**`orbit-run-summary-emit`** (resolution-log shape touches the universal-spine schema):
- **MODIFIED** `Address-reviews resolution-log shape` (existing requirement or new sub-section) — adds `walk_mode` top-level field, `recommendation_fork` per-resolution object, `ripple_cascade.applied / flagged_not_applied` split.

`Address-reviews command available` (recently MODIFIED in the auto-discover change) stays as-is — new requirements cover the new behavior; no need to re-touch.

Rationale: title `Ripple flag without auto-cascade` literally contradicts the new default; rename is non-negotiable. MODIFIED-with-rename (`## RENAMED FROM`) is one op vs REMOVED+ADDED's two.

### D5 — Backward compat / opt-out semantics

- **`--no-cascade`** — opts out of #14's auto-cascade. Ripple-flagged files still recorded in `flagged_not_applied`.
- **`--batch`** — opts INTO #11's batch-mode (walk-mode is the new default).
- **No flag for #18** — the `[discuss]` choice inside each fork prompt is the escape hatch. Don't design for hypothetical callers wanting the legacy "flatten" behavior.

Rationale: minimal flag surface; aligns with "don't add flags for scenarios that can't happen."

### D6 — Audit-trail / resolution-log JSON shape

Schema additions to `address-reviews-<TS>.json`:

```json
{
  "walk_mode": "per_finding" | "batch",
  "resolutions": [
    {
      "title": "...",
      "recommendation_fork": {
        "detected": true,
        "source": "structured" | "heuristic",
        "options_presented": ["A: ...", "B: ..."],
        "chosen": "A",
        "discuss_invoked": false
      },
      "ripple_cascade": {
        "applied": ["design.md", "tasks.md"],
        "flagged_not_applied": ["src/foo.ts (code; cascade off by default)"]
      }
    }
  ]
}
```

`recommendation_fork` omitted (or `"detected": false`) for findings with no disjunctive recommendation. `--no-cascade` populates only `flagged_not_applied`.

Rationale: `walk_mode` at top-level matches the per-run nature; `recommendation_fork` keyed to the finding matches per-step semantics; the `applied / flagged_not_applied` split is the minimum supporting both `--cascade` (default) and `--no-cascade` audit needs.

### D7 — Decision-fork detection rule (#18)

**Hybrid detection**:

- **Structured path** (orbit-emit pipelines): `orbit-review` and `orbit-audit-drift` add an optional `recommendation_options: [{label, body}]` field on each finding. Producers populate when the recommendation is genuinely disjunctive. Consumer (address-reviews) reads this directly.
- **Heuristic path** (external markdown findings): conservative regex pass over `**Description**:` content. Strict triggers only:
  - Numbered alternatives: `(A) ... (B)` or `1. ... 2.` shape
  - "either ... or" with clause-level branches
  - "**Options**:" prefix followed by a list
- **NOT** loose "or"-detection — too many false positives on prose like "fix now or later", "X or Y could happen."
- `recommendation_fork.source` records which detection path fired ("structured" vs "heuristic").

**Fork-prompt firing point**: in Step 3 walk, **after classify, before fix**, and **only when classify == "decision required"**. The fork prompt REPLACES the generic 2–4 option prompt in that path. `stale` / `trivial fix` / `unresolvable` classifications short-circuit before fork detection — they don't surface forks.

Rationale: decision-fork is a refinement of the existing "decision required" path, not a new lifecycle dimension. Smaller spec delta surface; simpler skill prose. Hybrid avoids blocking on producers we don't control (external AIs writing markdown) while keeping the cleaner structured contract for orbit's own pipeline.

**Scope note**: this expands the change to touch `orbit-review` and `orbit-audit-drift` minimally (add optional `recommendation_options` field to the finding schema — backward-compatible since it's optional). Captured in D4's spec-delta enumeration via the `orbit-run-summary-emit` MODIFIED operation, since the universal-spine schema is where the finding shape lives.

## Open questions

(all resolved — see Decisions section above)

## Considered & out

- **Closing #3's cascade portion** — landing this change supersedes #3's cascade subscope; #3's paste + severity portions remain separate. Worth a comment-update on #3 when this change archives, not a separate scope item.
- **Cross-change cascade** — explicitly OUT per #14. Limit cascade scope to current change-dir.
- **Restructuring the `recommendation` field in the review JSON schema as a hard breaking change** — keep backward compat. #18's structured path can ADD an optional `recommendation_options` array alongside the existing string field, not replace it.

## References

- GitHub issue #14: https://github.com/las-sal/openspec-orbit/issues/14 (P0 — cascade by default)
- GitHub issue #11: https://github.com/las-sal/openspec-orbit/issues/11 (P1 — per-finding walk default + batch mode)
- GitHub issue #18: https://github.com/las-sal/openspec-orbit/issues/18 (P2 — disjunctive recommendation fields as decision forks)
- GitHub issue #3: https://github.com/las-sal/openspec-orbit/issues/3 (cascade portion superseded by this change)
- Current `orbit-address-reviews` baseline: `openspec/specs/orbit-address-reviews/spec.md` (post-#10 sync)
- Bootstrap-orbit-status-cli empirical evidence: `openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/.orbit-runs/review-system-*.json` (W1 two-option recommendation flattened) + `ripple_flagged_files_aggregate` arrays from successive iters (5 WARNINGs in iter-N+1 already in iter-N's flags)
