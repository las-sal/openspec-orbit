# Reference: archive run-summary schema

The orbit additions to `/opsx:archive` write a JSON summary to `openspec/changes/archive/<YYYY-MM-DD>-<change-name>/.orbit-runs/archive-<TS>.json` after a successful archive operation (the `<YYYY-MM-DD>-` prefix is added by upstream's archive move step).

## Universal spine inheritance

This schema inherits the 6-field **universal spine** from `orbit-conventions`'s `Internal-run JSON summary format` requirement (see `openspec/specs/orbit-conventions/spec.md`):

- `command` (here: `"archive"`)
- `timestamp` (ISO-8601 UTC, `YYYY-MM-DDTHH-MM-SSZ`)
- `change` (change name string)
- `final_assessment` (narrative of what was archived, e.g., `"Archived <change-name> to <archive_path>."`)
- `next_recommended` (verbatim recommendation, e.g., `"Change archived to <archive_path>. Run /opsx:new or /opsx:explore to start the next change, or /opsx:audit-drift for a project-wide drift check."`)
- `kind: "lifecycle"` (archive is THE lifecycle command per the kind taxonomy — transitions a change out of the active set)

The schema below documents per-command extensions ADDED to that spine (`archive_path`, `audit`, `unresolved_markers`, `user_decision`, `sync_specs`, `warnings`). Per-command extensions are archive-specific state.

**Note on `sync_specs`** (transitional): this field persists from pre-#6 architecture. `openspec-orbit#6` will deprecate `/opsx:sync-specs` entirely; at that point `sync_specs` may be removed or repurposed in a follow-up change.

## Schema

```json
{
  "command": "archive",
  "timestamp": "<ISO-8601>",
  "change": "<change-name>",
  "final_assessment": "<narrative of what was archived, e.g., 'Archived <change-name> to <archive_path>.'>",
  "next_recommended": "Change archived to <archive_path>. Run /opsx:new or /opsx:explore to start the next change, or /opsx:audit-drift for a project-wide drift check.",
  "kind": "lifecycle",
  "archive_path": "openspec/changes/archive/<YYYY-MM-DD>-<change-name>/",
  "audit": {
    "ran": true | false,
    "skipped_via_flag": false,
    "failure_reason": "<text or null>",
    "findings_summary": { "critical": 0, "warning": 0, "suggestion": 0 },
    "summary_path": "<path to the audit-drift-<TS>.json summary, or null if not run>"
  },
  "unresolved_markers": {
    "found_at_archive_time": 0,
    "user_action": "addressed" | "converted_to_todo" | "proceeded_with_markers" | "no_markers_found"
  },
  "user_decision": "proceeded_with_no_critical" | "proceeded_despite_critical" | "audit_skipped_via_flag" | "aborted",
  "sync_specs": {
    "ran": true | false,
    "skipped_reason": "<text or null>",
    "capabilities_updated": ["<capability name>"],
    "counts": { "added": 0, "modified": 0, "removed": 0, "renamed": 0 }
  },
  "warnings": [
    "<text — e.g., 'Archived with 2 incomplete artifacts'>"
  ]
}
```

## Field notes

- **`audit.ran`** = `false` either because `--skip-audit` was set (also flag `skipped_via_flag: true`) or because audit-drift failed (also set `failure_reason`). The two cases are disambiguated by which of those flags is set.
- **`audit.findings_summary`** mirrors the audit-drift run summary's counts; the full audit detail lives at the path in `audit.summary_path`.
- **`unresolved_markers.user_action`** maps to the `AskUserQuestion` outcome from the marker-warning step:
  - `addressed` — user halted to run `/opsx:address-reviews`; this archive run did not complete (no summary written in that case — this field is only set when the archive actually proceeds, so `addressed` won't actually appear here; documenting for completeness)
  - `converted_to_todo` — bulk transformation applied; archive proceeded
  - `proceeded_with_markers` — user chose to proceed with markers in place
  - `no_markers_found` — no markers detected; no prompt fired
- **`user_decision`** captures the gate decision:
  - `proceeded_with_no_critical` — clean audit (or warnings only), no prompt fired
  - `proceeded_despite_critical` — user explicitly chose to proceed past CRITICAL findings
  - `audit_skipped_via_flag` — `--skip-audit` was used
  - `aborted` — user aborted at the critical-findings prompt; archive cancelled (no summary written — documenting for completeness)
- **`sync_specs.capabilities_updated`** lists capability names whose `openspec/specs/<capability>/spec.md` was edited.
- **`sync_specs.counts`** mirrors sync-specs' own delta count; lets downstream tools query without re-parsing.
- **`warnings`** captures the upstream warning surfaces (incomplete artifacts, incomplete tasks, sync skipped) — kept human-readable since they vary in shape.
