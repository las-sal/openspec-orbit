# External Review Request: `address-reviews-defaults-and-decision-forks` (system mode, iter-1 external)

You are running a second-opinion **system-mode** review of an OpenSpec change in the **openspec-orbit** repo. This is a post-apply review — the implementation has landed; your job is to validate the implementation against the spec and the wider product state.

## How to read this prompt

This file is self-contained — read it top to bottom, then pull the repo (`https://github.com/las-sal/openspec-orbit`) and read the files it references. Write your findings file at the path specified in the **Output** section below, then **commit and push it back** so the orbit user can pull for ingest.

## Cycle context

The change has gone through:
- **2 internal proposal reviews + 2 walks** (iter-1 found a CRITICAL on cascade-scope that drove the "Option D" reframe; iter-2 fresh-subagent caught residue; all 12 findings resolved)
- **1 external proposal review by GPT-5 Codex** (`.orbit-runs/external-proposal-2026-05-27T03-56-14Z.md`) — caught 2 codegen-readiness WARNINGs + 1 SUGGESTION; all resolved
- **`/opsx:apply`** ran in 4 chunks (35 AI-doable tasks; 8 user-validation handoff tasks remain intentionally unchecked per orbit convention)
- **1 internal system review (fresh subagent)** — caught 2 stale Step 3d → Step 3e cross-references in sibling reference docs + 1 pre-existing audit-drift drift fixed inline; all 3 resolved

This is **iter-1 external system review** — first cross-AI look at the post-apply state. The Option D reframe + the lifecycle reorder + the 3-issue bundle (cascade-by-default + walk-mode-default + decision-fork detection) is now landed; your fresh perspective is the point.

