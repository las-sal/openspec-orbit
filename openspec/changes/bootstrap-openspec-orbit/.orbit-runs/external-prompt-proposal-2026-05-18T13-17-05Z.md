# External Review: bootstrap-openspec-orbit (iteration 4)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning. The change has been through three prior external review passes (and resolutions); a major rename happened **between iter 3 and this pass**. Apply pushback discipline to the resolutions, and look specifically for residue introduced by the rename.

## Repo

`https://github.com/las-sal/openspec-orbit` (private; ask `las-sal` for access if needed, or work from a local checkout)

This repo IS the change. It dogfoods orbit on orbit itself — the change `bootstrap-openspec-orbit` proposes building orbit (the openspec overlay), and the proposal was authored using orbit's own (planned) workflow.

## ⚠ Major change since iter 3: review-proposal + review-system → unified review

Between iter 3 (resolved cleanly) and this pass, the authoring user decided to **collapse `/opsx:review-proposal` and `/opsx:review-system` into a single `/opsx:review <name> [--as proposal|system]` command** with mode inference from `tasks.md` state. Rationale: shared machinery dominates (scorecard, severity, flag family, pushback discipline); `--as` already used by `/opsx:review-external`; fewer top-level commands; conceptual unity. The merge collapsed:

- 2 spec files → 1 merged `specs/orbit-review/spec.md`
- 2 sketch files → 1 merged `sketches/review.md`
- 2 task groups → 1 merged group (Group 2)
- 9 capabilities → 8 capabilities

The rename touched **~150 references across 19 files**. A combination of sed-based mechanical replacement + hand-editing was used. Predictable residue surface: places sed didn't reach, places where mechanical replacement produced awkward phrasing.

**This rename is exactly the kind of change orbit's specs call out as high-residue ("renamed in some places, not others" blind-spot pattern). Look hard.**

## Project context (read first)

- `README.md` — full workflow + command reference + project orientation. Read this first.
- `openspec/config.yaml` — openspec configuration (mostly default)
- (no `CLAUDE.md`, no `openspec/project.md`, no `*_convention.md` files yet)
- (no `openspec/lenses/` content yet)
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history

## Cycle context

- **Iteration**: 4 (fourth external review for this change in proposal mode)
- **Prior internal findings still open**: 0 (resolved before iter 1)
- **Prior external findings still open**: 0
  - Iteration 1 (GPT-5 Codex, fresh): 7 findings, all resolved
  - Iteration 2 (Claude in-session subagent, fresh): 12 findings, 11 resolved + 1 suppressed
  - Iteration 3 (GPT-5 Codex, continued from iter 1): 4 findings, all resolved
- **Resolved since iter 3** (4 findings + the rename):
  - WARNING: proposal.md + sketch purpose still said "emits prompt to chat" → updated to file-backed contract
  - WARNING: Chat-output "exactly N items" wording vs. dirty-tree warning contradiction → reframed as optional/required items list
  - WARNING: README external-cycle walkthrough didn't reflect full-chat + commit/push contracts → updated steps 7-8
  - SUGGESTION: README workflow diagram still said "~2 decisions" → standardized to "2+"
  - **MAJOR RENAME** (post iter 3): collapsed review-proposal + review-system into unified `/opsx:review --as <mode>` command, merged specs/sketches/tasks accordingly

Do not push back on stale findings from prior passes — those were addressed. Just flag what *you* observe.

**Common blind-spot pattern across prior iterations**: "renamed in some places, not others." Each iteration found residue from prior iterations' resolutions in places those resolutions didn't reach. Iter 4 has a **much larger** rename to chase residue from.

## What to read for THIS review (`--as proposal` mode)

- `openspec/changes/bootstrap-openspec-orbit/proposal.md` — motivation, scope, **8 capabilities** (was 9 before merge)
- `openspec/changes/bootstrap-openspec-orbit/design.md` — context, goals, 12 design decisions, risks
- `openspec/changes/bootstrap-openspec-orbit/specs/<capability>/spec.md` — 8 capability spec files:
  - `orbit-review` (NEW — merged from orbit-review-proposal + orbit-review-system)
  - `orbit-review-external`, `orbit-audit-drift`, `orbit-address-reviews`, `orbit-explore-modifications`, `orbit-propose-modifications`, `orbit-archive-modifications`, `orbit-conventions`
