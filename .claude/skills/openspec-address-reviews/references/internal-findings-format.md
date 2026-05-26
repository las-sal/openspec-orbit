# Reference: internal-findings file format (for `--from-file` JSON parsing)

The `--from-file <path>` flag accepts TWO format families: (a) external-review markdown (see `external-findings-format.md`), (b) internal review JSON (this file). The parser auto-detects format via content sniff: leading `{` routes to JSON; leading `# External Review:` routes to markdown; anything else triggers a format-mismatch error.

This format MUST match what `/opsx:review` (both `--as proposal` and `--as system`) writes to `openspec/changes/<name>/.orbit-runs/review-<mode>-<TS>.json`. The full JSON schema lives at `.claude/skills/openspec-review/references/run-summary-schema.md`; this file documents the `--from-file` parser's subset of that schema (the fields it reads to construct virtual markers).

## Expected file format

```json
{
  "command": "review",
  "timestamp": "<ISO-8601>",
  "change": "<change-name>",
  "mode": "proposal" | "system",
  "iteration": <integer>,
  "findings_summary": {
    "critical": <int>,
    "warning": <int>,
    "suggestion": <int>
  },
  "findings": [
    {
      "pass": "<pass id>",
      "severity": "CRITICAL" | "WARNING" | "SUGGESTION",
      "file": "<path>",
      "line": <integer>,
      "title": "<finding title>",
      "recommendation": "<actionable recommendation>"
    }
  ]
}
```

Other top-level fields (`final_assessment`, `next_recommended`, `kind`, `depth`, `flags`, `passes_run`, `passes_skipped`, `stale_suppressed`, `iteration_note`) MAY be present and are IGNORED by the parser — they're only consumed by other tooling (e.g., orbit-status).

## Parser contract

For each entry in the JSON's `findings[]` array, construct a virtual marker with:

| Virtual marker field | Source JSON field |
|---|---|
| `severity` | The entry's `severity` field (`CRITICAL` / `WARNING` / `SUGGESTION`) |
| `title` | The entry's `title` field |
| `file:line` | The entry's `file` + `line` fields joined as `<file>:<line>` |
| `description` | The entry's `recommendation` field |
| `source` | Always `internal-review` (vs `external` for markdown, `inline` for grep-found) |

Virtual markers walk the same lifecycle as inline markers, with one exception: **the marker-removal step (Step 3d in the SKILL.md walk) is a no-op** — there's no source-file marker text to delete. (Same behavior as external-markdown virtual markers; the no-op is shared across all virtual-marker provenance.)

### Command field discriminator

The parser MUST check the top-level `command` field after JSON parse succeeds:

- `command: "review"` → proceed with `findings[]` extraction.
- Any other `command` value (e.g., `"audit-drift"`, `"address-reviews"`, `"apply"`, `"archive"`, `"propose"`) → emit a clean error message naming the supported `command` value (`"review"`) and the observed value, then exit without acting on any findings.

V1 supports `review-<mode>-*.json` only. Other internal JSONs (audit-drift findings, address-reviews logs) have different walk semantics — they're out of scope for v1 and rejected explicitly rather than silently mis-parsed.

### Fresh pushback applies to JSON virtual markers

The JSON file's own `stale_suppressed[]` array already filtered stale findings at review time. **Fresh pushback against current state IS still applied** when address-reviews ingests the JSON: state may have changed between the review and the resolve (e.g., user fixed an issue ad-hoc; another commit landed). The SKILL.md walk step (3a — Apply pushback) executes for each JSON-virtual marker the same way it would for an inline marker or external-markdown virtual marker.

This is intentional double-pushback: the JSON-side filter is a subset of staleness; the lifecycle-side check is a superset.

## Malformed input handling

If the file is JSON but malformed in any of these ways, the parser MUST refuse to act on partial input:

- **JSON parse failure** (file looks like JSON but invalid syntax) → emit a parse-error message naming the file + the JSON parse-error position (best-effort), then exit. User fixes the file and re-runs.
- **Missing `command` field** OR `command` value other than `"review"` → emit the unsupported-command error (see "Command field discriminator" above).
- **Missing `findings[]` field** OR `findings` present but not an array → emit a clean error naming the missing/malformed `findings[]` requirement; reference this file for the expected shape; exit.
- **Empty `findings: []`** → NOT an error. Succeed with zero virtual markers; the resolution log reports a clean empty walk (`✓ Resolved: 0, ⚠ Stale: 0, ⏸ Deferred: 0, ✗ Escalated: 0`) with a one-line note `Source JSON had no findings to walk; resolution log is informational.` A clean review with `findings: []` is the expected state for an all-clear pass — feeding it through `--from-file` is a valid no-op invocation.

## Tolerated variations

The parser SHOULD be lenient on:

- **Missing optional `line` field** on a finding → construct the virtual marker with `file` only (no line component); resolution log shows the file path without `:line`. Some findings are file-scoped rather than line-scoped (e.g., "this file needs a top-level section added").
- **Missing optional `pass` field** → virtual marker's `pass` provenance is reported as `unknown` in the resolution log; lifecycle proceeds normally.
- **Extra top-level JSON fields** (e.g., `dogfood_note`, `convergence_state`, `flags`, `iteration_note`) → ignored; not consumed by the parser. These exist for other tooling and don't affect virtual-marker construction.
- **Mixed-case `severity`** values (`Critical`, `warning`, etc.) → normalize to upper-case before comparison.

The parser MUST be strict on:

- **Top-level `command` field** — must be exactly `"review"` (case-sensitive) for v1.
- **Per-finding `severity` field** — must be one of `CRITICAL`, `WARNING`, `SUGGESTION` after case normalization.
- **Per-finding `title` field** — must be a non-empty string.
- **Per-finding `file` field** — must be a non-empty string (the file path).
- **Per-finding `recommendation` field** — must be a non-empty string (becomes the virtual marker's description).

The reason for the strict/lenient split: the strict items are what the orbit run-summary schema guarantees by spec (per `orbit-conventions`'s `Internal-run JSON summary format` requirement + `orbit-review`'s per-command extensions); the lenient items are best-effort fields that downstream tools handle but the parser doesn't strictly require.

## Quick worked example of valid input

```json
{
  "command": "review",
  "timestamp": "2026-05-26T20:01:08Z",
  "change": "harden-review-mode-recommendations",
  "mode": "system",
  "iteration": 2,
  "findings_summary": {
    "critical": 0,
    "warning": 0,
    "suggestion": 1
  },
  "findings": [
    {
      "pass": "2",
      "severity": "SUGGESTION",
      "file": ".claude/skills/openspec-review/references/run-summary-schema.md",
      "line": 22,
      "title": "Schema reference does not document new convergence fields emitted by iteration-aware logic",
      "recommendation": "Out of scope for this change. Track as a follow-up: future change touching the run-summary contract should add convergence_state / convergence_state_label / convergence_inputs (plus skip_reasons and dogfood_note) to references/run-summary-schema.md."
    }
  ]
}
```

This example has 0 CRITICAL, 0 WARNING, 1 SUGGESTION. Ingesting it via `--from-file` produces one virtual marker:

- `severity`: `SUGGESTION`
- `title`: `Schema reference does not document new convergence fields emitted by iteration-aware logic`
- `file:line`: `.claude/skills/openspec-review/references/run-summary-schema.md:22`
- `description`: (the `recommendation` text)
- `source`: `internal-review`

The lifecycle walks this marker: pushback verifies the gap still applies → classify as `decision required` (out-of-scope/defer or include-in-current-change) → user makes the call → log entry written. Marker removal is no-op (no source file to delete from).
