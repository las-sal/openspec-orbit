# Sketch: `/opsx:review-external`

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — sits in the `review-*` family), principle 2 (cost up front — external review surfaces issues the authoring AI misses).

## Purpose

Packages a review request for an external AI (codex, fresh Claude, GPT, etc.) to run a second-opinion review. Generates a ready-to-paste prompt and a defined output format so the external AI's findings come back as a parseable file — no copy-paste-per-issue.

Sister command to `/opsx:review-proposal` and `/opsx:review-system`:
- `review-proposal` = internal review of proposal artifacts (pre-apply)
- `review-system` = internal review of whole system (post-apply)
- **`review-external`** = generate the packaging for an external AI to run a review pass

The external AI's findings flow back through `/opsx:address-reviews --from-file <path>` for resolution.

## Why this exists

From your transcripts: ~5 review cycles per change, each involving a cross-AI handoff. Without orbit, that means:

- Manually constructing a prompt each time ("read these files, check X, Y, Z")
- Manually copying findings back, finding by finding
- No iteration history visible to the external reviewer
- Different external reviewers get different framings depending on what you type

`review-external` makes the handoff durable: same prompt structure every time, iteration context auto-populated, findings written to a defined path that orbit can ingest.

## Inputs

- `<change-name>` (optional) — if omitted, prompt via `AskUserQuestion` from `openspec list --json`.
- **Mode flag (default inferred from change state):**
  - `--as proposal` — package a proposal-review (pre-apply context, focus on artifacts).
  - `--as system` — package a system-review (post-apply context, focus on code + cohesion).
  - No flag → infer from `tasks.md`:
    - Unchecked boxes → `proposal`
    - All checked + code exists → `system`
    - Ambiguous → prompt user via `AskUserQuestion`
    - Inferred mode shown in output: "Generating external-review prompt as `proposal` (inferred from tasks state)."

No depth flags, no `--parallel` — this command emits text and exits.

## What it produces

Output is **two things**:

1. **A versioned prompt file written into the repo**:

   ```
   openspec/changes/<change-name>/.orbit-runs/external-prompt-<as>-<TS>.md
   ```

   This is the full handoff prompt as a committed file. Committing it (a) gives
   the external AI a fixed URL to read instead of receiving a paste, (b) creates
   an audit trail of exactly what prompt was given for each external pass, (c)
   keeps the chat-side surface tiny.

2. **A short invocation snippet to chat** (typically 3-5 lines): the prompt
   file path, a 1-3 sentence copy-paste-ready instruction for the external AI
   ("Pull <repo URL> and read <prompt-file-path>; follow its instructions"),
   and the eventual findings path for the user to pass to
   `/opsx:address-reviews --from-file` once findings come back.

The user pushes the prompt file (so the external AI can read it from GitHub),
pastes the short invocation snippet into the external AI, it pulls the repo
and reads the prompt file, then writes findings to:

```
openspec/changes/<change-name>/.orbit-runs/external-<as>-<TS>.md
```

Where `<as>` is `proposal` or `system`, and the findings file's timestamp will
typically be later than the prompt file's timestamp (the external AI's
wall-clock when it finishes). Prompt and findings files pair implicitly by
chronology and mode.

If the external AI has no file-write capability (pure chat interface), the
prompt file instructs it to output the findings markdown so the user can save
to the path manually.

## The prompt content (mode-specific)

The prompt file's content is a self-contained markdown block the external AI reads after pulling the repo. Two mode-specific variants share a common skeleton.

### Common skeleton (both modes)

````markdown
# External Review: <change-name> (iteration <N>)

You are reviewing an OpenSpec change as a second pair of eyes. Your value
is your independent take — be thorough; flag anything that looks wrong,
inconsistent, or unclear. Don't be charitable to the authoring AI's
reasoning.

## Repo

<repo URL or path>

## Project context (read first)

