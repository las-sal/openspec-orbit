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
  "source": "whole-repo" | "scope" | "from-file",
  "source_path": "<scope path or --from-file path or null>",
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

- **`source`** distinguishes invocation paths: `whole-repo` for default scan, `scope` for positional `<scope>` argument, `from-file` for `--from-file <path>`.
- **`source_path`** carries the scope or file path; `null` for `whole-repo`.
- **`external_reviewer`** parsed from the `**Reviewer**:` field in the `--from-file` input; lets downstream tools track which AI's findings have been ingested. Only populated when `marker_source` is `external` (external-review markdown input); set to `null` for inline / internal-review / audit-drift sources (those have no human-readable reviewer-name field).
- **`input_findings_summary`** counts findings by severity at the input boundary (before pushback suppression). Total findings always = sum of `resolved` + `stale_suppressed` + `deferred` + `escalated` in `resolution_summary`.
- **`pushback_verification`** is a short prose note (1-2 sentences) summarizing pushback work — "all 9 findings verified against current state; 0 stale suppressions" or "9 findings; 3 stale-suppressed with commit evidence."
- **`marker_source`** distinguishes virtual-marker provenance: `inline` (grep-found `@review:` markers), `external` (external-review markdown parsed from `--from-file`), `internal-review` (internal `review-<mode>-*.json` JSON parsed from `--from-file`), `audit-drift` (`audit-drift-*.json` JSON parsed from `--from-file`).
- **`classification`** is the pushback-and-classify outcome before action.
- **`outcome`** is the final disposition; aligns with the ✓ Resolved / ⚠ Stale / ⏸ Deferred / ✗ Escalated counts in the resolution log.
- **`persisted_escalations`** captures `@review(escalated):` markers deliberately left in place; mirrors the resolution log's escalated section so downstream queries don't re-parse the log.
- **`next_recommended`** is the closing suggestion shown in the final-assessment line.
