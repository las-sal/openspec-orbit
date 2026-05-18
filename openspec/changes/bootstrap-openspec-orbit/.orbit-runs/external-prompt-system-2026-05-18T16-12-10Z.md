# External Review: bootstrap-openspec-orbit (iteration 1, system mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is
your independent take — be thorough; flag anything that looks wrong,
inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

This is a **system-mode** review (post-apply). The change has been applied:
new orbit commands and modifications to upstream skills exist as markdown
artifacts under `.claude/`. Review the whole product state, not just the
change artifacts.

## Repo

https://github.com/las-sal/openspec-orbit

(Pull `main`; latest commit applies the change as a single landed patch.)

## Project context (read first)

- `CLAUDE.md` — handoff orientation; reinforces orbit's three execution disciplines at the project level
- `README.md` — full user-facing surface; command reference, workflow walkthrough, repo layout, install instructions
- `openspec/changes/bootstrap-openspec-orbit/.orbit-runs/` — iteration history; see what's already been addressed in prior cycles
- Note: this repo currently has no `openspec/project.md`, no `*_convention.md` at project root, and no `openspec/lenses/` content — this is the bootstrap change establishing the orbit overlay itself, so lenses/conventions are downstream of orbit shipping

## Cycle context

- Iteration: 1 (first external review in **system mode**; sets independent baseline)
- Prior **proposal-mode** review cycles ran 5 external iterations + 6 internal address-reviews runs and converged cleanly (all proposal-mode findings resolved before `/opsx:apply` ran)
- The change was applied in a single commit (`1e62e02`) on 2026-05-18 immediately before this review
- Internal **system-mode** review iter 1 ran just before this prompt and produced 1 WARNING + 1 SUGGESTION (both in `README.md`, both about my own install-instruction wording residue); the resolution to those fixes is in the working tree but **may or may not be committed** by the time you pull (see "Important commit-state note" below)
- No prior `external-system-*.md` files exist; you are the first external system-mode reviewer

### Important commit-state note

The repo has uncommitted README fixes at the time this prompt is written. The user is expected to commit those fixes + this prompt file before pushing. If they have, you'll see clean `README.md` install instructions (lines 890 + 912 area). If they haven't, you'll see two clear bugs in that area — those are the iter-1 internal-review findings, not new ones; flag them only if they're still present in the committed state you pull.

Do not push back on stale findings — pushback discipline is enforced on
resolution, not review. Just flag what you observe.

## What to read for THIS review (`--as system`)

```
- openspec/changes/bootstrap-openspec-orbit/proposal.md, design.md, tasks.md, specs/
- The change's commit range: git log --oneline 28b9c05..HEAD
  (28b9c05 is the initial commit; everything since is the orbit work)
  Specifically: git diff 28b9c05..HEAD -- .claude/ CLAUDE.md README.md
- openspec/changes/bootstrap-openspec-orbit/.orbit-runs/ (prior iteration history)
- openspec/specs/ — note: EMPTY (no archived baseline; this IS the bootstrap)
- openspec/lenses/ — note: ABSENT (no perspectives or critical paths captured yet)
- Code paths "touched by the change" = the .claude/ markdown overlay; orbit ships prompts, not executable code
```

### Special note on this change's shape

Orbit ships **markdown prompts** (SKILL.md files + slash command bodies), not executable code. The "code" being reviewed is:

- `.claude/skills/openspec-{review,review-external,audit-drift,address-reviews}/` — four new orbit skill directories (each with a `SKILL.md` and a `references/` subdir for emitted artifacts / JSON schemas)
- `.claude/skills/openspec-{explore,propose,archive-change}/SKILL.md` — three upstream skills with orbit additions layered on (the upstream content is preserved verbatim; orbit additions are in an "Orbit additions" section near the bottom of each file)
- `.claude/commands/opsx/{review,review-external,audit-drift,address-reviews}.md` — four new slash command bodies
- `.claude/commands/opsx/{explore,propose,archive}.md` — three modified slash command bodies (with "Orbit additions (summary)" sections appended)
- `CLAUDE.md` (new at project root)
- `README.md` (updated with lens entry shapes, JSON schema section, current repo layout, full install instructions)

