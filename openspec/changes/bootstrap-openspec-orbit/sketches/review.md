# Sketch: `/opsx:review <name> [--as proposal|system]`

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17, merged from `sketches/review-proposal.md` + `sketches/review-system.md` on 2026-05-18 when the two commands collapsed into one with `--as` mode.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — adopts `verify-change`'s reporting convention verbatim). System mode composes with upstream `verify-change` rather than replacing it.

## Purpose

Single editorial review command for an OpenSpec change. Two modes:

- **`--as proposal`** — pre-apply review of artifacts (proposal, design, spec deltas, tasks, explore.md). 9 passes. Codifies the recurring spec-side review-shape from ~80 transcripts.
- **`--as system`** — post-apply review of the whole product state. Wraps upstream `verify-change` as Pass 0; adds 6 system-wide passes that examine baseline compliance, cohesion, surfaces, perspectives, critical paths, and drift.

**Generates findings. Does not resolve them.** Resolution flows through `/opsx:address-reviews` (markers, paste, finding-files) or by user replying directly to the report.

Both modes share: scorecard (Completeness/Correctness/Coherence), severity ladder (CRITICAL/WARNING/SUGGESTION), flag family, pushback discipline, persistence to `.orbit-runs/`. They differ on: which passes run, what gate the final assessment references (`/opsx:apply` vs `/opsx:archive`), and a few mode-specific flags (`--mark` in proposal, `--skip-verify` in system).

## Why this is one command, not two

The original design had two separate commands (`/opsx:review-proposal` and `/opsx:review-system`). They were merged because:

- Shared machinery dominates: same scorecard, same severity, same flag family, same output convention, same pushback discipline, same iteration-tracking.
- The differences (which passes run, which gate to reference) are mode-specific behavior — exactly what a `--as` flag is for.
- `/opsx:review-external --as proposal|system` already uses this pattern. Two commands ↔ one command with `--as` is consistent.
- Fewer top-level commands to remember. Mode is contextual (pre-apply vs post-apply), not a separate verb.

The merge happened after 4 review cycles converged cleanly on the v1 design; the rename was a polish step that came from sleeping on the design.

## Inputs