- `CLAUDE.md` — handoff orientation
- `openspec/project.md` — project goals + stack
- `*_convention.md` at repo root — naming, error handling, etc.
- `openspec/lenses/perspectives.md` — named callers worth validating from
- `openspec/lenses/critical-paths.md` — user flows worth walking end-to-end
- `openspec/changes/<change-name>/.orbit-runs/` — iteration history;
  see what's already been addressed in prior cycles

## Cycle context

- Iteration: <N>
- Prior internal findings open: <count + brief list>
- Prior external findings open: <count + brief list>
- Resolved since last review: <brief list>

Do not push back on stale findings — pushback discipline is enforced on
resolution, not review. Just flag what you observe.

## Output format

Write your findings to:

`openspec/changes/<change-name>/.orbit-runs/external-<as>-<TS>.md`

(Where <TS> is today's timestamp in ISO format. Pick a fresh timestamp
so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: <change-name> (iteration <N>)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

### <Next finding title>
...

## WARNING

### ...

## SUGGESTION

### ...

## Notes

<Optional: anything that doesn't fit the severity ladder.>
```

If your environment doesn't support file writes (chat-only interface),
output the markdown directly and the user will save it.

## After completing the review

1. **Also list findings in chat** — in addition to writing the file, output
   a concise summary in this chat so the user sees findings immediately
   without opening the file. Per finding: severity + title + `file:line`,
   optionally a one-line description.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/<change-name>/.orbit-runs/external-<as>-<TS>.md
   git commit -m "External review (<as>, iter <N>): <change-name>

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this
chat (per the chat-only fallback above) and the user will commit it
manually.

## ──────────────────────────────────────────────────────────────────
[Mode-specific section appended below]
````

### `--as proposal` appendix

````markdown
## What to read for THIS review (proposal mode)

- `openspec/changes/<change-name>/proposal.md` — motivation, scope
- `openspec/changes/<change-name>/design.md` — decisions, trade-offs
- `openspec/changes/<change-name>/specs/<capability>/spec.md` — delta specs
- `openspec/changes/<change-name>/tasks.md` — implementation tasks
- `openspec/changes/<change-name>/explore.md` — historical record of
  the exploration that led to this proposal
- `openspec/specs/` — current archived baseline (for consistency checks)

## What to look for

1. **Structure & delta integrity** — all artifacts present? Delta sections
   valid (ADDED/MODIFIED/REMOVED/RENAMED)? Task back-references where
   useful?
2. **Internal coherence** — proposal motivation aligns with design
   decisions; design decisions align with spec requirements; spec
   requirements covered by tasks. No scope creep.
3. **Cross-doc coherence** — CLAUDE.md / project.md / conventions still
   accurate after this change lands? Updates needed?
4. **Archive consistency** — new ADDED requirements don't contradict
   existing archived requirements. RENAMED FROM symbols exist in
   baseline. REMOVED requirements aren't still referenced.
5. **Codegen readiness** — no implicit requirements; no decisions
   left to codegen; no ambiguous units / types / ranges.
6. **Gap hunt (generative completeness)** — given JUST these artifacts,
   could a fresh AI implement the system without inventing requirements?
   What questions would arise?
7. **Drift hunt** — old vocabulary still present in any artifacts?
   Special focus if this is a rename / refactor change.
8. **Inline review marker residue** — any `@review:` markers still
   present in the change-dir artifacts? They must be addressed before
   apply (excluding markers in orbit-conventions or orbit-address-
   reviews specs that document the marker syntax itself).
9. **Pre-handoff sweep** — small things easily missed on a first read.
````

### `--as system` appendix

````markdown
## What to read for THIS review (system mode)

- `openspec/changes/<change-name>/` — same artifacts as proposal mode
- The codebase — what does this change actually touch?
  - Use `git diff` against the change's start point or inspect tasks.
- `openspec/specs/` — full archived baseline (Pass 1 below relies on this)
- `openspec/lenses/perspectives.md` and `critical-paths.md` — your guides
  for Passes 3 and 4 below

## What to look for

1. **Spec conformance for THIS change** — does the implementation match
   the change's delta specs and tasks? (Same as upstream `verify-change`
   would check.)
2. **Baseline compliance** — does this change break any *archived*
   requirements in `openspec/specs/`? (Wider scope than just deltas.)
3. **Cohesion** — find callers/dependents NOT in the change's tasks.
   Any signature drift, semantic shifts, side effects rippling beyond
   the change's intended scope?
4. **Surface walk** — for each capability that defines a public surface
   (CLI / MCP / HTTP / etc.), does each entry still behave coherently?
5. **Perspective reviews** — for each named perspective in
   `openspec/lenses/perspectives.md`, simulate typical call patterns.
   Awkward, inconsistent, or surprising interactions?
6. **Critical-path scan** — for each flow in
   `openspec/lenses/critical-paths.md`, walk end-to-end. Breakage?
   Regression? Drift? Race conditions on the user-facing path?
7. **Drift / residue** — old vocabulary lingering in non-delta'd specs,
   project.md, CLAUDE.md, conventions, lenses?
````

## Iteration counting

Tracked separately for proposal-mode and system-mode reviews:

```
openspec/changes/<name>/.orbit-runs/
├── review-proposal-<TS>.json     ← internal proposal run #1, #2, ...
├── review-system-<TS>.json       ← internal system run #1, #2, ...
├── external-proposal-<TS>.md     ← external proposal review #1, #2, ...
└── external-system-<TS>.md       ← external system review #1, #2, ...
```

When `review-external` runs, it counts the existing files matching the mode pattern and reports iteration N = (matching files + 1).

## What `review-external` does NOT do

- **Doesn't run the review** — it just packages it. The external AI runs the review.
- **Doesn't ingest findings** — that's `/opsx:address-reviews --from-file`.
- **Doesn't auto-trigger** — user invokes when ready.
- **Doesn't validate the external AI's output** — assumes good faith. If output is malformed, `address-reviews` will report parse errors.

## Heuristics & graceful degradation

- **Always show the inferred mode** in the chat output so the user can correct.
- **If `<change-name>` is provided but doesn't exist**, halt with a clear error.
- **If `.orbit-runs/` is empty**, iteration is 1; cycle context says "first review."
- **If repo has uncommitted changes**, warn — external review will be against committed state.

## Open design questions

1. **What if the user wants to externalize an audit-drift run?** Skipped for v1 — audit-drift is project-wide, not change-scoped, and your transcripts don't show external-audit patterns. Easy to add later: `/opsx:review-external --as audit-drift` (no `<change-name>`).
2. **Should the prompt include a diff?** For `--as system`, the prompt could embed a `git diff` snippet instead of asking the external AI to compute it. Pro: less work for external. Con: longer prompt, may exceed context for large diffs. Lean: don't embed; tell external to compute. Reconsider if external AIs report friction.
3. **Multiple external reviewers in one cycle** — if you ask codex AND fresh Claude in the same iteration, both write to `.orbit-runs/external-<as>-<TS>.md` with different timestamps. address-reviews can ingest each separately. No special handling needed; the iteration counter just keeps incrementing.

## Composition

```
/opsx:review-proposal <change>          (internal)
        │
        ▼
findings in chat + Final Assessment
        │
        ▼
/opsx:review-external <change>          (package for external)
        │  (--as inferred or explicit)
        ▼
prompt file written to .orbit-runs/external-prompt-<as>-<TS>.md
+ tiny invocation snippet emitted to chat
        │
        ▼
user pushes prompt file; pastes invocation snippet into codex
        │
        ▼
codex pulls repo, reads prompt file, reads change + context + lenses
codex writes findings to .orbit-runs/external-<as>-<TS>.md
codex lists findings in chat + commits + pushes
        │
        ▼
/opsx:address-reviews --from-file <path>
        │
        ▼
resolution log; markers walked with pushback discipline
        │
        ▼
(cycle: re-run review-proposal to confirm clean,
 or re-run review-external for another external pass)
```

The same shape applies for the system-side cycle (`/opsx:review-system` → `/opsx:review-external --as system` → `/opsx:address-reviews --from-file ...`).
