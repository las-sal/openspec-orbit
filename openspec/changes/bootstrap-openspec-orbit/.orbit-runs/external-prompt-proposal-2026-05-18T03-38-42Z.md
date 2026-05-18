# External Review: bootstrap-openspec-orbit (iteration 3)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning. The authoring AI has now completed three resolution passes (one internal + two external) and resolved 30+ findings. Apply pushback discipline to *those* outcomes: re-examine whether the resolutions actually closed the issues or just renamed them, and look for the residue each resolution left behind.

## Repo

`https://github.com/las-sal/openspec-orbit` (private; ask `las-sal` for access if needed, or work from a local checkout)

This repo IS the change. It dogfoods orbit on orbit itself — the change `bootstrap-openspec-orbit` proposes building orbit (the openspec overlay), and the proposal was authored using orbit's own (planned) workflow. Be alert for self-referential inconsistencies — places where the meta-design described in specs doesn't match what's actually done in `explore.md`, sketches, or the change layout itself.

## Project context (read first)

- `README.md` — full workflow + command reference + project orientation. Read this first to understand what orbit is.
- `openspec/config.yaml` — openspec configuration (mostly default in this repo)
- (no `CLAUDE.md` yet, no `openspec/project.md` yet, no `*_convention.md` files yet — this change establishes the orbit overlay before adopting it elsewhere)
- (no `openspec/lenses/` content yet — empty in this repo)
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history; review the prior internal and external passes

## Cycle context

- **Iteration**: 3 (third external review for this change in proposal mode)
- **Prior internal findings still open**: 0 (12 surfaced and all resolved — see `address-reviews-2026-05-18T02-15-00Z.json`)
- **Prior external findings still open**: 0
  - Iteration 1 (GPT-5 Codex): 7 findings (6 W + 1 S), all resolved — see `address-reviews-2026-05-18T03-13-46Z.json`
  - Iteration 2 (Claude in-session subagent, clean context): 12 findings (7 W + 5 S), 11 resolved + 1 suppressed with pushback — see `address-reviews-2026-05-18T03-31-17Z.json`
- **Resolved since iteration 2** (the in-session subagent pass):
  - WARNING: design.md path still pointed to pre-propose openspec/explore/ → updated to openspec/changes/
  - WARNING: review-external sketch composition diagram still showed chat-paste handoff → updated to file-write + invocation snippet flow
  - WARNING: review-proposal sketch + review-system sketch diagrams showed --from-paste → corrected to --from-file (the v1 path)
  - WARNING: orbit-review-proposal Pass 8 not operationally defined → added scenario distinguishing actual unresolved markers from documentation appearances
  - WARNING: explore.md had 3 current-state /opsx:audit-residue references → updated to /opsx:audit-drift
  - SUGGESTION: Mode-C threshold drift across artifacts → standardized on "2+ substantive decisions"
  - SUGGESTION: orbit-review-system Pass 0 roll-up referenced "verify-change's own mapping" without pinning → pinned inline
  - SUGGESTION: task 6.10 didn't make --from-file v1 status explicit → tagged "(lean v1 inclusion)"
  - SUGGESTION: 1 finding suppressed (S3) with pushback evidence — conflated proposal.md content with local-checkout directory name
- **Subsequent design pivot**: chat output requirement tightened from "concise summary" to "full findings markdown identical to file content" — your prompt template reflects this. The current iteration's output instruction (in the "After completing the review" section below) is the NEW contract; the older "concise summary" wording in prior `.orbit-runs/external-prompt-*` files is historical.

Do not push back on stale findings from prior passes — those were addressed (or explicitly suppressed). Just flag what *you* observe.

**Common blind-spot pattern across prior iterations**: "renamed in some places, not others." Each fix introduced new residue elsewhere. Iter 1 (codex) fixed README paths but missed design.md. Iter 2 (subagent) fixed design.md path but missed the dispersion to sketch composition diagrams. Iter 3, look specifically for: residue of iter-2 resolutions in places those resolutions didn't touch (other diagrams, other prose mentions, README sections, task descriptions).

## What to read for THIS review (`--as proposal` mode)

