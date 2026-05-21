# Reference: audit-drift run-summary schema

Each `/opsx:audit-drift` invocation persists a JSON summary. Path varies by context:

- **Change-scoped** (library or pre-archive): `openspec/changes/<change-name>/.orbit-runs/audit-drift-<TS>.json`
- **Standalone** (no change context): `openspec/.orbit-runs/audit-drift-<TS>.json` (create `openspec/.orbit-runs/` if needed)

## Universal spine inheritance

This schema inherits the 6-field **universal spine** from `orbit-conventions`'s `Internal-run JSON summary format` requirement (see `openspec/specs/orbit-conventions/spec.md`):

- `command` (here: `"audit-drift"`)
- `timestamp` (ISO-8601 UTC, `YYYY-MM-DDTHH-MM-SSZ`)
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
      "recommendation": "Delta the file in a future change or apply a hotfix commit."
    }
  ],
  "stale_suppressed": [
    { "category": "<category id>", "title": "...", "evidence": "..." }
  ]
}
```

## Field notes

- **`context`** drives `final_assessment` phrasing (standalone / pre-archive / library) and whether the run is change-scoped or project-level.
- **`caller`** only populated for library + pre-archive contexts.
- **`categories_run` / `categories_skipped`** are strings (category IDs); skip reasons in the report body, not the array.
- **`findings_summary.by_category`** keys are category IDs `"1"` through `"4"`.
- **`final_assessment`** is `null` for library context (findings handed back to the caller for folding into the caller's report).
