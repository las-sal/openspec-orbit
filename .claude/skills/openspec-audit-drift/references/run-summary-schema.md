# Reference: audit-drift run-summary schema

Each `/opsx:audit-drift` invocation persists a JSON summary. Path varies by invocation mode (three modes, per `orbit-run-summary-emit`'s `Audit-drift standalone recommendations` requirement):

- **Change-scoped standalone** (`/opsx:audit-drift <name>` — user invokes with a change name; no caller signal): `openspec/changes/<change-name>/.orbit-runs/audit-drift-<TS>.json`
- **Project-wide standalone** (`/opsx:audit-drift` — no positional; no caller signal): `openspec/.orbit-runs/audit-drift-<TS>.json` (create `openspec/.orbit-runs/` if needed)
- **Library / pre-archive** (caller signal present — invoked by `/opsx:review --as system` Pass 6 or `/opsx:archive` pre-archive sweep): `openspec/changes/<change-name>/.orbit-runs/audit-drift-<TS>.json` if change context is provided by caller; otherwise inline findings fold into the caller's archive emit per the existing convention.

## Universal spine inheritance

This schema inherits the 6-field **universal spine** from `orbit-conventions`'s `Internal-run JSON summary format` requirement (see `openspec/specs/orbit-conventions/spec.md`):

- `command` (here: `"audit-drift"`)
- `timestamp` (ISO-8601 UTC; JSON field `YYYY-MM-DDTHH:MM:SSZ`; filename `<TS>` token `YYYY-MM-DDTHH-MM-SSZ` with colons replaced by hyphens for filesystem safety)
- `change` (change name string for change-scoped / library / pre-archive contexts; `null` for project-wide standalone)
- `final_assessment` (narrative)
- `next_recommended` (verbatim recommendation; varies by findings-vs-clean per `orbit-run-summary-emit`'s `Audit-drift standalone recommendations` requirement)
- `kind: "editorial"` (audit-drift is an editorial command per the kind taxonomy)

The schema below documents per-command extensions ADDED to that spine (`context`, `caller`, `depth`, `flags`, `categories_run`, `categories_skipped`, `findings_summary`, `findings`, `stale_suppressed`). Per-command extensions are audit-drift-specific state.

## Schema

```json
{
  "command": "audit-drift",
  "timestamp": "<ISO-8601>",
  "change": "<change-name string for change-scoped/library/pre-archive; null for project-wide standalone>",
  "final_assessment": "<stock phrasing or null for library context>",
  "next_recommended": "<e.g., '/opsx:address-reviews <name> --from-file <this-json>' on findings; verbatim copy from prior latest JSON on clean (per orbit-run-summary-emit's Audit-drift standalone recommendations); null for library/pre-archive>",
  "kind": "editorial",
  "context": "standalone" | "library" | "pre-archive",
  "caller": "<calling command if library/pre-archive, else null>",
  "depth": "fast" | "full" | "thorough",
  "flags": {
    "parallel": false,
    "focus": null,
    "since": null,
    "strict": false
  },
  "categories_run": ["1", "2", "3", "4"],
  "categories_skipped": [],
  "findings_summary": {
    "critical": 0,
    "warning": 0,
    "suggestion": 0,
    "by_category": {
      "1": { "critical": 0, "warning": 0, "suggestion": 0 },
      "2": { "critical": 0, "warning": 0, "suggestion": 0 },
      "3": { "critical": 0, "warning": 0, "suggestion": 0 },
      "4": { "critical": 0, "warning": 0, "suggestion": 0 }
    }
  },
  "findings": [
    {
      "category": "1",
      "severity": "CRITICAL" | "WARNING" | "SUGGESTION",
      "file": "openspec/specs/foo/spec.md",
      "line": 42,
      "title": "Stale 'BridgeServer' reference (renamed to HostLifecycle in 2026-03)",
      "recommendation": "Delta the file in a future change or apply a hotfix commit.",
      "recommendation_options": [
        { "label": "A", "body": "<option A body>" },
        { "label": "B", "body": "<option B body>" }
      ]
    }
  ],
  "stale_suppressed": [
    { "category": "<category id>", "title": "...", "evidence": "..." }
  ]
}
```

## Field notes

- **`context`** drives `final_assessment` phrasing — one of `standalone | library | pre-archive`. Note: `context` alone does NOT distinguish project-wide from change-scoped standalone — that distinction is inferred from the `change` field (`null` = project-wide standalone; `<name>` = change-scoped standalone, library, or pre-archive). The four cells are: standalone-project-wide (`context: "standalone"`, `change: null`), standalone-change-scoped (`context: "standalone"`, `change: <name>`), library (`context: "library"`, `change: <name>`), pre-archive (`context: "pre-archive"`, `change: <name>`).
- **`caller`** only populated for library + pre-archive contexts.
- **`categories_run` / `categories_skipped`** are strings (category IDs); skip reasons in the report body, not the array.
- **`findings_summary.by_category`** keys are category IDs `"1"` through `"4"`.
- **`final_assessment`** is `null` for library context (findings handed back to the caller for folding into the caller's report).
- **`recommendation_options`** is OPTIONAL on each finding. Present only when the finding's recommendation is genuinely disjunctive — multiple defensible remediation paths the user must choose between. Producer-side contract (per `orbit-audit-drift` spec's `Optional recommendation_options field on audit-drift finding entries`): MUST contain ≥ 2 entries; each entry MUST have non-empty `label` (typically `"A"`, `"B"`, `"C"`, …) and non-empty `body`. Per-category emit guidance: Category 1 (vocabulary residue) typically single-recommendation; Category 2 (lens staleness) often disjunctive (rename-vs-remove); Category 3 (cross-doc consistency) often disjunctive (which side is canonical); Category 4 (archive coherence) typically single. The prose `recommendation` field still summarizes the disjunction for human readers. Field shape is identical to `orbit-review`'s `recommendation_options`; consumer (address-reviews) parses both producers uniformly. Omit for single-recommendation findings.
