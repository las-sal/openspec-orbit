# Sketch: `/opsx:audit-drift`

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — adopts `verify-change`'s reporting convention), principle 2 (cost up front — runs comprehensive scan by default), principle 3 (specs as source of truth — keeps spec/doc references coherent).

## Purpose

Project-wide scan for **drift in captured knowledge vs. reality**. Catches the failure modes that `sync-specs` doesn't (non-delta'd files going stale after framework-vocabulary changes), keeps `openspec/lenses/` honest, and surfaces cross-doc inconsistencies before they mislead the next implementer.

Single unified command — `audit-drift` covers all drift-shaped concerns. No `audit-residue` / `audit-lenses` / `audit-coherence` family.

Three invocation paths:
- **Library call**: `/opsx:review-system` Pass 6 calls `audit-drift` internally.
- **Standalone**: user invokes when "something feels off."
- **Auto-call**: `/opsx:archive` invokes it as a pre-archive sweep (opt-out via `--skip-audit`).

## Why it exists

From `OPENSPEC_LESSONS.md`:

> When running `/opsx:archive` on a change that touched framework-wide vocabulary, the spec-sync step ONLY merges files that have explicit deltas in `openspec/changes/<name>/specs/`. Everything else in `openspec/` is left untouched and almost always becomes stale.

`audit-drift` exists because the union of `sync-specs` + manual vigilance has been demonstrably insufficient. Even disciplined users miss residue. A scan that runs every time `/opsx:archive` is invoked closes the loop.

## Inputs

- **No required args** — project-wide scan, no `<change-name>` needed (unlike the review commands).
- **Depth modes (mutually exclusive, pick one):**
  - `--fast` — vocabulary residue + lens staleness only. Skips cross-doc consistency (expensive LLM pass) and archive coherence. For quick sanity checks.
  - `--full` (**default**) — all four scan categories, sequential. The workhorse per guiding principle 2.
  - `--thorough` — all categories + extras (deeper cross-doc reasoning, archive coherence extends to a broader window). Specifics tracked in GitHub issue #2 alongside review-command thoroughness.
- **Execution mode (orthogonal):**
  - `--parallel` — scan categories run in concurrent subagents. Categories 1, 3, 4 each become their own subagent; category 2 stays in main. Real win is context partitioning, not just speed.
