# Reference: review run-summary schema

Each `/opsx:review` invocation persists a JSON summary to `openspec/changes/<change-name>/.orbit-runs/review-<mode>-<TS>.json` where `<TS>` is ISO-8601 with hyphens (e.g., `review-proposal-2026-05-18T14-35-23Z.json`).

Create `.orbit-runs/` if it doesn't exist. The file is **committed** (not gitignored) so iteration history travels with the change into archive.

## Universal spine inheritance

This schema inherits the 6-field **universal spine** from `orbit-conventions`'s `Internal-run JSON summary format` requirement (see `openspec/specs/orbit-conventions/spec.md`):

- `command` (matches filename prefix; here: `"review"`)
- `timestamp` (ISO-8601 UTC; JSON field `YYYY-MM-DDTHH:MM:SSZ`; filename `<TS>` token `YYYY-MM-DDTHH-MM-SSZ` with colons replaced by hyphens for filesystem safety)
- `change` (change name string)
- `final_assessment` (narrative; the stock final-assessment line)
- `next_recommended` (verbatim recommendation, e.g., `"/opsx:address-reviews <name> --from-file <path>"`)
- `kind: "editorial"` (review is an editorial command per the kind taxonomy)

The schema below documents per-command extensions ADDED to that spine (`mode`, `iteration`, `depth`, `flags`, `passes_run`, `passes_skipped`, `findings_summary`, `findings`, `stale_suppressed`, `iteration_note`). Per-command extensions are review-specific state; they supplement, not replace, the spine.

## Schema

```json
{
  "command": "review",
  "timestamp": "<ISO-8601>",
  "change": "<change-name>",
  "final_assessment": "<stock phrasing from final-assessment table>",
  "next_recommended": "<e.g., '/opsx:address-reviews <name> --from-file <path>' or 'proceed to /opsx:apply <name>'>",
  "kind": "editorial",
  "mode": "proposal" | "system",
  "iteration": <integer, per-mode>,
  "depth": "fast" | "full" | "thorough",
  "flags": {
    "parallel": false,
    "focus": null,
    "strict": false,
    "fresh": false,
    "mark": false,
    "skip_verify": false
  },
  "passes_run": ["1", "2", ...],
  "passes_skipped": [],
  "findings_summary": {
    "critical": 0,
    "warning": 0,
    "suggestion": 0,
    "by_pass": {
      "1": { "critical": 0, "warning": 0, "suggestion": 0 },
      "2": { "critical": 0, "warning": 0, "suggestion": 0 }
    }
  },
  "findings": [
    {
      "pass": "1",
      "severity": "CRITICAL" | "WARNING" | "SUGGESTION",
      "file": "design.md",
      "line": 159,
      "title": "<finding title>",
      "recommendation": "<actionable recommendation>",
      "recommendation_options": [
        { "label": "A", "body": "<option A body>" },
        { "label": "B", "body": "<option B body>" }
      ]
    }
  ],
  "stale_suppressed": [
    {
      "pass": "<pass id>",
      "title": "<original finding title>",
      "evidence": "<grep output or commit hash showing why it's stale>"
    }
  ],
  "iteration_note": "<one-sentence note or null>"
}
```

## Field notes

- **`iteration`** is per-mode: proposal-mode and system-mode iterations count separately on the same change.
- **`passes_run` / `passes_skipped`** are strings (pass IDs); skip reasons go in the report body but not the summary array.
- **`findings_summary.by_pass`** mirrors the report's per-pass grouping for quick downstream parsing.
- **`stale_suppressed`** captures findings that pushback removed; they don't appear in the user-facing report but do persist here for audit.
- **`final_assessment`** is one of the stock phrasings (mode-specific gate text); see the final-assessment table in SKILL.md.
- **`iteration_note`** is the one-line "Note: N of these findings appeared in the last run" comparison; `null` when this is the first run for the mode.
- **`recommendation_options`** is OPTIONAL on each finding. Present only when the finding's recommendation is genuinely disjunctive — multiple defensible paths the user must choose between. Producer-side contract (per `orbit-review` spec's `Optional recommendation_options field on finding entries`): MUST contain ≥ 2 entries; each entry MUST have non-empty `label` (typically `"A"`, `"B"`, `"C"`, … or `"1"`, `"2"`, …) and non-empty `body`. The prose `recommendation` field still summarizes the disjunction for human readers. Consumer (address-reviews) uses this for the structured decision-fork detection path; absence triggers heuristic fallback. Omit the field for single-recommendation findings.
