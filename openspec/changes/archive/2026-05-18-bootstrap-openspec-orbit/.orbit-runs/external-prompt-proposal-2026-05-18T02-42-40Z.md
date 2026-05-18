# External Review: bootstrap-openspec-orbit (iteration 1)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning. The authoring AI did its own internal review pass and resolved 12 findings before this handoff. Apply pushback discipline to *that* outcome: re-examine whether the resolutions actually closed the issues or just renamed them.

## Repo

`https://github.com/las-sal/openspec-orbit` (private; ask `las-sal` for access if needed, or work from a local checkout)

This repo IS the change. It dogfoods orbit on orbit itself — the change `bootstrap-openspec-orbit` proposes building orbit (the openspec overlay), and the proposal was authored using orbit's own (planned) workflow. Be alert for self-referential inconsistencies — places where the meta-design described in specs doesn't match what's actually done in `explore.md`, sketches, or the change layout itself.

## Project context (read first)

- `README.md` — full workflow + command reference + project orientation. Read this first to understand what orbit is.
- `openspec/config.yaml` — openspec configuration (mostly default in this repo)
- (no `CLAUDE.md` yet, no `openspec/project.md` yet, no `*_convention.md` files yet — this change establishes the orbit overlay before adopting it elsewhere)
- (no `openspec/lenses/` content yet — empty in this repo)
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history; the prior internal review pass and its resolutions are here

## Cycle context

- **Iteration**: 1 (first external review for this change in proposal mode)
- **Prior internal findings still open**: 0 (12 surfaced by internal `/opsx:review-proposal` and all resolved before this handoff — see `.orbit-runs/address-reviews-2026-05-18T02-15-00Z.json` for the resolution log)
- **Prior external findings still open**: 0 (first external pass)
- **Resolved since last review** (12 internal findings, all addressed):
  - WARNING: defined "substantive decisions" threshold (`orbit-explore-modifications`)
  - WARNING: added marker classification heuristics (`orbit-address-reviews`)
  - WARNING: added pushback decision procedure (`orbit-address-reviews`)
  - WARNING: embedded literal external-review prompt template (`orbit-review-external`)
  - 8 SUGGESTIONs: task back-references convention, distribution model + naming taxonomy as requirements, light-vs-heavy enforcement boundary, Pass 6 Gap Hunt operational heuristic, README on `.orbit-runs/` committed status, structured-claim extraction in audit-drift

Do not push back on stale findings from the prior internal pass — those were addressed in the resolution log. Just flag what *you* observe.

## What to read for THIS review (`--as proposal` mode)

- `openspec/changes/bootstrap-openspec-orbit/proposal.md` — motivation, scope, capabilities list (9 capabilities)
- `openspec/changes/bootstrap-openspec-orbit/design.md` — context, goals, 12 design decisions, risks, open questions
- `openspec/changes/bootstrap-openspec-orbit/specs/<capability>/spec.md` — 9 capability spec files (each describes a unit of orbit's behavior):
  - `orbit-review-proposal`
  - `orbit-review-system`
  - `orbit-review-external`
  - `orbit-audit-drift`
  - `orbit-address-reviews`
  - `orbit-explore-modifications`
  - `orbit-propose-modifications`
  - `orbit-archive-modifications`
  - `orbit-conventions`
- `openspec/changes/bootstrap-openspec-orbit/tasks.md` — 105 implementation tasks across 12 groups
- `openspec/changes/bootstrap-openspec-orbit/explore.md` — historical record of the exploration that produced this proposal (Premise / Decisions / Open questions / Considered & out / References)
- `openspec/changes/bootstrap-openspec-orbit/sketches/*.md` — 8 detailed sketches per command (proposal-side + system-side review commands, audit-drift, address-reviews, review-external, explore mods, propose mods, archive mods)
- `openspec/specs/` is empty — no baseline to check against (this change establishes the baseline)

## What to look for

Apply the 9 review-proposal passes:

1. **Structure & Delta Integrity** — All artifacts present? Delta sections valid (`## ADDED Requirements` etc.)? Task back-references where useful? Does `openspec validate bootstrap-openspec-orbit` pass?
2. **Internal Coherence** — Proposal aligns with design aligns with specs aligns with tasks? No scope creep? Numbers / counts consistent? (Counts-drift was a documented home-env lesson.)
3. **Cross-Doc Coherence** — README.md still accurate after these specs? Updates needed? (No CLAUDE.md / project.md / conventions in this repo yet — those start empty.)
4. **Archive Consistency** — Skip with note: `openspec/specs/` is empty in this repo; this change establishes the baseline.
5. **Codegen Readiness** — Implicit requirements? Decisions left to codegen? "TBD" or "or similar" without picks? Ambiguous units / types / formats? Could a fresh AI implement each spec requirement without inventing defaults?
6. **Gap Hunt (generative completeness probe)** — For each spec requirement, ask: are there unstated assumptions an implementer would have to invent? Error and edge-case paths specified? State transitions explicit? Could two implementers produce the same behavior from this spec?
7. **Drift Hunt** — Old vocabulary lingering? Recent renames in this exploration to look for:
   - `review-code` → `review-system`
   - `audit-residue` → `audit-drift`
   - `<!-- REVIEW: -->` → `@review:`
   - `openspec/system/` → `openspec/lenses/`
   - `openspec-review` (repo name) → `openspec-orbit`
   - `/opsx:handoff` → `/opsx:review-external`
   Any leakage in active content? (Historical references in `Considered & out` or alternatives sections are legitimate; only flag if they appear in current-state content.)
8. **Inline Review Marker Residue** — Any `@review:` markers still present in the change-dir artifacts (excluding the orbit-conventions and orbit-address-reviews specs which document the marker syntax)? Markers must be addressed before apply.
9. **Pre-Handoff Sweep** — Small things easily missed on a first read. Final eye for anything that would embarrass the author post-apply.

**A specific concern worth your attention**: this change dogfoods orbit's own workflow. Be alert for self-referential inconsistencies — places where the meta-design described in specs doesn't match what's actually done in `explore.md`, sketches, or the change layout itself.

**Another specific concern**: the prior internal review surfaced and resolved 12 findings. Some of those resolutions added scenarios to specs. Sanity-check whether the resolutions actually closed the issues or just added text — look at the underlying intent, not just whether new scenario blocks exist.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md`

Where `<TS>` is today's timestamp in ISO format (e.g., `2026-05-18T10-30-00Z`). Pick a fresh timestamp so this file doesn't overwrite prior reviews.

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 1)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

### <Next finding>
**File**: <path>:<line>
**Description**: ...

## WARNING

### ...

## SUGGESTION

### ...

## Notes

<Optional: overall impression, broader concerns, what reads as well-designed.>
```

If your environment doesn't support file writes (chat-only interface), output the markdown directly so the user can save it to the path above.

## After completing the review

If your environment supports git operations, commit and push your findings file so the authoring AI can pick it up without manual intervention:

```bash
git add openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md
git commit -m "External review (proposal, iter 1): bootstrap-openspec-orbit

<one-line summary: severity counts + headline finding if any>"
git push
```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
