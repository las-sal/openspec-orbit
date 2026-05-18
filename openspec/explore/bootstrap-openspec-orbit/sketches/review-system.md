# Sketch: `/opsx:review-system`

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (coherence with openspec form and spirit) — adopts `verify-change`'s reporting convention verbatim. Composes with upstream `verify-change` rather than replacing it.

## Purpose

Editorial review of **whole-product state** after a change has been applied. Wraps upstream `verify-change` and adds 6 system-wide passes that ask "is the whole product still honest after this change?" — not just "did we build what this change said we'd build."

**Generates findings. Does not resolve them.** Resolution flows through `/opsx:address-reviews` or by user replying directly to the report.

Post-apply, pre-archive gate. Sister command of `/opsx:review-proposal` (pre-apply, artifact-side) and the natural superset of upstream `/opsx:verify` (post-apply, structural-only).

## Why it exists (over just running verify-change)

`verify-change` is rigorous within scope — but its scope is **the change's own deltas**. It never opens `openspec/specs/` (archived baseline), never walks callers outside the task list, never enumerates surfaces, never validates from caller perspectives, never scans critical paths. Every one of these gaps maps to a recurring user prompt from ~80 transcripts:

| review-system pass | grounded in transcript pattern |
|---|---|
| Pass 1 — Baseline Compliance | "make sure the rest of the system still works" / "review for any drift" |
| Pass 2 — Cohesion | "go down every code path, look at every API" |
| Pass 3 — Surface Walk | "review every CLI command, review every MCP command" |
| Pass 4 — Perspective Reviews | "review from claude desktop's POV" / "from the swift host's POV" |
| Pass 5 — Critical-Path Scan | "pay special attention to critical and highly used code paths. Focus on user impacting issues" |
| Pass 6 — Drift / Residue | "make sure no old nomenclature has escaped" (lesson from OPENSPEC_LESSONS.md) |

## Inputs

- `<change-name>` (optional) — if omitted, prompt via `AskUserQuestion` from `openspec list --json`. Note: scoped *by* a change name, but examines the whole system.
- **Depth modes (mutually exclusive, pick one):**
  - `--fast` — cheap subset only: Pass 0 (verify-change) + Pass 1 (Baseline Compliance) + Pass 6 (Drift / Residue). Skips heavy passes 2/3/4/5. For mid-cycle quick sanity checks. Combine with `--skip-verify` for the absolutely-fastest subset (Passes 1 + 6 only).
  - `--full` (**default**) — all passes 0–6, sequential. The workhorse mode per guiding principle 2.
  - `--thorough` — all passes + extras (perspectives walked twice with different framing, critical paths re-walked with adversarial lens, baseline-compliance second-pass on broadest scope). Specifics tracked in GitHub issue. For pre-`/opsx:archive` deep clean.
- **Execution mode (orthogonal, combinable with any depth):**
  - `--parallel` — spawn subagents for heavy passes (2/3/4/5) concurrently. ~3-4× wall-clock; partitions context for larger projects. The real win is context partitioning, not just speed — without it, on a project the size of home-control, a single context can't hold the whole codebase + baseline specs + change diff. Opt-in for v1; possibly default in v2.
- **Secondary flags:**
  - `--fresh` — main work runs in a clean-context subagent (different from `--parallel`; about avoiding conversation framing, not parallelism). Opt-in for v1.
  - `--focus <lens>` — `rename` / `flip` / `refactor` / `hotpath` (see "Special-case lenses"). Additive emphasis on named passes; doesn't skip others.
  - `--skip-verify` — assume `/opsx:verify-change` already ran cleanly; skip Pass 0 and run system-wide passes only.
  - `--strict` — fail-fast on first CRITICAL (default: run all passes, present unified report). For CI-style usage.

## What it reads

Use openspec CLI as source of truth where possible.