For Pass 2 (Cohesion), the "callers/dependents" are cross-references between skills: e.g., orbit-review's system mode invokes audit-drift; archive auto-invokes audit-drift; address-reviews `--from-file` parses the format that review-external writes. These prompt-to-prompt contracts are what cohesion-walking checks for in this change.

## What to look for

Run these 7 passes:

### Pass 0 — Verify-change-style structural check

- Are all 84 of the 102 tasks legitimately completed? Spot-check a few against actual file existence.
- Are the 18 incomplete tasks all annotated as user-driven deferred work? Is the deferral justified, or are some hiding real gaps?
- Spec coverage: do all 8 capability specs map to implementations?
  - `orbit-review` → `.claude/skills/openspec-review/SKILL.md`
  - `orbit-review-external` → `.claude/skills/openspec-review-external/SKILL.md`
  - `orbit-audit-drift` → `.claude/skills/openspec-audit-drift/SKILL.md`
  - `orbit-address-reviews` → `.claude/skills/openspec-address-reviews/SKILL.md`
  - `orbit-explore-modifications` → orbit-additions section in `.claude/skills/openspec-explore/SKILL.md`
  - `orbit-propose-modifications` → orbit-additions section in `.claude/skills/openspec-propose/SKILL.md`
  - `orbit-archive-modifications` → orbit-additions section in `.claude/skills/openspec-archive-change/SKILL.md`
  - `orbit-conventions` → distributed: README, CLAUDE.md, embedded in each new SKILL.md, references/ schemas
- Design adherence: do the implementations honor design.md decisions D1–D13?
- For each spec requirement, can you find the corresponding implementation behavior in the SKILL.md?

### Pass 1 — Baseline Compliance

Skip with note: no `openspec/specs/` baseline exists yet (this IS the bootstrap). Document the skip; flag if there's anything in the change that pretends a baseline exists.

### Pass 2 — Cohesion

The "callers/dependents" for this change are cross-references between skills. Check:

