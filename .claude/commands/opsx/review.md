---
name: "OPSX: Review"
description: Editorial review of an OpenSpec change in proposal mode (pre-apply) or system mode (post-apply)
category: Workflow
tags: [workflow, review, orbit]
---
Run an editorial review of an OpenSpec change. Two modes share one command:

- **`--as proposal`** — pre-apply review of artifacts (proposal, design, spec deltas, tasks, explore.md). 9 passes.
- **`--as system`** — post-apply review of the whole product state. Wraps upstream `verify-change` as Pass 0 + adds 6 system-wide passes.

When `--as` is omitted, the mode is inferred from `tasks.md` state (unchecked → proposal; all checked + code → system; ambiguous → prompts).

**Generates findings; does not resolve them.** Use `/opsx:address-reviews` to close findings (inline `@review:` markers, paste, or `--from-file`).

## Input

`/opsx:review [<change-name>] [--as proposal|system] [flags]`

- `<change-name>` — optional. If omitted, prompts via `openspec list --json`.
- `--as proposal|system` — optional. Inferred from `tasks.md` if omitted.

## Flags

```
--fast | --full | --thorough     depth (default --full)
--parallel                       subagent parallelism for heavy passes
--focus <lens>                   rename | flip | refactor | extension | hotpath
--strict                         fail-fast on first CRITICAL
--fresh                          clean-context subagent for main work
--mark                           proposal mode only: drop @review: markers based on findings
--skip-verify                    system mode only: skip Pass 0 (verify-change)
```

## What it does

Invokes the `openspec-review` skill, which:

1. Resolves change name, mode, and flags (mode inference from `tasks.md` state when `--as` omitted)
2. Reads change artifacts via the openspec CLI (`openspec status`, `openspec instructions apply`)
3. Loads project context (`CLAUDE.md`, `project.md`, `*_convention.md`), baseline specs (`openspec/specs/`), orbit lenses (`openspec/lenses/`)
4. Checks `.orbit-runs/` for prior `review-<mode>-*.json` summaries — informs iteration tracking and stale-finding suppression
5. Runs mode-specific passes:
   - **Proposal**: Passes 1–9 (Structure & Delta, Internal Coherence, Cross-Doc, Archive Consistency, Codegen Readiness, Gap Hunt, Drift Hunt, Inline Marker Residue, Pre-Handoff Sweep)
   - **System**: Pass 0 (delegates to `/opsx:verify` via the upstream `openspec-verify-change` skill) + Passes 1–6 (Baseline Compliance, Cohesion, Surface Walk, Perspective Reviews, Critical-Path Scan, Drift/Residue via `/opsx:audit-drift`)
6. Rolls findings into the 3-dimension scorecard (Completeness / Correctness / Coherence)
7. Emits the final-assessment line (mode-specific gate text: `/opsx:apply` vs `/opsx:archive`)
8. Persists a run summary to `openspec/changes/<change-name>/.orbit-runs/review-<mode>-<TS>.json`

## Final assessment (mode + iteration-aware)

The final-assessment line is one of a fixed set of stock phrasings. Proposal mode uses 3 phrasings (CRITICAL / Only W/S / all clear). System mode uses 1 CRITICAL phrasing + 10 "no CRITICAL" phrasings, selected by an iteration-aware logic that resolves one of **5 convergence states** based on `.orbit-runs/` artifacts. The system-mode no-CRITICAL default (State 1, no prior external) nudges toward external review before archive — empirical basis: in-context system review missed **3 of 3** real implementation bugs in `bootstrap-orbit-status-cli`'s first archived cycle (all caught by external GPT-5 Codex review).

### Proposal mode (3 phrasings)

| State | Phrasing |
|---|---|
| ≥1 CRITICAL | `X critical issue(s) found. Fix before /opsx:apply.` |
| Only WARNING/SUGGESTION | `No critical issues. Y warning(s) to consider. Ready to apply (with noted improvements).` |
| All clear | `All checks passed. Ready to apply.` |

### System mode — ≥1 CRITICAL

| State | Phrasing |
|---|---|
| ≥1 CRITICAL | `X critical issue(s) found. Fix before /opsx:archive.` |