- **Change artifacts**: `openspec/changes/<name>/{proposal,design,tasks}.md` + `specs/*/spec.md` + `explore.md`
- **Change diff**: what code did this change actually touch? (via `git diff` against the change's start point, or via the tasks list cross-referenced with file paths)
- **Baseline specs**: `openspec/specs/*/spec.md` — full archived baseline. Critical for Pass 1.
- **Codebase**: the parts not touched by the change but interconnected with what was. Critical for Pass 2.
- **Project lenses (required for full rigor)**: `openspec/lenses/perspectives.md`, `openspec/lenses/critical-paths.md`. Surfaces derived from `openspec/specs/<capability>/` (capabilities are surfaces). Without lenses content, Passes 4/5 skip with a note; Pass 3 still runs against derived surfaces.
- **Project context**: `CLAUDE.md`, `openspec/project.md`, `*_convention.md`. Used by Pass 6.

## Passes

`Pass 0` is `verify-change`. Passes 1–6 are orbit additions. Findings use the same CRITICAL / WARNING / SUGGESTION severity, file:line refs, actionable recommendations, false-positive bias.

### Pass 0 — verify-change (delegate)

Run upstream `verify-change` first. Capture its full report verbatim. Its findings feed the unified scorecard. Skipped if `--skip-verify`.

### Pass 1 — Baseline Compliance

Wider correctness check than verify-change.

- Read every requirement in `openspec/specs/*/spec.md` (not just the change's deltas).
- For each requirement, mentally walk the change's code and ask: *does this still hold?*
- Look for behaviors the change may have broken without realizing.
- Special attention to requirements whose surface names match anything the change touched.

Output: WARNING when a baseline scenario looks regressed; CRITICAL when a baseline requirement appears violated.

### Pass 2 — Cohesion (caller / dependent walk)

Surfaces side-effects beyond the change's intended scope.

- Identify files the change touched (from `git diff` and the tasks list).
- Find callers / dependents not in the change's tasks.
- For each: signature drift? Semantic shifts? Side effects? Dangling assumptions?
- Edge cases: imports / re-exports / dynamic dispatch / config-driven wiring.

Output: WARNING for caller-side type or contract drift; CRITICAL when a downstream is clearly broken.

### Pass 3 — Surface Walk

Enumerates every external surface the project exposes and checks each remains coherent.

- Derive surfaces from `openspec/specs/<capability>/spec.md` — each capability is a surface (CLI, MCP server, HTTP endpoints, etc.).
- For each surface entry: does it still work coherently after this change? Naming consistent? Help text accurate? Error responses sensible? Documented behavior matches code?
- If a surface entry was *removed* by the change, was that intended? (Cross-check against proposal.md.)
- If a surface entry was *added* by the change, was it specified? (Cross-check against spec deltas.)

Output: WARNING when a surface entry is broken or inconsistent; SUGGESTION for minor naming/help-text drift.

### Pass 4 — Perspective Reviews

Validates from caller-side rather than implementation-side.

- Read perspectives from `openspec/lenses/perspectives.md` (e.g., "Claude Desktop using MCP", "Swift host calling Python service").
- For each perspective: simulate typical call patterns. Does the calling surface feel coherent from that side? Are required call sequences reasonable? Are errors actionable from the caller's POV?
- Flags interactions that look awkward, surprising, or inconsistent from a registered caller's perspective.

Output: SUGGESTION/WARNING when something doesn't make sense from a caller's POV.

### Pass 5 — Critical-Path Scan

End-to-end walks of named user flows. Bug / race / regression hunting concentrates here.

- Read critical paths from `openspec/lenses/critical-paths.md` (e.g., "user opens claude desktop and asks about device state", "user adjusts thermostat via swift host").
- For each: walk the code end-to-end. Does the path still exist? Does it still produce the same observable behavior? Any new race windows, error swallowing, latency cliffs, or state inconsistencies?
- `--focus hotpath` raises rigor on this pass.

Output: CRITICAL for path breakage; WARNING for regression risk or correctness gaps.

### Pass 6 — Drift / Residue (cross-doc)

Calls into `/opsx:audit-drift` and folds findings.

- Grep for old vocabulary across `openspec/specs/`, `openspec/project.md`, `CLAUDE.md`, `*_convention.md`.
- Patterns drawn from the change's proposal/design (renames, removed binaries, restructured terms).
- Direct lesson from OPENSPEC_LESSONS.md — `sync-specs` only touches delta'd files; non-delta'd docs go stale and a fresh implementer follows the stale text.

Output: WARNING for stale references in active docs; SUGGESTION for archived/historical references that may be legitimate.

## Special-case lenses (`--focus`)

| lens | shifts emphasis to | because |
|---|---|---|
| `rename` | Pass 6 (drift), Pass 3 (surface re-walk for renamed entries) | renames leak old names into non-delta'd docs and surface entries |
| `flip` | Pass 2 (cohesion on direction), Pass 4 (perspectives from BOTH ends) | comms ownership must be unambiguous post-flip |
| `refactor` | Pass 2 (cohesion), Pass 6 (drift) | refactors ripple into callers and docs; old shape lingers |
| `hotpath` | Pass 5 (critical-path scan) | landed work touched performance/race-sensitive paths |

Lenses are additive; they don't skip passes.

## Reporting

Adopts `verify-change`'s convention verbatim. The 7 passes (Pass 0 from verify-change + 6 system-wide) roll up into 3 dimensions for the summary scorecard.

### Pass → dimension roll-up

| Dimension | Passes |
|---|---|
| **Completeness** | Pass 0 (verify-change completeness) + Pass 5 (critical paths end-to-end existence) |
| **Correctness** | Pass 0 (verify-change correctness) + Pass 1 (baseline compliance) + Pass 2 (cohesion) + Pass 5 (critical paths working) |
| **Coherence** | Pass 0 (verify-change coherence) + Pass 3 (surface walk) + Pass 4 (perspectives) + Pass 6 (drift/residue) |

Pass 5 spans Completeness and Correctness — "does the path exist?" and "does it work?" are separate questions but the same scan answers both.

### Output template

```markdown
## Review Report: <change-name> (system)

### Summary
| Dimension    | Status                                                |
|--------------|-------------------------------------------------------|
| Completeness | tasks 12/12, 3 critical paths walked, 0 gaps          |
| Correctness  | 2 baseline regressions, 1 cohesion drift              |
| Coherence    | 4 surfaces scanned, 1 perspective issue, 2 doc-stale  |

### CRITICAL (1)
1. specs/device-selection/spec.md:127 — baseline requirement
   `backend.list_rooms()` no longer satisfied; this change
   removed `HomeKitBackend` but didn't update the baseline
   spec or the contract.
   → Add a delta to this change updating device-selection,
     or revert the rename in this PR.

### WARNING (4)
…

### SUGGESTION (6)
…

### Final Assessment
1 critical issue, 4 warnings. Fix before `/opsx:archive`.
Suggested next: address baseline regression, then re-run
/opsx:review-system --focus refactor.
```

### Final-assessment phrasings

| State | Phrasing |
|---|---|
| ≥1 CRITICAL | `X critical issue(s) found. Fix before \`/opsx:archive\`.` |
| Only WARNING / SUGGESTION | `No critical issues. Y warning(s) to consider. Ready to archive (with noted improvements).` |
| All clear | `All checks passed. Ready to archive.` |

(Same shape as verify-change and review-proposal; only the gate differs.)

## Heuristics & graceful degradation

- **Lower severity when uncertain** — same as verify-change.
- **Every finding actionable** — file:line + specific recommendation.
- **Degrade gracefully**:
  - Empty / absent `openspec/lenses/` → skip Passes 4/5 with an explicit note. Pass 3 still runs (surfaces derived from `openspec/specs/`). Passes 1/2/6 unaffected.
  - Empty `openspec/specs/` (new project) → Pass 1 trivially passes.
  - No archived changes → Pass 6 has less to grep for; still runs against current docs.
- **Pushback hook** — if a finding contradicts current state (e.g., flag is "stale ref to X" but X was removed in a commit between when the reviewer ran and now), self-correct and note "stale finding suppressed" (lesson from OPENSPEC_LESSONS.md on parallel-reviewer findings).
- **Don't short-circuit by default** — run all passes even if early ones report CRITICAL, so the user gets one unified report. `--strict` opts into fail-fast.

## Open design questions

1. **Cost of system-wide passes**. Pass 2 (cohesion), Pass 4 (perspectives), Pass 5 (critical-path) are each substantial work. Options: run sequentially (slow but predictable); parallel via subagents (faster, more context burn); cache per-(change, ref) so a re-run after a fix only re-walks changed surfaces.
2. **`--mark` for review-system (source-code markers)** — review-proposal's `--mark` writes markers into change-dir markdown artifacts. review-system could analogously write `// @review:` / `# @review:` markers into source files based on its findings. Defer until address-reviews v1 proves the model in pure-scan mode; add in v2 alongside other comprehensive features (issue #3).
3. **`audit-drift` composition** (resolved 2026-05-17). Pass 6 calls `/opsx:audit-drift` as a library function; users can also invoke it standalone. Same logic, two entry points.
4. **Bug/race detection prominence**. Pass 5 catches user-flow regressions; should there be a dedicated bug/race heuristic pass, or stays folded into Pass 5? Lean: stays folded; if signal-to-noise on it is bad, split later.
5. **Whole-tree mode without a change name**. Sometimes the user wants "review the whole product, not tied to a change." That's closer to `/opsx:distill-specs` or `/opsx:audit-drift` running standalone — not review-system. Keep review-system change-scoped (matches verify-change's contract).
6. **Test coverage as a check**. verify-change's Correctness dimension does "scenario coverage." Does review-system's Pass 5 explicitly check for tests on critical paths? Lean: yes, fold into Pass 5 reporting.

## Composition with related commands

```
…
apply ──── /opsx:apply
            │
            ▼
review ─── /opsx:review-system [<name>] [--focus …] [--skip-verify] [--strict] [--fresh]
            │
            ├── Pass 0:  /opsx:verify-change (delegate) ─── upstream
            │
            ├── Pass 1–5: orbit system-wide passes
            │
            ├── Pass 6:  /opsx:audit-drift (library call) ─── orbit
            │
            └── unified scorecard + severity-grouped findings
                    │
                    ▼
resolve ── /opsx:address-reviews
            │
            ├── consumes @review: markers
            ├── consumes external-feedback pastes (--from-paste)
            └── pushback discipline; removes markers on resolution
                    │
                    └─── cycle (review → address → review …) until clean
                            │
                            ▼
archive ── /opsx:archive
```

## Parallels with `review-proposal`

By design, the two commands are structural twins:

| | `review-proposal` | `review-system` |
|---|---|---|
| Stage | pre-apply | post-apply |
| Gate | `/opsx:apply` | `/opsx:archive` |
| Reads from | proposal/design/specs/tasks + baseline | codebase + baseline + change diff |
| Wraps | nothing upstream (proposal stage has no upstream equivalent) | upstream `verify-change` |
| Differentiator pass | Pass 6 (gap hunt — generative completeness probe) | Passes 3/4/5 (surfaces / perspectives / critical paths — system-wide coherence) |
| Needs `openspec/lenses/` | optional (light reliance) | required for full rigor (Passes 4/5; Pass 3 derives surfaces from specs) |
| Marker convention | `@review:` (any file type — markdown bare; source/configs inside file-type comments) | same |
| Final gate phrasing | "Fix before /opsx:apply" | "Fix before /opsx:archive" |

Adopters who learn one command's shape can predict the other's.
