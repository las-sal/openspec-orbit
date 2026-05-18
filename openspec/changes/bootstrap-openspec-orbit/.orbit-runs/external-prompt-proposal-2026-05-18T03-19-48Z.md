# External Review: bootstrap-openspec-orbit (iteration 2)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning. The authoring AI ran an internal review pass and an external review pass (GPT-5 Codex, iteration 1) and resolved all findings before this handoff. Apply pushback discipline to *those* outcomes: re-examine whether the resolutions actually closed the issues or just renamed them.

## Repo

Local checkout at `/Users/sal/code/openspec-review/` (this is the openspec-orbit dev sandbox; the repo is `https://github.com/las-sal/openspec-orbit`).

This repo IS the change. It dogfoods orbit on orbit itself — the change `bootstrap-openspec-orbit` proposes building orbit (the openspec overlay), and the proposal was authored using orbit's own (planned) workflow. Be alert for self-referential inconsistencies — places where the meta-design described in specs doesn't match what's actually done in `explore.md`, sketches, or the change layout itself.

## Project context (read first)

- `README.md` — full workflow + command reference + project orientation. Read this first to understand what orbit is.
- `openspec/config.yaml` — openspec configuration (mostly default in this repo)
- (no `CLAUDE.md` yet, no `openspec/project.md` yet, no `*_convention.md` files yet — this change establishes the orbit overlay before adopting it elsewhere)
- (no `openspec/lenses/` content yet — empty in this repo)
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history; review the prior internal and external passes

## Cycle context

- **Iteration**: 2 (second external review for this change in proposal mode)
- **Prior internal findings still open**: 0 (12 surfaced and all resolved — see `address-reviews-2026-05-18T02-15-00Z.json`)
- **Prior external findings still open**: 0 (7 surfaced by GPT-5 Codex in iteration 1; all resolved — see `address-reviews-2026-05-18T03-13-46Z.json`)
- **Resolved since last external review** (7 findings from codex, all addressed):
  - WARNING: stale chat-paste contract in 3 places (spec, design.md D12, sketch) — updated to file-backed contract
  - WARNING: "and nothing else" vs. uncommitted-changes warning contradiction — reconciled (warning is allowed 4th item)
  - WARNING: address-reviews sketch said `--from-file` was v2 — corrected to v1
  - WARNING: review-external sketch's proposal-mode pass list missing Pass 8 (Inline Review Marker Residue) — added
  - WARNING: README orphan `@review` note (dash instead of colon) about crystallized mode — addressed (crystallized reframed as a transition, not a separate invocation)
  - WARNING: 18 README links pointed to pre-propose staging path (`openspec/explore/...`) — all updated to `openspec/changes/...`
  - SUGGESTION: archive spec typo "Pass-archive audit" → "Pre-archive audit"

Do not push back on stale findings from prior passes — those were addressed. Just flag what *you* observe.

## What to read for THIS review (`--as proposal` mode)

- `openspec/changes/bootstrap-openspec-orbit/proposal.md` — motivation, scope, capabilities list (9 capabilities)
- `openspec/changes/bootstrap-openspec-orbit/design.md` — context, goals, 12 design decisions, risks, open questions
- `openspec/changes/bootstrap-openspec-orbit/specs/<capability>/spec.md` — 9 capability spec files:
  - `orbit-review-proposal`, `orbit-review-system`, `orbit-review-external`, `orbit-audit-drift`, `orbit-address-reviews`, `orbit-explore-modifications`, `orbit-propose-modifications`, `orbit-archive-modifications`, `orbit-conventions`
- `openspec/changes/bootstrap-openspec-orbit/tasks.md` — 105 implementation tasks across 12 groups
- `openspec/changes/bootstrap-openspec-orbit/explore.md` — historical record of the exploration
- `openspec/changes/bootstrap-openspec-orbit/sketches/*.md` — 8 detailed sketches per command
- (optional) GPT-5 Codex's iteration-1 findings: `external-proposal-2026-05-18T03-00-12Z.md` — useful for "what did the previous external reviewer catch that I might also catch or that they missed?"

## What to look for

Apply the 9 review-proposal passes:

1. **Structure & Delta Integrity** — All artifacts present? Delta sections valid (`## ADDED Requirements` etc.)? Task back-references where useful? Does `openspec validate bootstrap-openspec-orbit` pass?
2. **Internal Coherence** — Proposal aligns with design aligns with specs aligns with tasks? No scope creep? Numbers / counts consistent?
3. **Cross-Doc Coherence** — README.md still accurate after these specs and the recent updates? (No CLAUDE.md / project.md / conventions in this repo yet.)
4. **Archive Consistency** — Skip with note: `openspec/specs/` is empty in this repo; this change establishes the baseline.
5. **Codegen Readiness** — Implicit requirements? Decisions left to codegen? Ambiguous units / types / formats? Could a fresh AI implement each spec requirement without inventing defaults?
6. **Gap Hunt (generative completeness probe)** — For each spec requirement, ask: are there unstated assumptions an implementer would have to invent? Error/edge paths specified? State transitions explicit?
7. **Drift Hunt** — Old vocabulary lingering? Recent renames in this exploration to be alert for:
   - `review-code` → `review-system`
   - `audit-residue` → `audit-drift`
   - `<!-- REVIEW: -->` → `@review:`
   - `openspec/system/` → `openspec/lenses/`
   - `openspec-review` → `openspec-orbit`
   - `/opsx:handoff` → `/opsx:review-external`
8. **Inline Review Marker Residue** — Any `@review:` markers still present in change-dir artifacts (excluding the orbit-conventions and orbit-address-reviews specs which document the marker syntax)?
9. **Pre-Handoff Sweep** — Small things easily missed on a first read.

**A specific concern**: this is the SECOND external review. The prior reviewer (codex) caught self-referential inconsistencies the internal pass missed. Look for what BOTH internal and external prior passes might have missed — common blind spots include things that "feel right because they were written down" but don't actually constrain implementation, or things that were renamed in some places but not others.

**Another specific concern**: compare your findings to codex's iteration-1 file. If you arrive at the same findings, that's signal those issues are real even if already "resolved" — was the resolution complete? If you find new things codex didn't, that's net-new value. If codex's findings now look misguided in retrospect, push back.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md`

Where `<TS>` is the current UTC timestamp in ISO format (e.g., `2026-05-18T03-30-00Z`). Pick a fresh timestamp; do not overwrite the iteration-1 file.

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 2)

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

<Optional: overall impression, broader concerns, what reads as well-designed, comparison to iteration-1 findings.>
```

If your environment doesn't support file writes (chat-only interface), output the markdown directly so the user can save it to the path above.

## After completing the review

1. **Also list findings in chat** — in addition to writing the file, output a concise summary in this chat so the user sees findings immediately without opening the file. Per finding: severity + title + `file:line`, optionally a one-line description.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 2): bootstrap-openspec-orbit

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

   **Note for this in-session run**: you are a subagent running inside the main authoring session. DO NOT commit or push — the main agent will handle that after you complete. Just write the findings file and report back.

If you don't have git access (or are following the in-session note above), the main agent will commit and push your file.