### System mode — no CRITICAL (5 convergence states × 2 sub-cases)

- **State 1 — no prior external system review** (`external-system-*.md` absent): recommend `/opsx:review-external` (or `/opsx:review --fresh` as lighter alternative) before archive; cite the bootstrap-orbit-status-cli 3-of-3 empirical evidence in the prose.
- **State 2 — Path A convergence (external content is clean)**: `External system review (clean) converged. Ready to archive.`
- **State 3 — Path B convergence (external findings resolved via address-reviews)**: `External system review findings resolved via /opsx:address-reviews. Ready to archive.`
- **State 4 — external present with unresolved findings**: recommend `/opsx:address-reviews --from-file <external-path>` to walk them before archive.
- **State 5 — external stale relative to artifact changes** (takes precedence over States 2–4): recommend re-running `/opsx:review-external` (or `/opsx:review --fresh`).

Each state has both an "all clear" and an "Only WARNING/SUGGESTION" sub-case — 10 stock phrasings total. See `.claude/skills/openspec-review/SKILL.md` Step 9 for the exact wordings.

### Iteration-aware logic

To select the convergence state, inspect `openspec/changes/<name>/.orbit-runs/` and resolve in this order:

1. **Presence** — no `external-system-*.md` → State 1.
2. **Stale (highest precedence)** — compare the external's filename `<TS>` token against the most recent `apply-*.json` token (per orbit-conventions `Internal-run JSON summary format`; lexical sort works because tokens are `YYYY-MM-DDTHH-MM-SSZ`). If apply is newer → State 5. Per-external scoping: only `apply-*.json` triggers stale, not unrelated `address-reviews-*.json`.
3. **Path A (content clean)** — each `## CRITICAL`/`## WARNING`/`## SUGGESTION` section in the external markdown contains only the empty-severity sentinel (`None.` or accepted equivalents `None`, `none.`, `(none)` per `openspec-address-reviews/references/external-findings-format.md`) with zero `### <title>` entries. If clean → State 2.
4. **Path B (resolution)** — most recent `address-reviews-*.json` whose `source_path` (canonicalized to repo-relative) references this external file has `resolution_summary.deferred == 0` AND `resolution_summary.escalated == 0` → State 3. Per-external scoping: address-reviews for OTHER inputs do NOT affect this external's convergence.
5. **Unresolved** — external has findings AND no Path B resolution → State 4.

Edge cases (multiple matching files, unparseable tokens, parse failures, dangling source_path) follow the v1 assumptions documented in the spec's `Edge-case assumptions for the iteration-aware logic` scenario; the SKILL.md Step 9 prose reproduces them.

**The recommendation is advisory, not a gate**: orbit surfaces the suggestion but the user retains the choice to skip.

## Output

Standard 3-dimension scorecard report with CRITICAL / WARNING / SUGGESTION severities, file:line refs, and actionable recommendations. Mode and iteration are shown in the header. Final-assessment line uses one of the stock phrasings above (see SKILL.md for the full table and a worked system-mode example).

**Disjunctive recommendations**: when a finding's recommendation requires the user to choose between two or more concrete alternatives (e.g., "Either file a follow-up issue, or extend scope to tasks.md"), the JSON emit includes an optional `recommendation_options: [{label, body}]` field on the finding entry alongside the prose `recommendation`. The structured field is consumed by `/opsx:address-reviews` for the decision-fork prompt UX; the prose `recommendation` still summarizes the disjunction. Producer-side contract: ≥ 2 entries; non-empty `label` and `body`; omit for single-recommendation findings. See SKILL.md's "Disjunctive recommendations" section + `references/run-summary-schema.md` for the full shape + emit rules.

## Execution disciplines

Three disciplines apply throughout (per `orbit-conventions`):

- **Read-before-reference (authoring-time)** — read the actual definition of any specific construct cited in a finding.
- **Change completeness (modification-time)** — `--mark` writes must be applied fully across all affected artifacts, with a sweep for residue after mechanical insertion.
- **Pushback (review-time)** — verify each finding against current state before reporting. Stale findings get suppressed with evidence.

See `.claude/skills/openspec-review/SKILL.md` for full behavior, scorecard rollup, and worked examples.
