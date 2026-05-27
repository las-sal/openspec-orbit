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

The schema below documents per-command extensions ADDED to that spine (`source`, `source_path`, `external_reviewer`, `input_findings_summary`, `pushback_verification`, `resolution_summary`, `resolutions`, `remaining_markers_in_scope`, `persisted_escalations`). Per-command extensions are address-reviews-specific state.

## Schema

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
  "source_token": "<for auto-discovered source: filename <TS> token of the selected JSON (e.g., '2026-05-27T00-43-10Z'); null otherwise>",
  "latest_apply_token": "<for auto-discovered source: most-recent apply-*.json filename token in the same .orbit-runs/, or null if no apply JSON exists; null for other sources>",
  "tie_break_rationale": "<for auto-discovered source: optional string describing how a TS-token tie-break was resolved; absent (or null) otherwise>",
  "external_reviewer": "<from --from-file's Reviewer field, if applicable, else null>",
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
      "files_updated": ["<paths edited as part of the resolution>"],
      "ripple_flagged": ["<paths flagged for sibling consistency, not edited>"],
      "outcome": "resolved" | "stale" | "deferred" | "escalated"
    }
  ],
  "remaining_markers_in_scope": 0,
  "persisted_escalations": [
    { "file": "...", "line": 0, "title": "...", "reason": "..." }
  ]
}
```

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
