# External Review: bootstrap-openspec-orbit (iteration 5)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

**This is a fresh-context review — you have NO prior session memory from earlier iterations.** Four prior review passes happened (one internal + three external); their findings are documented in `.orbit-runs/`. The cycle context below summarizes what's open and what's been resolved. The point of fresh-context iteration 5 is to verify the change reads cleanly to an AI that has no anchoring on prior findings — does the design hold up cold, or are there issues that anchored reviewers wouldn't catch?

## Repo

Local checkout at `/Users/sal/code/openspec-review/` (this is the openspec-orbit dev sandbox; the repo is `https://github.com/las-sal/openspec-orbit`).

This repo IS the change. It dogfoods orbit on orbit itself — the change `bootstrap-openspec-orbit` proposes building orbit (the openspec overlay), and the proposal was authored using orbit's own (planned) workflow. Be alert for self-referential inconsistencies — places where the meta-design described in specs doesn't match what's actually done in `explore.md`, sketches, or the change layout itself.

## Project context (read first)

- `README.md` — full workflow + command reference + project orientation. Read this first.
- `openspec/config.yaml` — openspec configuration (mostly default)
- (no `CLAUDE.md` yet, no `openspec/project.md` yet, no `*_convention.md` files yet — this change establishes the orbit overlay before adopting it elsewhere)
- (no `openspec/lenses/` content yet)
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history; 4 prior review/resolution cycles documented here

## Cycle context

- **Iteration**: 5 (fifth external review for this change in proposal mode)
- **Prior internal findings still open**: 0 (12 surfaced and all resolved)
- **Prior external findings still open**: 0
  - Iteration 1 (GPT-5 Codex, fresh): 7 findings, all resolved
  - Iteration 2 (Claude in-session subagent, fresh-context): 12 findings, 11 resolved + 1 suppressed
  - Iteration 3 (GPT-5 Codex, continued chat from iter 1): 4 findings, all resolved
  - Iteration 4 (GPT-5 Codex, continued chat — post-rename): 6 findings, all resolved
- **Major design changes since iter 4** (these are the targets of THIS iteration):
  - **Rename**: `/opsx:review-proposal` + `/opsx:review-system` collapsed into unified `/opsx:review <name> [--as proposal|system]` with mode inference from `tasks.md` state. 2 capability specs → 1 merged spec. 2 sketches → 1 merged sketch. Two task groups → 1 (numbering gap 2 → 4 preserved with explanatory note). 8 capabilities total (was 9).
  - **Change completeness discipline codified**: new requirement in `orbit-conventions` (5 scenarios) requiring any substantive modification to be applied fully across all affected artifacts before declared done. Origin: a real-time failure during this dogfood where mechanical sed replacement left known residue. Lesson now in the spec.
  - **Read-before-reference discipline codified**: new requirement in `orbit-conventions` (8 scenarios) requiring the AI to read the actual definition of any specific named construct before generating code/tests/specs that reference it. Origin: real-world home-env failure where the AI assumed an object's structure based on common patterns. Lesson now in the spec.
- **Discipline framing** (now three cross-cutting disciplines bracketing the authoring lifecycle):
  - Authoring-time: read-before-reference
  - Modification-time: change completeness
  - Review-time: pushback

Do not push back on stale findings from prior passes — they're addressed (or explicitly suppressed). Apply your own independent judgment.

## What to read for THIS review (`--as proposal` mode)

- `openspec/changes/bootstrap-openspec-orbit/proposal.md` — motivation, scope, **8 capabilities** (was 9 before merge)
- `openspec/changes/bootstrap-openspec-orbit/design.md` — context, goals, 12 design decisions, risks
- `openspec/changes/bootstrap-openspec-orbit/specs/<capability>/spec.md` — 8 capability spec files:
  - `orbit-review` (unified, contains the merged proposal-mode + system-mode behavior)
  - `orbit-review-external`, `orbit-audit-drift`, `orbit-address-reviews`, `orbit-explore-modifications`, `orbit-propose-modifications`, `orbit-archive-modifications`, `orbit-conventions` (now contains the three cross-cutting disciplines)