- `openspec/changes/bootstrap-openspec-orbit/tasks.md` — task list (Group 3 was merged into Group 2; numbering jumps 2 → 4)
- `openspec/changes/bootstrap-openspec-orbit/explore.md` — historical exploration record
- `openspec/changes/bootstrap-openspec-orbit/sketches/*.md` — 7 sketches (review-proposal.md and review-system.md merged into review.md)
- `README.md` at repo root
- Prior iterations' findings + resolutions in `.orbit-runs/` for context

## What to look for

Apply the 9 review-proposal passes (note: this is "proposal-mode passes" now, applied to this proposal's artifacts):

1. **Structure & Delta Integrity** — All artifacts present? Delta sections valid? Does `openspec validate bootstrap-openspec-orbit` pass?
2. **Internal Coherence** — Proposal/design/specs/tasks aligned? Counts consistent (was 9 caps, now 8 — does everything reflect that)? Mode-aware behaviors described consistently?
3. **Cross-Doc Coherence** — README accurate? Sketches consistent with the merged spec?
4. **Archive Consistency** — Skip with note: `openspec/specs/` empty.
5. **Codegen Readiness** — Implicit requirements? Ambiguity? Could a fresh AI implement each requirement?
6. **Gap Hunt** — For each requirement, unstated assumptions? Error paths? State transitions?
7. **Drift Hunt** — Residue from the rename specifically:
   - References to `review-proposal` / `review-system` in current-state content (not historical "Considered & out" entries)
   - Awkward phrasings from sed-based mechanical replacement (parenthetical "(proposal mode)" / "(system mode)" inserts that read clumsily)
   - Argument-order issues: `/opsx:review --as proposal <name>` vs `/opsx:review <name> --as proposal` (the latter is the canonical form)
   - Old skill file names: `.claude/skills/openspec-review-proposal/`, `.claude/skills/openspec-review-system/` (should be `openspec-review/`)
   - Old command body file names: `.claude/commands/opsx/review-proposal.md`, `.claude/commands/opsx/review-system.md` (should be `review.md`)
   - Stale file-path references in tasks.md and elsewhere
   - File-naming pattern references — these CAN still mention `review-proposal-<TS>.json` and `review-system-<TS>.json` because those are the actual JSON summary file names (per the merged spec, `review-<mode>-<TS>.json` resolves to `review-proposal-<TS>.json` or `review-system-<TS>.json`); only flag if a JSON-name reference is being used as a command/capability name instead of a file name
8. **Inline Review Marker Residue** — Any `@review:` markers in change-dir artifacts that aren't documentation/examples?
9. **Pre-Handoff Sweep** — Small things missed on first read.

**Specific concerns for iter 4**:

- The merge into `/opsx:review` should mean exactly one capability spec (`orbit-review`); verify there are no leftover references to `orbit-review-proposal` or `orbit-review-system` as capability names in current-state content.
- The tasks.md merged Group 2 should be coherent (no overlapping responsibilities, no missed tasks); verify the 13 sub-tasks cover both modes without gaps.
- The merged sketch (`sketches/review.md`) should cover both modes; spot-check for missing content from the original two sketches.
- Inferred-mode messaging: the spec says when `--as` is omitted, the inferred mode is shown in the report header. Verify the spec scenario language is consistent (vs. e.g., `/opsx:review-external`'s "Generating external-review prompt as..." pattern).
- Numbering gap in tasks.md (Groups 2 → 4): cosmetic; flag as SUGGESTION if you notice.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md`

Where `<TS>` is the current UTC timestamp in ISO format. Pick a fresh timestamp; do NOT overwrite prior external-proposal files.

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 4)

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

<Optional: overall impression, broader concerns, comparison to prior iterations.>
```

If your environment doesn't support file writes, output the markdown directly so the user can save it.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section, every `### Title`, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 4): bootstrap-openspec-orbit

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown and the user will commit it manually.