- `openspec/changes/bootstrap-openspec-orbit/proposal.md` — motivation, scope, capabilities list (9 capabilities)
- `openspec/changes/bootstrap-openspec-orbit/design.md` — context, goals, 12 design decisions, risks, open questions
- `openspec/changes/bootstrap-openspec-orbit/specs/<capability>/spec.md` — 9 capability spec files:
  - `orbit-review-proposal`, `orbit-review-system`, `orbit-review-external`, `orbit-audit-drift`, `orbit-address-reviews`, `orbit-explore-modifications`, `orbit-propose-modifications`, `orbit-archive-modifications`, `orbit-conventions`
- `openspec/changes/bootstrap-openspec-orbit/tasks.md` — 105 implementation tasks across 12 groups
- `openspec/changes/bootstrap-openspec-orbit/explore.md` — historical record of the exploration
- `openspec/changes/bootstrap-openspec-orbit/sketches/*.md` — 8 detailed sketches per command
- `README.md` at repo root — adopter-facing documentation; should align with specs
- (optional) Prior iterations' findings:
  - `external-proposal-2026-05-18T03-00-12Z.md` — codex iter 1 (7 findings)
  - `external-proposal-2026-05-18T05-15-00Z.md` — subagent iter 2 (12 findings)

## What to look for

Apply the 9 review-proposal passes:

1. **Structure & Delta Integrity** — All artifacts present? Delta sections valid? Task back-references where useful? Does `openspec validate bootstrap-openspec-orbit` pass?
2. **Internal Coherence** — Proposal/design/specs/tasks aligned? No scope creep? Numbers/counts consistent?
3. **Cross-Doc Coherence** — README.md still accurate after recent updates? (No CLAUDE.md / project.md / conventions in this repo yet.)
4. **Archive Consistency** — Skip with note: `openspec/specs/` is empty (this change establishes the baseline).
5. **Codegen Readiness** — Implicit requirements? Decisions left to codegen? Ambiguous units/types/formats? Could a fresh AI implement each spec requirement without inventing defaults?
6. **Gap Hunt (generative completeness)** — For each spec requirement: unstated assumptions? Error paths specified? State transitions explicit? Two-implementer agreement?
7. **Drift Hunt** — Old vocabulary lingering? Recent renames to watch for:
   - `review-code` → `review-system`
   - `audit-residue` → `audit-drift`
   - `<!-- REVIEW: -->` → `@review:`
   - `openspec/system/` → `openspec/lenses/`
   - `openspec-review` → `openspec-orbit`
   - `/opsx:handoff` → `/opsx:review-external`
   - **NEW since iter 2**: "concise summary" → "full findings markdown" in chat (for review-external's after-completion instruction)
   - **NEW since iter 2**: "~2", "~2-3", "2 or more" → "2+ substantive decisions" for Mode-C crystallization threshold
8. **Inline Review Marker Residue** — Any `@review:` markers still present in change-dir artifacts (excluding orbit-conventions and orbit-address-reviews specs which document the marker syntax)? Per the new Pass 8 operational scenario in `specs/orbit-review-proposal/spec.md`: distinguish unresolved markers (CRITICAL) from documentation appearances inside code blocks (NOT findings).
9. **Pre-Handoff Sweep** — Small things easily missed on a first read.

**Specific concerns for iter 3**:

- The "concise summary → full findings" pivot (codified just before this iteration) might have residue: places that still use "summary" wording, or older `.orbit-runs/external-prompt-*.md` files (historical, expected; only flag if active content references them as canonical).
- The Pass 8 operational scenario was added — check that the requirement and Pass 8 are now consistent (Pass 8 listed in the requirement text + Pass 8 has its own operational scenario).
- The design.md path fix used parenthetical note about `openspec/explore/<name>/` being pre-propose staging. Cross-check that the parenthetical is accurate (it should describe orbit-propose-modifications' staging-to-changes move).
- Three of the four `.orbit-runs/external-prompt-*.md` files are from THIS iteration cycle (iter 1, iter 2, iter 3). They're not active spec content — they're historical handoff artifacts. Only flag if you see them treated as canonical references.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md`

Where `<TS>` is the current UTC timestamp in ISO format (e.g., `2026-05-18T04-15-00Z`). Pick a fresh timestamp; do NOT overwrite the iteration-1 or iteration-2 files.

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 3)

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

<Optional: overall impression, broader concerns, what reads as well-designed, comparison to prior iterations.>
```

If your environment doesn't support file writes (chat-only interface), output the markdown directly so the user can save it to the path above.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize — the chat output is the immediately-visible read for the user (they should be able to evaluate every finding without opening the file). The file remains the canonical record for `--from-file` parsing.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 3): bootstrap-openspec-orbit

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