- `<change-name>` (optional) — if omitted, prompt via `AskUserQuestion` from `openspec list --json`.
- **Mode flag** `--as proposal|system` — if omitted, inferred from `tasks.md` state (unchecked → proposal; all checked + code → system; ambiguous → user prompted).
- **Depth modes (mutually exclusive):**
  - `--fast` — cheap subset only.
    - Proposal mode: Passes 1 (Structure & Delta), 7 (Drift Hunt), 8 (Inline Review Marker Residue).
    - System mode: Pass 0 (verify-change) + Pass 1 (Baseline) + Pass 6 (Drift/Residue). Combine with `--skip-verify` for Passes 1 + 6 only.
  - `--full` (**default**) — all mode-specific passes, sequential. The workhorse.
  - `--thorough` — all passes + extras (tracked in issue #2).
- **Execution mode:**
  - `--parallel` — spawn subagents for heavy passes. Proposal mode parallelizes Passes 2, 4, 6. System mode parallelizes Passes 2, 3, 4, 5.
- **Secondary flags:**
  - `--focus <lens>` — emphasize one of: `rename` / `flip` / `refactor` / `extension` (proposal-mode lenses) / `hotpath` (system-mode lens).
  - `--strict` — fail-fast on first CRITICAL.
  - `--fresh` — clean-context subagent for main work; opt-in.
  - `--mark` (proposal mode only) — drop `@review:` markers in artifacts based on findings.
  - `--skip-verify` (system mode only) — skip Pass 0 if `/opsx:verify-change` ran separately.

## What it reads

Use openspec CLI as source of truth (`openspec list --json`, `openspec status --change <name> --json`, `openspec instructions apply --change <name> --json`).

Common to both modes:
- Change artifacts: `openspec/changes/<name>/{proposal,design,tasks}.md` + `specs/*/spec.md` + `explore.md`
- Project context: `CLAUDE.md`, `openspec/project.md`, `*_convention.md`
- Baseline: `openspec/specs/*/spec.md`
- Orbit config: `openspec/lenses/perspectives.md`, `openspec/lenses/critical-paths.md`

System mode also reads:
- Change diff (via `git diff` against the change's start point)
- Codebase paths around the change (for cohesion + surface walks)

## Passes

### Proposal mode (9 passes)

```
PASS                                  WHAT IT CHECKS
─────────────────────────────────────────────────────────────────────
1. Structure & Delta Integrity        artifacts present; delta sections valid;
                                       openspec validate passes
2. Internal Coherence                 proposal/design/specs/tasks align;
                                       counts consistent; no scope creep
3. Cross-Doc Coherence                CLAUDE.md / project.md / conventions
                                       still accurate after this change
4. Archive Consistency                ADDED don't contradict baseline; RENAMED
                                       FROM exists; REMOVED not referenced
5. Codegen Readiness                  no implicit requirements; no ambiguity;
                                       no decisions left to codegen
6. Gap Hunt (generative completeness) could a fresh AI implement this from
                                       these specs alone? unstated assumptions?
                                       error paths? state transitions?
7. Drift Hunt                         old vocabulary lingering; consistency
                                       with *_convention.md
8. Inline Review Marker Residue       any @review: markers still present?
                                       (CRITICAL — must address before apply)
9. Pre-Handoff Sweep                  small things missed on first read
```

### System mode (7 passes)

```
PASS                                  WHAT IT CHECKS
─────────────────────────────────────────────────────────────────────
0. verify-change (delegated upstream) tasks done; spec coverage; design
                                       adhered (full verify-change findings)
1. Baseline Compliance                does this change break archived
                                       baseline requirements?
2. Cohesion                           callers/dependents outside the tasks —
                                       signature drift, ripple effects
3. Surface Walk                       every CLI/MCP/HTTP surface still
                                       coherent? (surfaces derived from specs)
4. Perspective Reviews                from each registered caller-perspective
                                       in lenses/perspectives.md
5. Critical-Path Scan                 each flow in lenses/critical-paths.md,
                                       walked end-to-end
6. Drift / Residue                    calls /opsx:audit-drift as a library
```

## Special-case lenses

`--focus <lens>` adds emphasis to specific passes (additive — doesn't skip others):

| lens | mode | passes emphasized | why |
|---|---|---|---|
| `rename` | proposal | 7 (drift), 3 (cross-doc), 4 (archive) | renames leak old names everywhere |
| `flip` | proposal | 2 (internal coherence on direction), 5 (codegen readiness) | comms ownership must be unambiguous |
| `refactor` | both | proposal: 4, 7, 3; system: 2, 6 | old shape ripples into callers/docs |
| `extension` | proposal | 6 (gap hunt) | adding capability — coverage must match existing depth |
| `hotpath` | system | 5 (critical-path scan) | landed work touched performance/race-sensitive paths |

## Reporting

Both modes use the same 3-dimension scorecard, severity ladder, file:line refs, and final-assessment shape — only the gate text differs.

### Roll-up

**Proposal mode**:
- Completeness: Passes 1, 5, 6
- Correctness: Passes 2, 4
- Coherence: Passes 3, 7, 8, 9

**System mode**:
- Completeness: Pass 0 (verify-change completeness portion) + Pass 5 (critical paths existence)
- Correctness: Pass 0 (verify-change correctness) + Pass 1 (Baseline) + Pass 2 (Cohesion) + Pass 5 (critical paths working)
- Coherence: Pass 0 (verify-change coherence) + Pass 3 (Surface) + Pass 4 (Perspectives) + Pass 6 (Drift/Residue)

### Final-assessment phrasings

| Mode | State | Phrasing |
|---|---|---|
| proposal | ≥1 CRITICAL | `X critical issue(s) found. Fix before \`/opsx:apply\`.` |
| proposal | Only WARNING/SUGGESTION | `No critical issues. Y warning(s) to consider. Ready to apply (with noted improvements).` |
| proposal | All clear | `All checks passed. Ready to apply.` |
| system | ≥1 CRITICAL | `X critical issue(s) found. Fix before \`/opsx:archive\`.` |
| system | Only WARNING/SUGGESTION | `No critical issues. Y warning(s) to consider. Ready to archive (with noted improvements).` |
| system | All clear | `All checks passed. Ready to archive.` |

## Heuristics & graceful degradation

- **Bias toward lower severity** when uncertain.
- **Every finding actionable** — file:line + specific recommendation.
- **Pushback discipline** — verify each finding against current state before reporting. Stale findings get suppressed with an evidence note.
- **Graceful degradation**:
  - Proposal mode + empty `openspec/specs/` → Pass 4 (Archive Consistency) skipped with note.
  - Proposal mode + no `*_convention.md` → Pass 3 lens-driven checks skip.
  - System mode + empty `openspec/lenses/` → Passes 4/5 skip; Pass 3 still runs against derived surfaces.

## Persistence + iteration tracking

Each run writes a JSON summary to `openspec/changes/<name>/.orbit-runs/review-<mode>-<TS>.json`. Iteration count is per-mode (proposal-mode and system-mode tracked separately).

Iteration note appears in the report when prior `review-<mode>-<TS>.json` files exist for the same mode: `Note: N of these findings appeared in the last run on <date>. M new this run.`

## Open design questions

1. **`--fresh` default** — opt-in for v1; revisit as default in v2 if data supports.
2. **Cost of system-mode passes** — heavy. `--parallel` helps. Caching deferred to v2 (issue #1).
3. **`--mark` for system mode** — writing markers into source code (`// @review:` / `# @review:`) is v2 (issue #3).
4. **`--thorough` specifics** — tracked in issue #2.

## Composition with related commands

```
/opsx:propose <name>
        │
        ▼
/opsx:review <name>             ← infers --as proposal from unchecked tasks
        │
        ▼
findings + 3-dim scorecard
        │
        ▼
/opsx:review-external <name>    ← package external review (--as proposal)
        │
        ▼
external AI reads prompt + writes findings file
        │
        ▼
/opsx:address-reviews --from-file <path>
        │
        ▼  (cycle until clean)
        │
/opsx:apply <name>
        │
        ▼
/opsx:review <name>             ← infers --as system from all-tasks-checked + code
        │
        ▼
findings + 3-dim scorecard (verify-change Pass 0 + 6 system-wide passes)
        │
        ▼
/opsx:review-external <name>    ← package external review (--as system)
        │
        ▼  (cycle until clean)
        │
/opsx:archive <name>            ← auto-invokes audit-drift pre-sweep
```

## What this means for the SKILL.md modifications

One SKILL.md at `.claude/skills/openspec-review/SKILL.md` and one slash command body at `.claude/commands/opsx/review.md`. The skill prompt branches on `--as` mode (or inferred mode). Mode-specific pass implementations + reporting gates live inside the same prompt, organized by mode-specific sections.

A single skill file is appropriate because the shared behavior (scorecard, severity, flag handling, pushback, persistence) is the bulk of the prompt; mode-specific content sits as sections inside it.
