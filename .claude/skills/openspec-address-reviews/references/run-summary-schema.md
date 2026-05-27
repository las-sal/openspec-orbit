# Reference: address-reviews run-summary schema

Each `/opsx:address-reviews` invocation persists a JSON summary. Path varies:

- **Change-scoped** (when scope is a single change directory or `--from-file` points into a change's `.orbit-runs/`): `openspec/changes/<change-name>/.orbit-runs/address-reviews-<TS>.json`
- **Whole-repo / cross-change**: `openspec/.orbit-runs/address-reviews-<TS>.json`

## Universal spine inheritance

This schema inherits the 6-field **universal spine** from `orbit-conventions`'s `Internal-run JSON summary format` requirement (see `openspec/specs/orbit-conventions/spec.md`):

- `command` (here: `"address-reviews"`)
- `timestamp` (ISO-8601 UTC; JSON field `YYYY-MM-DDTHH:MM:SSZ`; filename `<TS>` token `YYYY-MM-DDTHH-MM-SSZ` with colons replaced by hyphens for filesystem safety)
- `change` (change name string for change-scoped; `null` for whole-repo / cross-change scope)
- `final_assessment` (narrative)
- `next_recommended` (verbatim recommendation, e.g., `"re-run /opsx:review --as proposal to confirm convergence"`)
- `kind: "editorial"` (address-reviews is an editorial command per the kind taxonomy)

The schema below documents per-command extensions ADDED to that spine (`source`, `source_path`, `external_reviewer`, `walk_mode`, `walk_mode_source`, `walk_mode_shifted_at_finding`, `input_findings_summary`, `pushback_verification`, `resolution_summary`, `resolutions` with `recommendation_fork` and `ripple_cascade` sub-objects, `remaining_markers_in_scope`, `persisted_escalations`). Per-command extensions are address-reviews-specific state.

## Schema (v2)

```json
{
  "command": "address-reviews",
  "timestamp": "<ISO-8601>",
  "change": "<change-name string for change-scoped; null for whole-repo / cross-change>",
  "final_assessment": "<short narrative of the resolution outcome>",
  "next_recommended": "<suggested next command, e.g., 're-run /opsx:review --as proposal to confirm convergence'>",
  "kind": "editorial",
  "source": "whole-repo" | "scope" | "from-file" | "auto-discovered",
  "source_path": "<scope path or --from-file path or auto-discovered JSON path or null>",
  "source_command": "<for auto-discovered source: the selected JSON's command field — 'review' or 'audit-drift'; null otherwise>",
  "source_token": "<for auto-discovered source: filename <TS> token of the selected JSON; null otherwise>",
  "latest_apply_token": "<for auto-discovered source: most-recent apply-*.json filename token in the same .orbit-runs/, or null if no apply JSON exists; null for other sources>",
  "tie_break_rationale": "<for auto-discovered source: optional string describing how a TS-token tie-break was resolved; absent (or null) otherwise>",
  "external_reviewer": "<from --from-file's Reviewer field, if applicable, else null>",
  "walk_mode": "per_finding" | "batch",
  "walk_mode_source": "flag" | "verbal" | "command-shape-interruption",
  "walk_mode_shifted_at_finding": "<1-indexed finding number when mode shifted mid-walk; present only when walk_mode_source == 'command-shape-interruption'>",
  "input_findings_summary": {
    "critical": 0,
    "warning": 0,
    "suggestion": 0
  },
  "pushback_verification": "<short note: how many findings verified against current state; how many stale>",
  "resolution_summary": {
    "resolved": 0,
    "stale_suppressed": 0,
    "deferred": 0,
    "escalated": 0
  },
  "resolutions": [
    {
      "severity": "CRITICAL" | "WARNING" | "SUGGESTION",
      "title": "<finding title>",
      "marker_source": "inline" | "external" | "internal-review" | "audit-drift",
      "file": "<path>",
      "line": 41,
      "classification": "trivial_fix" | "decision_required" | "stale" | "unresolvable",
      "action": "<what was done — applied edit, filed as task, converted to @todo:, escalated, etc.>",
      "recommendation_fork": {
        "detected": true,
        "source": "structured" | "heuristic",
        "options_presented": [{ "label": "A", "body": "..." }, { "label": "B", "body": "..." }],
        "chosen": "A",
        "discuss_invoked": false,
        "structured_path_skipped_reason": "<optional; e.g., 'only 1 entry in array', 'missing label on entry index 2'>"
      },
      "ripple_cascade": {
        "applied": ["<paths edited as part of the cascade>"],
        "flagged_not_applied": [{ "path": "...", "reason": "<OUT-category reason OR `--no-cascade suppressed`>" }]
      },
      "outcome": "resolved" | "stale" | "deferred" | "escalated"
    }
  ],
  "remaining_markers_in_scope": 0,
  "persisted_escalations": [
    { "file": "...", "line": 0, "title": "...", "reason": "..." }
  ]
}
```

## v1 → v2 format migration (reader guidance)

The v2 shape is a **structural change** from v1, not a rename. Readers handling archived JSONs from both eras MUST detect which version they're reading:

- **v2 detection**: presence of `walk_mode` (top-level) OR `ripple_cascade` (per-resolution) indicates v2. v2 emits these fields on every walk.
- **v1 detection**: presence of `ripple_flagged_files_aggregate` (top-level flat array of strings) indicates v1. v1 did NOT emit `walk_mode` or per-resolution `ripple_cascade`.

**Why NOT a rename**: v1 aggregated ripple-flagged paths in a single top-level array across ALL resolutions, with no per-resolution attribution. v2 splits per-resolution into `ripple_cascade.applied[]` + `ripple_cascade.flagged_not_applied[]`. The shapes are NOT bijective — v1 archived JSONs cannot be losslessly upconverted to v2 (per-resolution attribution is lost in v1). Treat v1's aggregate as the union of all per-resolution ripple sets across the run; attribution to specific findings is unavailable.

**v2 fields not present in v1**: `walk_mode`, `walk_mode_source`, `walk_mode_shifted_at_finding`, per-resolution `recommendation_fork`, per-resolution `ripple_cascade`. Downstream consumers (e.g., `orbit-status` sibling-repo tier-1 best-effort parse) handle absence as v1 and proceed gracefully.

## Field notes

- **`source`** distinguishes invocation paths: `whole-repo` for default scan, `scope` for positional `<scope>` argument, `from-file` for `--from-file <path>`, `auto-discovered` for the auto-discovery fallback (change-name positional + markers absent + candidate JSON found in `.orbit-runs/` — per the `Auto-discovery resolution log captures audit-trail evidence` scenario in the orbit-address-reviews spec).
- **`source_path`** carries the scope, `--from-file` path, OR auto-discovered JSON path; `null` for `whole-repo`. For `auto-discovered` source, this is the repo-relative path to the selected JSON (e.g., `openspec/changes/<name>/.orbit-runs/review-system-2026-05-27T00-43-10Z.json`).
- **`source_command`**, **`source_token`**, **`latest_apply_token`**, **`tie_break_rationale`** — the auto-discovery audit-trail fields. Emitted ONLY when `source: "auto-discovered"`; `null` (or absent for `tie_break_rationale`) for other sources. Together they document the discovery decision: `source_command` = the selected JSON's top-level `command` value (`"review"` or `"audit-drift"`); `source_token` = the selected JSON's filename `<TS>` token (the winner of the recency comparison); `latest_apply_token` = the most-recent `apply-*.json` filename token in the same `.orbit-runs/` (existence-checked at discovery time; supports the D-no-stale-detection design choice — orbit doesn't refuse to walk a JSON older than the latest apply, but the resolution log records both timestamps so downstream auditors can assess staleness); `tie_break_rationale` = optional explanation for when the recency comparison invoked the lexicographic-sort tie-break.
- **`external_reviewer`** parsed from the `**Reviewer**:` field in the `--from-file` input; lets downstream tools track which AI's findings have been ingested. Only populated when `marker_source` is `external` (external-review markdown input); set to `null` for inline / internal-review / audit-drift sources (those have no human-readable reviewer-name field).
- **`input_findings_summary`** counts findings by severity at the input boundary (before pushback suppression). Total findings always = sum of `resolved` + `stale_suppressed` + `deferred` + `escalated` in `resolution_summary`.
- **`pushback_verification`** is a short prose note (1-2 sentences) summarizing pushback work — "all 9 findings verified against current state; 0 stale suppressions" or "9 findings; 3 stale-suppressed with commit evidence."
- **`marker_source`** distinguishes virtual-marker provenance: `inline` (grep-found `@review:` markers), `external` (external-review markdown parsed from `--from-file`), `internal-review` (internal `review-<mode>-*.json` JSON parsed from `--from-file`), `audit-drift` (`audit-drift-*.json` JSON parsed from `--from-file`).
- **`classification`** is the pushback-and-classify outcome before action.
- **`outcome`** is the final disposition; aligns with the ✓ Resolved / ⚠ Stale / ⏸ Deferred / ✗ Escalated counts in the resolution log.
- **`persisted_escalations`** captures `@review(escalated):` markers deliberately left in place; mirrors the resolution log's escalated section so downstream queries don't re-parse the log.
- **`next_recommended`** is the closing suggestion shown in the final-assessment line.
- **`walk_mode`** records the lifecycle mode the run used (per #11): `per_finding` (default — each marker walked sequentially with its own pushback → classify → fix → cascade → remove cycle) or `batch` (all markers complete pushback + classify + fix in one pass, then cascade as a single aggregated step).
- **`walk_mode_source`** distinguishes how batch-mode was entered: `flag` (`--batch` argv), `verbal` (recognized batch-intent phrase in the invocation message — "fix them all", "batch them", "go ahead with all", or equivalent), `command-shape-interruption` (bare unambiguous mid-walk mode-switch message — "go batch", "switch to batch", "batch the rest"). Omitted when `walk_mode == "per_finding"`.
- **`walk_mode_shifted_at_finding`** is the 1-indexed finding number at which the mode shifted; present ONLY when `walk_mode_source == "command-shape-interruption"`; omitted otherwise.
- **`recommendation_fork`** (per-resolution, OPTIONAL) — present only when decision-fork detection fired in Step 3b.5 (per #18). Object captures: `detected` (always true when present), `source` (`"structured"` = parser used the input finding's `recommendation_options[]` field; `"heuristic"` = parser scanned the recommendation prose for disjunctive signals), `options_presented` (the options shown to the user as `[{label, body}]`), `chosen` (the option label the user picked; matches one of `options_presented[].label`), `discuss_invoked` (true when the `[discuss]` escape hatch was used before the final choice), `structured_path_skipped_reason` (optional; present only when structured detection was attempted but skipped due to malformed input — e.g., "only 1 entry in array", "missing label on entry index 2"). Omitted entirely for findings with no fork.
- **`ripple_cascade`** (per-resolution) — replaces v1's top-level `ripple_flagged_files_aggregate`. Object splits each resolution's ripples into `applied` (paths cascade-edited consistent with the primary fix; recorded for audit) AND `flagged_not_applied` (paths identified as ripple targets but NOT edited, with `{path, reason}` entries). Reasons fall into TWO source categories:
  - **Structural OUT-category reasons** (4 codes): `audit-trail file; cascade skipped by policy` / `baseline spec; add a delta to your current change's specs/<capability>/spec.md to capture this ripple` / `cross-change ripple; cascade scope is current change only` / `safe-exclusion path; never edited`. Fire when cascade was ON and the file matched an OUT prefix.
  - **Mode-suppression reason** (1 code): `--no-cascade suppressed`. Fires uniformly for ALL ripple-flagged files when `--no-cascade` was set, regardless of IN/OUT classification.

## Worked-example JSON snippets

### (a) Walk-mode clean run (per_finding default, all IN, no fork)

```json
{
  "command": "address-reviews",
  "walk_mode": "per_finding",
  "resolutions": [
    {
      "title": "<finding>",
      "classification": "trivial_fix",
      "action": "applied edit at design.md:14",
      "ripple_cascade": {
        "applied": ["openspec/changes/foo/design.md", "openspec/changes/foo/tasks.md"],
        "flagged_not_applied": []
      },
      "outcome": "resolved"
    }
  ]
}
```

### (b) Batch-mode with verbal source

```json
{
  "command": "address-reviews",
  "walk_mode": "batch",
  "walk_mode_source": "verbal",
  "resolutions": [/* ... */]
}
```

### (c) Decision-fork with [discuss] invoked

```json
{
  "resolutions": [
    {
      "title": "Test coverage gap — file follow-up or extend scope?",
      "classification": "decision_required",
      "recommendation_fork": {
        "detected": true,
        "source": "structured",
        "options_presented": [
          { "label": "A", "body": "file a follow-up issue tracking the v2 polish" },
          { "label": "B", "body": "extend scope to tasks.md" }
        ],
        "chosen": "B",
        "discuss_invoked": true
      },
      "action": "applied user choice B — added Group 19 to tasks.md",
      "ripple_cascade": { "applied": ["openspec/changes/foo/tasks.md"], "flagged_not_applied": [] },
      "outcome": "resolved"
    }
  ]
}
```

### (d) `--no-cascade` — all ripples flagged-not-applied

```json
{
  "walk_mode": "per_finding",
  "resolutions": [
    {
      "title": "<finding>",
      "ripple_cascade": {
        "applied": [],
        "flagged_not_applied": [
          { "path": "openspec/changes/foo/design.md", "reason": "--no-cascade suppressed" },
          { "path": "openspec/changes/foo/tasks.md", "reason": "--no-cascade suppressed" }
        ]
      }
    }
  ]
}
```