- `openspec/changes/bootstrap-openspec-orbit/tasks.md` — task list (Group 2 merged, numbering gap 2 → 4 with explanatory note)
- `openspec/changes/bootstrap-openspec-orbit/explore.md` — historical exploration record (Decisions section captures all major decisions including the recent discipline additions and the rename)
- `openspec/changes/bootstrap-openspec-orbit/sketches/*.md` — 7 sketches (review-proposal.md + review-system.md merged into review.md)
- `README.md` at repo root — adopter-facing documentation
- Prior iterations' findings + resolutions in `.orbit-runs/` for context only (don't re-litigate)

## What to look for

Apply the 9 review-proposal passes:

1. **Structure & Delta Integrity** — All artifacts present? Delta sections valid? Does `openspec validate bootstrap-openspec-orbit` pass?
2. **Internal Coherence** — Proposal/design/specs/tasks aligned? Capability count consistent at 8? The three disciplines mentioned consistently (in conventions spec, explore.md decisions, README cross-cutting section, README CLAUDE.md snippet)? Mode-aware behavior described consistently for `/opsx:review`?
3. **Cross-Doc Coherence** — README accurate? Sketches consistent with merged spec? CLAUDE.md snippet matches the actual spec requirements?
4. **Archive Consistency** — Skip with note: `openspec/specs/` empty.
5. **Codegen Readiness** — Implicit requirements? Ambiguity? Could a fresh AI implement each requirement?
6. **Gap Hunt** — For each requirement, unstated assumptions? Error paths? State transitions? Does the new merged `orbit-review` spec genuinely cover both modes without gaps?
7. **Drift Hunt** — Old vocabulary lingering anywhere current-state? Common targets to check:
   - `review-proposal` / `review-system` (current-state, not historical) — should be gone from active content
   - File paths to deleted sketches/spec dirs
   - "5 new commands" (should be 4 after merge)
   - Awkward sed-residue parentheticals like "review (system mode)"
   - Argument-order issues: `/opsx:review --as proposal <name>` vs `/opsx:review <name> --as proposal`
8. **Inline Review Marker Residue** — Any `@review:` markers in change-dir artifacts that aren't documentation/examples?
9. **Pre-Handoff Sweep** — Small things missed on first read.

**Specific concerns for iter 5**:

- The merged `orbit-review` spec is the largest single spec; verify it doesn't have internal contradictions between mode-specific scenarios.
- The two NEW discipline requirements added to `orbit-conventions` should be coherent with each other and with the existing pushback discipline. Check that the three disciplines don't overlap incorrectly or contradict.
- The README CLAUDE.md snippet describes the three disciplines for adopters. Verify the snippet's description of each discipline matches the corresponding spec requirement (no drift between adopter-facing summary and normative spec).
- The "lesson origin" scenarios in the two new disciplines cite specific historical events (the rename dogfood for change-completeness; the home-env test-case incident for read-before-reference). Verify these references are accurate to their respective sources (`.orbit-runs/external-proposal-2026-05-18T13-46-41Z.md` etc. for the rename; home-env is the user's external context).
- tasks.md has a numbering gap (Group 2 → Group 4) with an explanatory note inserted. Verify the note is clear and located correctly.

**Apply the read-before-reference discipline yourself during review**: when you reference a specific construct in your findings (a file:line, a requirement name, a capability name), make sure you've read it. Don't infer.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-<TS>.md`

Where `<TS>` is the current UTC timestamp in ISO format. Pick a fresh timestamp; do NOT overwrite prior external-proposal files.

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 5)

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

If your environment doesn't support file writes (chat-only interface), output the markdown directly so the user can save it.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in your response. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title`, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize — the chat output is the immediately-visible read.

2. **DO NOT commit or push** — you are an in-session subagent running inside the authoring AI's session. The authoring AI will handle git operations after you complete. Just write the findings file and report back.

3. If you find no issues, say so explicitly with a clean assessment — that's a valid and informative outcome for an iter-5 fresh-context review.