- **Secondary flags:**
  - `--focus <area>` — limit to `vocabulary` / `lenses` / `docs` / `archive`. Useful when investigating a known issue.
  - `--strict` — fail-fast on first CRITICAL.
  - `--since <ref>` — for archive-coherence scan, override the default "last 5 changes" window with a git ref. (Note: this is the one place a `--since` flag makes sense for orbit — it's about audit window, not review scope.)

## What it reads

- **Source for vocabulary deltas**: archived changes in `openspec/changes/archive/` (within scan window). Their `## RENAMED Requirements` and `## REMOVED Requirements` blocks define what *shouldn't* still appear in current docs.
- **Target docs**: `openspec/specs/**/spec.md`, `openspec/project.md`, `CLAUDE.md`, `*_convention.md`, `README.md`.
- **Lens content**: `openspec/lenses/perspectives.md`, `openspec/lenses/critical-paths.md`.
- **Current capabilities**: `openspec/specs/<capability>/` directories (for resolving lens surface references).
- **Cross-doc claims**: structured statements across project context docs (CLAUDE.md, project.md, *_convention.md).
- **Archive deltas**: recent archived changes' delta specs (within scan window) for cross-checking against current baseline.

## Scan categories

Each category produces findings with CRITICAL / WARNING / SUGGESTION severity, file:line refs, and actionable recommendations.

### Category 1 — Vocabulary residue

The original `OPENSPEC_LESSONS.md` failure mode.

- Build a residue-pattern set from archived changes' deltas:
  - `RENAMED FROM` symbols → expected gone from current state
  - `REMOVED` requirement names and core vocabulary → expected gone
- Grep all target docs for residue patterns.
- For each hit:
  - **Legitimate historical reference** (e.g., a "Removed in v1" note) → no finding.
  - **Stale description of current behavior** → WARNING. Recommend either (a) adding a delta to a future change, or (b) hotfix commit.

Output: WARNING per stale hit; CRITICAL when stale vocabulary in `project.md` or `CLAUDE.md` (these are governing docs — fresh implementer follows them).

### Category 2 — Lens staleness

- For each perspective in `openspec/lenses/perspectives.md`:
  - Resolve referenced surfaces against `openspec/specs/<capability>/`. Missing capabilities → WARNING ("perspective X references surface Y not present in baseline").
  - Best-effort: do the validation criteria still match the capability's current spec? (E.g., "tool names readable" — are the tools still named the way they were?)
- For each critical path in `openspec/lenses/critical-paths.md`:
  - Resolve each touchpoint to a real capability + interaction. Unresolvable touchpoints → WARNING.
  - Check "Expected behavior" against current specs. Severe mismatch → WARNING.

Output: WARNING for stale lens entries. Recommend update or removal. SUGGESTION for minor drift.

### Category 3 — Cross-doc consistency

The most expensive category. Likely the one that benefits most from `--parallel`.

- Extract structured claims from `CLAUDE.md`, `openspec/project.md`, `*_convention.md`:
  - Names of capabilities / surfaces / tools mentioned
  - Quantitative claims (counts, ports, paths)
  - Architectural assertions (who talks to whom, directionality)
- Cross-check claims between docs. Inconsistencies → WARNING.
- Cross-check claims against current specs in `openspec/specs/`. Doc-vs-spec disagreement → WARNING (governing docs override specs in some cases, but the inconsistency is worth flagging).

Output: WARNING per inconsistency; CRITICAL if `project.md` and `openspec/specs/` materially disagree on a capability's behavior.

### Category 4 — Archive coherence

- For each archived change within the scan window (default: last 5; override with `--since <ref>`):
  - Read the archived `proposal.md` and delta specs.
  - For ADDED requirements: did they make it to baseline? (Should be in `openspec/specs/<capability>/spec.md`.) Missing → CRITICAL (sync-specs failure).
  - For RENAMED requirements: do the TO symbols exist in baseline? Are any FROM symbols still present anywhere? Failures → WARNING.
  - For REMOVED requirements: are they actually gone from baseline? Lingering → CRITICAL.

Output: CRITICAL for confirmed sync-specs misses; WARNING for partial propagation.

## Reporting

Standard 3-dimension scorecard, same as review commands.

### Category → dimension roll-up

| Dimension | Categories |
|---|---|
| **Completeness** | Category 4 (Archive Coherence — are all required updates propagated?) |
| **Correctness** | Category 1 (Vocabulary Residue — references valid?), Category 2 (Lens Staleness — lens claims valid?) |
| **Coherence** | Category 3 (Cross-Doc Consistency — docs agree?) |

### Output template

```markdown
## Audit Report: <project> (scan window: last 5 archived changes)

### Summary
| Dimension    | Status                                                   |
|--------------|----------------------------------------------------------|
| Completeness | 5/5 archived changes propagated, 0 baseline misses       |
| Correctness  | 3 vocabulary-residue hits in 2 files, 1 lens staleness   |
| Coherence    | 1 CLAUDE.md ↔ project.md inconsistency                   |

### CRITICAL (1)
1. project.md:78 — references `BridgeServer` as canonical wiring;
   this symbol was removed in archived change
   `mcp-bridge-test-redesign`. project.md is a governing doc;
   a fresh implementer will re-couple to the deleted backend.
   → Update project.md to reference current canonical wiring
     (likely `HostLifecycle` + `MatrixRunner`).

### WARNING (5)
…

### SUGGESTION (2)
…

### Final Assessment
1 critical issue, 5 warnings. Fix project.md before next archive.
Suggested next: fix governing-doc residue, then re-run /opsx:audit-drift.
```

### Final-assessment phrasings

Same shape as review commands; gate varies by invocation context:

| Invoked from | State | Phrasing |
|---|---|---|
| Standalone | ≥1 CRITICAL | `X critical issue(s) found.` (no gate mentioned) |
| Standalone | Only WARNING/SUGGESTION | `No critical issues. Y warning(s) to consider.` |
| Standalone | All clear | `All checks passed.` |
| Pre-archive auto-call | ≥1 CRITICAL | `X critical issue(s) found. Address before \`/opsx:archive\`?` (prompt, not gate) |
| Pre-archive auto-call | Only WARNING/SUGGESTION | `No critical issues. Y warning(s) noted. Proceeding with archive.` |
| Pre-archive auto-call | All clear | `Drift audit clean. Proceeding with archive.` |
| Library call (from review-system) | (any) | Findings folded into review-system's report; no standalone assessment. |

## Heuristics & graceful degradation

- **Bias toward lower severity** when uncertain — same as verify-change.
- **Every finding actionable** — file:line + specific recommendation (delta, hotfix, lens update, etc.).
- **Degrade gracefully**:
  - Empty `openspec/lenses/` → Category 2 skips with a note.
  - No archived changes → Categories 1 and 4 have little to scan; report "no archive context yet."
  - Single doc only (no CLAUDE.md, no project.md, no `*_convention.md`) → Category 3 trivially passes.
- **Pushback hook** — if a finding contradicts current state (e.g., "stale ref to X" but X was already fixed in a recent commit), self-correct and note "stale finding suppressed." Same discipline as review commands.
- **Don't short-circuit** — run all categories even if early ones report CRITICAL. `--strict` opts into fail-fast.

## Open design questions

1. **Window for archive-coherence scan** (Category 4). Default to "last 5 archived changes" or "last 30 days" or "current release cycle"? Probably configurable in `openspec/config.yaml` or via `--since <ref>` flag. Lean: last 5 archives as default; configurable.
2. **Vocabulary-pattern source for Category 1**. Pure archive delta extraction, or also pull from `*_convention.md` (deprecated-names section)? Lean: archive deltas as primary source; conventions as optional hint file if it exists.
3. **Cross-doc reasoning in Category 3 is the most expensive pass**. Worth subagent parallelism (`--parallel`) for sure. May also benefit from caching across runs (covered by issue #1).
4. **Should archive auto-invoke be hard-gated on CRITICAL?** Decided no (per Decisions in `explore.md`): user prompted to confirm, not blocked. Re-examine if v1 shows users routinely archiving with critical drift.
5. **What goes in `--thorough`?** Tracked in issue #2 along with review-command thoroughness specifics.

## Composition with related commands

```
                  /opsx:audit-drift
                   │
       ┌───────────┼───────────────────────────┐
       │           │                            │
       │       library                    pre-archive
   standalone    call from              auto-call from
    invocation  review-system            /opsx:archive
                Pass 6                  (opt-out --skip-audit)

       │           │                            │
       ▼           ▼                            ▼
  reports        folds findings           reports + prompt
  findings;      into review-system        before completing
  user reads     scorecard                 archive
```

`/opsx:review-system` Pass 6 stops being "the drift/residue check"; it's just "calls `/opsx:audit-drift`." Composition matches Pass 0 (calls `verify-change`). Two upstream-shaped delegations in the same command — orbit's `review-system` is a thin orchestrator over upstream + orbit primitives.

## Parallels with review commands

By design:

| | `review-proposal` | `review-system` | `audit-drift` |
|---|---|---|---|
| Scope | one change's artifacts | one change's whole-system impact | project-wide |
| Stage | pre-apply | post-apply | continuous + pre-archive |
| Wraps | nothing upstream | `verify-change` | (nothing — orbit primitive) |
| Library use | — | calls `audit-drift` | called by review-system + archive |
| Standard | 3-dim scorecard, severity | 3-dim scorecard, severity | 3-dim scorecard, severity |
| Flag family | --fast/--full/--thorough, --parallel, … | (same) | (same, with --since for window) |

All three commands share the same output convention; adopters learn it once.
