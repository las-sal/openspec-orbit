---
name: "OPSX: Address Reviews"
description: Resolve @review: markers across the repo (or external-review findings via --from-file) with pushback discipline
category: Workflow
tags: [workflow, address-reviews, orbit]
---
Resolve `@review:` markers anywhere in the repo (or ingest external-review findings from a file) by walking each through: pushback → classify → fix → ripple-flag → remove-marker.

**Primary use case**: close the cross-AI review cycle. Ingest the external-AI findings file written by `/opsx:review-external` via `--from-file`, walk each finding with pushback discipline.

**Secondary use case**: walk inline `@review:` markers scattered across the repo (markdown / source / configs) with structured pushback. Markers are removed from source on resolution.

## Input

`/opsx:address-reviews [<scope>] [--from-file <path>] [flags]`

- `<scope>` — optional. Path, pattern, or change name. Resolves in this order: active change at `openspec/changes/<scope>/` → archived change at `openspec/changes/archive/<YYYY-MM-DD>-<scope>/` (regex match; latest date wins on multi-match) → path/pattern fallback. Default (no positional): whole-repo scan with safe exclusions (`.git`, `node_modules`, `dist`, `build`).
- `--from-file <path>` — ingest review findings from a file. Auto-detects format via content sniff: external-review markdown (per `references/external-findings-format.md`) OR internal findings JSON (`review-<mode>-*.json` or `audit-drift-*.json`, per `references/internal-findings-format.md`). V1 internal-JSON ingest accepts `command: "review"` OR `command: "audit-drift"`; other internal JSON commands (`address-reviews`, `apply`, `archive`, etc.) are rejected with a clean error. `command: "address-reviews"` is rejected on purpose (cycle prevention).

## Flags

```
--keep-resolved-markers          debug: don't remove markers after resolution
```

(Scope restriction is handled by the positional `<scope>` argument, which accepts a path, glob pattern, or change name. `--only` and `--list` were considered but cut from lean v1; see issue #3.)

## What it does

Invokes the `openspec-address-reviews` skill, which executes the lean v1 lifecycle:

1. **Discover** — discovery priority order: (a) `@review:` markers in scope; (b) if change-name positional + no markers + no `--from-file`: auto-discover most-recent `review-<mode>-*.json` OR `audit-drift-*.json` in the change's `.orbit-runs/` (single global most-recent by filename `<TS>` token; lexicographic tie-break on collision); (c) if neither: clean "no findings" exit. Explicit `--from-file <path>` is exclusive — marker grep + auto-discovery both skip; content-sniff routes to JSON parser (leading `{`) or markdown parser (leading `# External Review:`); else format-mismatch error.
2. **Triage** — present a numbered list; user can scope to a subset
3. **Walk each sequentially**:
   - **Pushback** — verify against current state (grep / git log / file read); classify stale findings and suppress them
   - **Classify** — stale / trivial fix / decision required / unresolvable
   - **Fix** — apply trivial fix, or surface 2–4 options via `AskUserQuestion` for decisions
   - **Ripple flag** — list affected related files (no auto-cascade in v1)
   - **Remove marker** — delete from source on resolution (invariant; `--keep-resolved-markers` overrides)
4. **Report** — emit a resolution log with ✓ Resolved / ⚠ Stale / ⏸ Deferred / ✗ Escalated counts and per-marker entries
5. **Persist** — write run summary to `.orbit-runs/address-reviews-<TS>.json`

## Output

Resolution log (NOT a 3-dimension scorecard — this command resolves rather than scans):

- Summary table (✓ ⚠ ⏸ ✗ counts)
- Per-section listings: file:line, brief description, action taken, ripple-flagged files
- Final-assessment line: remaining markers in scope + suggested next step

## Marker convention

| Form | Meaning |
|---|---|
| `@review: <text>` | Needs review/decision — full lifecycle |
| `@review(escalated): <text>` | Escalated; not auto-walked unless explicitly scoped |
| `@todo: <text>` | Out of scope (known follow-up work, not a review item) |

Markdown carries the marker bare; source code and configs wrap it in the file type's comment syntax (`// @review:`, `# @review:`, `/* @review: */`).

## Execution disciplines

- **Pushback (primary)** — verify each marker against current state before fixing. Stale → remove without edit + evidence note.
- **Read-before-reference** — re-read each file before applying any fix; verify after edits.
- **Change completeness** — ripple-flag related files; v1 lists them rather than auto-cascading.

## Constraints

- **Never creates new `@review:` markers.** Only `/opsx:review --as proposal --mark` does that. `--mark` is optional, NOT a prerequisite — auto-discovery makes the canonical `/opsx:review <name>` → `/opsx:address-reviews <name>` workflow work without pre-marking.
- **No auto-cascade in v1.** Ripple-flagged files are listed, not edited.

See `.claude/skills/openspec-address-reviews/SKILL.md` for full lifecycle, classification heuristics, ingest format, and worked example.