- orbit-review system mode says it invokes `/opsx:audit-drift` in Pass 6 → does the audit-drift skill exist and behave consistently with what review expects?
- orbit-review system mode says Pass 0 delegates to upstream `openspec-verify-change` → does the contract make sense (e.g., does review fold verify-change's findings into the scorecard correctly per the mapping table)?
- orbit-archive says it auto-invokes audit-drift pre-archive → does audit-drift have the `pre-archive` invocation context the archive SKILL relies on?
- orbit-review `--mark` writes `@review:` markers → does address-reviews recognize and resolve them?
- orbit-review-external's "Output format" block specifies the parseable findings format → does address-reviews `--from-file` actually parse that format? (Both sides must agree.)
- The references/ files (prompt-template.md, mode-sections.md, run-summary-schemas) are emitted artifacts and parser contracts — are they consistent with the SKILL.md prose that references them?

Flag any contract drift, missing reciprocals, or signature mismatches between these prompt-to-prompt boundaries.

### Pass 3 — Surface Walk

Capabilities ARE surfaces. For each of the 8 capabilities, check whether the slash-command surface (flags, arguments, invocation forms) is coherent and complete:

- `/opsx:review` flags: `--as proposal|system`, `--fast/--full/--thorough`, `--parallel`, `--focus <lens>`, `--strict`, `--fresh`, `--mark` (proposal only), `--skip-verify` (system only). Are these reflected in: (a) the spec, (b) SKILL.md, (c) command body, (d) README command-reference section? Any drift?
- `/opsx:review-external` flags: `--as proposal|system`. Consistent everywhere?
- `/opsx:audit-drift` flags: `--fast/--full/--thorough`, `--parallel`, `--focus <area>`, `--since <ref>`, `--strict`. Consistent?
- `/opsx:address-reviews` flags: `--only <pattern>`, `--from-file <path>`, `--keep-resolved-markers`. Consistent?
- `/opsx:archive` flags: `--skip-audit` (orbit addition). Consistent across upstream-preserved content + orbit-additions section?

Final-assessment phrasings (stock forms): there are 6 mode/state combinations for orbit-review and a separate set for orbit-audit-drift (standalone/library/pre-archive contexts). Are the exact strings consistent across SKILL.md, command body, and README?

### Pass 4 — Perspective Reviews

Skip with note: no `openspec/lenses/perspectives.md` exists. (Lens content is downstream of orbit shipping.) Document the skip.

### Pass 5 — Critical-Path Scan

Skip with note: no `openspec/lenses/critical-paths.md` exists. Document the skip.

### Pass 6 — Drift / Residue

Run the equivalent of `/opsx:audit-drift`:

- **Category 1 (Vocabulary residue)**: No archived deltas exist yet (no `openspec/changes/archive/`). Degenerate; document and move on.
- **Category 2 (Lens staleness)**: No lenses exist. Degenerate; document and move on.
- **Category 3 (Cross-doc consistency)** — the main one. Check:
  - `CLAUDE.md` describes orbit's behavior. Does that description match what the SKILL.md files actually do?
  - `README.md` describes orbit's commands and conventions. Does it match the SKILL.md content?
  - The command-reference section of README lists flags/passes/scorecard. Does this match the new SKILL.md surfaces 1:1?
  - The repo-layout section of README claims certain files exist. Do they?
  - Cross-check counts: how many capabilities does each artifact claim? How many passes? How many new commands? Are these consistent across `proposal.md` / `design.md` / `tasks.md` / `README.md` / `CLAUDE.md`?
- **Category 4 (Archive coherence)**: No archived changes. Degenerate; document and move on.

### Additional probes worth running

These are non-standard probes worth doing because the change is unusual (prompts-only, bootstrap-itself):

- **Self-application sanity**: does the change's spec coherently describe a system that includes the change's own implementation? (E.g., does orbit-conventions require what the change actually shipped?)
- **References/ split**: each new orbit SKILL.md has a `references/` subdir holding the emitted artifacts and JSON schemas. Are the pointers from SKILL.md to references/ accurate (e.g., does `Read references/run-summary-schema.md` in the prose match the actual filename)?
- **Three execution disciplines (read-before-reference / change completeness / pushback)**: these are required to be embedded in each new orbit SKILL.md per design D13. Verify presence and consistency of phrasing across the 4 new SKILLs. Also verify the discipline statements in CLAUDE.md match.
- **Modified upstream SKILL.md files** (`openspec-explore`, `openspec-propose`, `openspec-archive-change`): orbit's design says upstream content is preserved verbatim and orbit additions are purely additive (in an "Orbit additions" section). Verify upstream content is genuinely unchanged — no accidental edits.
- **Deferred tasks (18 of 102)**: are they all genuinely user-driven, or are some hiding real gaps? Spot-check 2.13, 11.x, 12.3-12.5.

## Output format — write to:

`openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format with hyphens, e.g., `2026-05-18T17-30-00Z`. Pick a fresh timestamp so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: bootstrap-openspec-orbit (iteration 1, system mode)

**Reviewer**: <your model name>
**Date**: 2026-05-18

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

## WARNING
...

## SUGGESTION
...

## Notes

<Optional: overall impression, broader concerns, what you couldn't check, etc.>
```

If your environment doesn't support file writes (chat-only interface), output
the markdown directly and the user will save it.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the
   findings file, output the COMPLETE findings markdown in this chat. Same
   content as the file: every severity section (`## CRITICAL` / `## WARNING` /
   `## SUGGESTION`), every `### Title` entry, every `**File**:` and
   `**Description**:` field. Do NOT abbreviate or summarize — the chat output
   is the immediately-visible read for the user (they should be able to
   evaluate every finding without opening the file). The file remains the
   canonical record for `--from-file` parsing.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-system-<TS>.md
   git commit -m "External review (system, iter 1): bootstrap-openspec-orbit

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat
(per the chat-only fallback above) and the user will commit it manually.