**Do not just re-surface prior findings** (they're all marked resolved). But also: don't anchor on them either — read the current state fresh.

## What the change does (TL;DR)

Bundles 3 GitHub issues into `/opsx:address-reviews`:

- **#14 (P0)** — cascade by default: `/opsx:address-reviews` Step 3d auto-applies ripple-flagged edits to IN-set files. `--no-cascade` opts out. OUT = 4 lifecycle-invariant categories: audit-trail (`.orbit-runs/`), baseline specs (`openspec/specs/`), cross-change/archive dirs, safe-exclusions (`.git/`, `node_modules/`, `dist/`, `build/`).
- **#11 (P1)** — walk-mode default: per-finding lifecycle (pushback → classify → fork-detection → fix → cascade → remove). `--batch` opts INTO batch-mode via 3 triggers: flag in argv, verbal trigger in invocation message, or bare command-shape mid-walk interruption.
- **#18 (P2)** — decision-fork detection: gated on `classify == "decision required"`; hybrid (structured `recommendation_options[]` from producer JSON, heuristic fallback over recommendation prose); `[discuss]` escape hatch.

Plus resolution-log JSON shape additions (`walk_mode` + `recommendation_fork` + per-resolution `ripple_cascade.applied/flagged_not_applied`; replaces v1 top-level `ripple_flagged_files_aggregate`).

## Repository + paths

- Repo URL: `https://github.com/las-sal/openspec-orbit`
- Project context: `CLAUDE.md` at repo root (orbit's identity + three execution disciplines)
- Change directory: `openspec/changes/address-reviews-defaults-and-decision-forks/`
- Spec deltas in the change:
  - `specs/orbit-address-reviews/spec.md` — 5 ops (MODIFIED lifecycle, RENAMED+MODIFIED cascade, 3 ADDED: walk-mode, decision-forks, resolution-log shape)
  - `specs/orbit-review/spec.md` — 1 ADDED (optional `recommendation_options` field)
  - `specs/orbit-audit-drift/spec.md` — 1 ADDED (same field)
- Baseline specs (canonical state — DO NOT propose direct edits; baselines mutate only at archive via sync-specs):
  - `openspec/specs/orbit-address-reviews/spec.md`
  - `openspec/specs/orbit-review/spec.md`
  - `openspec/specs/orbit-audit-drift/spec.md`
  - `openspec/specs/orbit-run-summary-emit/spec.md` (downstream consumer of the emit schema; cross-doc consistency check)
  - `openspec/specs/orbit-conventions/spec.md`
- Implementation surface (the 13 files this change modified):
  - `.claude/skills/openspec-address-reviews/SKILL.md` (primary — Step 3 + Step 3b.5 + Step 3d cascade + frontmatter)
  - `.claude/skills/openspec-address-reviews/references/run-summary-schema.md` (v2 schema)
  - `.claude/skills/openspec-address-reviews/references/internal-findings-format.md` (recommendation_options + parser-contract row)
  - `.claude/skills/openspec-address-reviews/references/external-findings-format.md` (Step 3e cross-ref fix)
  - `.claude/commands/opsx/address-reviews.md` (mirror)
  - `.claude/skills/openspec-review/SKILL.md` (disjunctive recommendations producer section)
  - `.claude/skills/openspec-review/references/run-summary-schema.md` (recommendation_options field)
  - `.claude/commands/opsx/review.md` (mirror)
  - `.claude/skills/openspec-audit-drift/SKILL.md` (disjunctive recommendations + audit-drift drift fix)
  - `.claude/skills/openspec-audit-drift/references/run-summary-schema.md` (recommendation_options field)
  - `.claude/commands/opsx/audit-drift.md` (mirror)
  - `.claude/skills/openspec-onboard/SKILL.md` (high-level address-reviews bullet + Phase 4 + command table row)
  - `.claude/commands/opsx/onboard.md` (mirror)
  - `README.md` (`/opsx:address-reviews` section)

## Prior runs in `.orbit-runs/` (read for context)

```
review-proposal-2026-05-26T18-00-00Z.json     internal proposal iter-1 (1 CRITICAL → Option D reframe)
address-reviews-2026-05-26T18-30-00Z.json     iter-1 walk
review-proposal-2026-05-26T19-00-00Z.json     internal proposal iter-2 (fresh subagent)
address-reviews-2026-05-26T19-30-00Z.json     iter-2 walk
external-proposal-2026-05-27T03-56-14Z.md     GPT-5 Codex iter-1 external
address-reviews-2026-05-27T04-15-00Z.json     iter-3 walk (Codex findings)
apply-2026-05-27T04-30-00Z.json               Chunk 1 emit
apply-2026-05-27T04-45-00Z.json               Chunk 2 emit
apply-2026-05-27T05-00-00Z.json               Chunk 3 emit
apply-2026-05-27T05-15-00Z.json               Chunk 4 emit
review-system-2026-05-27T05-30-00Z.json       internal system iter-1 (fresh subagent)
address-reviews-2026-05-27T05-45-00Z.json     iter-4 walk (system findings)
```

The most recent address-reviews JSON (`05-45-00Z`) post-dates the most recent apply (`05-15-00Z`), so the implementation should now be coherent with the spec.

## Your job — system-mode passes

System mode wraps `openspec-verify-change` as Pass 0 + adds 6 system-wide passes. Each pass produces findings (possibly zero) tagged CRITICAL / WARNING / SUGGESTION. Bias toward lower severity when uncertain; bias toward false-negative (miss > misfire).

**Pass 0 — verify-change** (delegated structural check). `openspec validate --strict` passes. Confirm tasks state matches expected: 35 [x] + 8 [ ] is the CORRECT final-chunk state — per `orbit-conventions`'s `Apply per-chunk-end emission` semantic rule, `tasks_remaining > 0 AND user_validation_remaining == tasks_remaining` with `chunk_complete: true` is expected when user-validation tasks exist. NOT a CRITICAL.

**Pass 1 — Baseline Compliance**. Does this change break any archived `openspec/specs/<capability>/spec.md` requirement? The new spec deltas are not yet merged into baseline (archive does that via sync-specs). Read the baseline specs for the 3 modified capabilities + `orbit-conventions` + `orbit-run-summary-emit`. Verify the change doesn't contradict existing baseline requirements. Specifically check:
- Does the new `Resolution-log JSON shape extensions` requirement violate the universal-spine contract from `orbit-conventions`?
- Does the `walk_mode` / `walk_mode_source` field set fit cleanly with `orbit-run-summary-emit`'s expectations?
- Does the `recommendation_options` finding-shape addition to `orbit-review` + `orbit-audit-drift` break any downstream baseline-tooling assumption (e.g., orbit-status sibling-repo)?

**Pass 2 — Cohesion**. Callers / dependents outside the change's tasks list — does the new behavior have ripple effects in code or skill prose that weren't surfaced? For example:
- Does anything else in the repo grep for `ripple_flagged_files_aggregate` (the deleted v1 field) and now break?
- Does any other SKILL.md or doc reference "Step 3d of openspec-address-reviews" and now point at `Ripple cascade` instead of `Remove marker` (the lifecycle reorder)?
- Does anything reference the old `Ripple flag without auto-cascade` requirement name?

**Pass 3 — Surface Walk**. For every CLI / public-function surface affected, walk through and confirm coherence. The change touches: `/opsx:address-reviews` (primary), `/opsx:review` (producer affordance), `/opsx:audit-drift` (producer affordance), `/opsx:onboard` (docs). Each one should describe the new behavior consistently. Spec → SKILL.md → command-mirror chain should agree end-to-end.

**Pass 4 — Perspective Reviews**. `openspec/lenses/perspectives.md` (if present) names callers worth simulating from. If absent or empty, note "lens absent; skip Pass 4". If present, for each perspective: do their needs work with the new defaults?

**Pass 5 — Critical-Path Scan**. `openspec/lenses/critical-paths.md` (if present) lists flows worth walking end-to-end. If absent, note "lens absent; skip Pass 5". If present, for each flow: trace through the change's impact.

**Pass 6 — Drift / Residue**. Sweep for:
- **Category 1 (Vocabulary residue)**: pre-Option-D terms eliminated by the reframe — `code file; cascade off`, `in-scope markdown`, `project-level governing docs`, `project-level-skill umbrella`, `markdown-only`, `no auto-cascade in v1`. Legitimate exceptions: `design.md` "Alternatives Considered" section (history); `.orbit-runs/` and `archive/` (immutable). Anything ELSE is residue.
- **Category 2 (Lens staleness)**: if `openspec/lenses/*` exists, check refs.
- **Category 3 (Cross-doc consistency)**: do `CLAUDE.md` / `README.md` / governing docs agree with current implementation? Does the spec delta agree with `SKILL.md`?
- **Category 4 (Archive coherence)**: N/A — this change hasn't been archived yet.

## Specific probes worth investigating

The post-apply state is the highest-risk surface:

1. **Lifecycle reorder (3d/3e swap) ripple in dependent docs**. The fresh-subagent internal system review caught 2 `Step 3d` cross-references in `references/external-findings-format.md` + `internal-findings-format.md` that were updated to `Step 3e`. Are there OTHER cross-references to "Step 3d" or "Step 3e" elsewhere in the repo that need adjustment? (Sibling skill files? README? CLAUDE.md? archive folder?)

2. **`Address-reviews command available` requirement in baseline** vs the new lifecycle. The baseline requirement (recently MODIFIED in `address-reviews-auto-discovers-internal-json`, archived 2026-05-26) describes the auto-discovery + positional-resolution logic. After this change's lifecycle reorder + cascade-by-default, does auto-discovered JSON walk in walk-mode by default? The baseline requirement doesn't mention `walk_mode` — does the new requirement implicitly govern auto-discovered walks too?

3. **v1 → v2 schema migration**. v1 `ripple_flagged_files_aggregate` was top-level; v2 splits per-resolution `ripple_cascade.applied/flagged_not_applied`. Does the migration documentation cover the structural difference adequately? Are there any downstream consumers (e.g., `orbit-status` per the cross-repo references) that need to handle both shapes?

4. **`recommendation_options` field shape consistency across all 7 files** that mention it. Producer specs say `[{label, body}]` with ≥2 entries + non-empty label/body. Consumer audit-log uses identical structured shape (`options_presented: [{label, body}]`). SKILL.md prose + reference docs + command mirrors should all agree.

5. **Decision-fork detection heuristic strictness**. The spec says heuristic triggers ONLY on: numbered alternatives (`(A) … (B)`, `1. … 2.`, `[A] … [B]`), "Either … or" with clause-level branches, "Options:" prefix (both bold variants). Are these documented consistently? Is the "NOT trigger" guidance (loose "or" in prose) clear enough?

6. **`--batch` verbal trigger detection scope**. The spec restricts verbal trigger to the invocation MESSAGE only (not subsequent walk-step responses). The exception is bare command-shape mid-walk messages (`go batch`, etc.). Is this boundary well-defined or could two implementers disagree about what counts as "command-shape"?

7. **Apply-phase chunk emit fields**. The apply JSONs (`apply-<TS>.json`) emit `tasks_completed`, `tasks_remaining`, `user_validation_remaining`, `chunk`, `chunk_name`, `chunk_complete`. Does the implementation correctly populate these per `orbit-run-summary-emit/spec.md`?

8. **The audit-drift drift fix (S1 from internal-system iter-1)**. The `--from-file <this-json>` argument was removed from the change-scoped recommendation per the baseline spec. Is the fix complete? Are there other audit-drift code paths emitting the obsolete arg?

These are PROBES, not guaranteed findings. If you don't find anything substantive, "None" is the right answer.

## Pushback discipline

**Do NOT apply pushback at your end.** That's address-reviews' job at ingest time. Flag everything you observe; if you think a prior finding still applies (despite being marked resolved), say so — address-reviews will re-verify against current state.

## Output format

Write your findings to: `openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-system-<YOUR-TS>.md` where `<YOUR-TS>` is the current UTC time in `YYYY-MM-DDTHH-MM-SSZ` format.

Format MUST match this exact structure (orbit external-review markdown format; address-reviews `--from-file` parses it):

```markdown
# External Review: address-reviews-defaults-and-decision-forks (iteration 1)

**Reviewer**: <your model name>
**Date**: 2026-05-27

## CRITICAL

### <Finding title>
**File**: <repo-relative-path>:<line>
**Description**: <what's wrong + specific recommendation>

(If no CRITICAL findings, write the single body line `None.`)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(If no WARNING findings, write `None.`)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(If no SUGGESTION findings, write `None.`)

## Notes

<Optional: overall impression, broader concerns, or recommendation for next step>
```

Format constraints:
- Severity headers exactly `## CRITICAL`, `## WARNING`, `## SUGGESTION` (case-sensitive)
- Each finding's title under `### ` (3 hashtags + space)
- `**File**:` and `**Description**:` field labels exact
- `None.` (with trailing period) for empty severity sections
- The `## Notes` section is optional

Multi-line descriptions are fine within the `**Description**:` field — just continue on subsequent lines.

## When you're done

After writing your findings file, **commit and push it back to the remote** so the orbit user can pull it for ingest. Suggested commit pattern (matches prior orbit external-review commits):

```
git add openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-system-<YOUR-TS>.md
git commit -m "External review (system, iter 1): address-reviews-defaults-and-decision-forks"
git push
```

The orbit user will then `git pull` and run `/opsx:address-reviews address-reviews-defaults-and-decision-forks --from-file openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-system-<YOUR-TS>.md` (or rely on auto-discovery, which will resolve to your file if it's the most recent in `.orbit-runs/`).

Thank you for the second opinion.
