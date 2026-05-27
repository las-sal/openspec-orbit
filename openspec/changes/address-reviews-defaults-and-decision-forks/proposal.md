## Why

`/opsx:address-reviews` ships with three conservative v1 defaults that hurt orbit's review-loop ergonomics in dogfooding:

1. **Batch-mode default** (#11) — the skill resolves all findings together rather than walking each one. In practice every recent change has prompted the user to verbally request a walk ("walk them?"), and the v1 batch default never fires.
2. **Ripple-flag-only** (#14, P0) — Step 3e lists ripple-affected files but does not edit them. `bootstrap-orbit-status-cli` iter-N+1 system review caught 5 WARNINGs that were already named in iter-N's `ripple_flagged_files_aggregate` and ignored — load-bearing ripple work silently fell through.
3. **Disjunctive recommendation flattening** (#18) — when a finding's `recommendation` field offers "either A or B" or numbered alternatives, the skill collapses it to a passive line in the report. `bootstrap-orbit-status-cli` W1 made its two-option decision only after explicit user pushback.

All three live on the same Step 3 walk axis of the same skill and flip in the same direction (active defaults). Bundling them is the cheapest path to a coherent UX: walk-mode provides the per-finding checkpoint surface, decision-fork prompts surface embedded choices once you're on a finding, and cascade applies the resolution to ripple-flagged markdown after each fix.

## What Changes

- **Walk-mode becomes the default** for Step 3 of the address-reviews lifecycle. Each finding gets its own checkpoint: pushback → classify → fix → ripple-cascade → remove-marker. `--batch` flag (or verbal `--batch` in the invocation message) opts INTO the legacy batch-mode behavior.
- **Ripple cascade becomes the default** when a finding resolves. Cascade scope is defined by exclusion — four lifecycle-invariant OUT categories (audit trail `.orbit-runs/*`; baseline `openspec/specs/`; cross-change directories; safe-exclusions `.git/`/`node_modules/`/`dist/`/`build/`). Any ripple-flagged file NOT in the OUT list is IN, regardless of extension. File type is NOT a discriminator: `.py`, `.swift`, `.c`, `.sh`, `.md`, dotfiles, configs — all eligible. The same safety mechanisms (pushback against current state, decision-fork prompts for "decision required" classifications, `--no-cascade` per-invocation opt-out) protect cascade edits uniformly across file types. `--no-cascade` opts out entirely; ripple-flagged files are still recorded in the audit-trail as `flagged_not_applied`.
- **Disjunctive recommendation fields surface as decision forks**. Detection is hybrid: structured (orbit-emit pipelines populate an optional `recommendation_options` array on each finding) + heuristic fallback (conservative regex for numbered alternatives, "either…or" with clause-level branches, or "**Options:**" prefix in external markdown). The fork prompt fires after classify, before fix, and only when classify == "decision required" — replacing the generic 2–4 option prompt in that path. No flag opt-out: `[discuss]` inside the prompt is the escape hatch.
- **Audit-trail JSON shape (`address-reviews-<TS>.json`)** extended with top-level `walk_mode` field, per-resolution `recommendation_fork` object (present only when detected), and per-resolution `ripple_cascade.applied / flagged_not_applied` split.
- **Finding-emit JSON shape** (`/opsx:review` and `/opsx:audit-drift` `findings[]` entries) gains an optional `recommendation_options: [{label, body}]` array — populated when the finding's recommendation is genuinely disjunctive. Address-reviews reads this for the structured-detection path; absence falls back to heuristic detection over the `recommendation` string.
- **BREAKING (within in-flight changes only)**: `Ripple flag without auto-cascade` requirement is renamed to `Ripple cascade by default`. No in-flight changes carry references to the old title (verified at propose time).

## Capabilities

### New Capabilities

(none — all changes land in existing capabilities)

### Modified Capabilities

- `orbit-address-reviews`: walk-mode default, cascade default, decision-fork surface, resolution-log JSON shape additions (D2–D7 consumer side)
- `orbit-review`: optional `recommendation_options[]` field on findings (D7 structured-path producer side)
- `orbit-audit-drift`: optional `recommendation_options[]` field on findings (D7 structured-path producer side)

## Impact

- **Affected commands**: `/opsx:address-reviews` (primary behavioral surface), `/opsx:review` and `/opsx:audit-drift` (small additive change to finding emit schema for the structured decision-fork path).
- **Affected skills**: `openspec-address-reviews/SKILL.md` (Step 3 lifecycle rewrite), `openspec-address-reviews/references/run-summary-schema.md` (resolution-log shape), `openspec-review/SKILL.md` + `openspec-audit-drift/SKILL.md` (optional `recommendation_options` field documented), `openspec-onboard/SKILL.md` (command-table description update if flag surface changes).
- **Affected docs**: `README.md` (address-reviews section + workflow examples), `.claude/commands/opsx/address-reviews.md` (command-mirror).
- **Backward compatibility**: existing `--from-file` flow continues to work; new fields (`walk_mode`, `recommendation_fork`, `ripple_cascade`) are additive on the resolution-log JSON. Closes issues #14 (P0), #11 (P1), #18 (P2); supersedes the cascade subscope of #3.
