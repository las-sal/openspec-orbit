## MODIFIED Requirements

### Requirement: Internal-run JSON summary format

The system SHALL use a consistent JSON shape for run-summary emissions across **all orbit commands that mutate artifacts** — workflow, editorial, and lifecycle alike. Every run-summary JSON SHALL include a universal spine; commands extend the spine with per-kind and per-command extensions.

The universal spine (6 required fields):

```
command          string       identifies which command emitted (matches filename prefix)
timestamp        string       ISO-8601 UTC; JSON field uses standard colon format `YYYY-MM-DDTHH:MM:SSZ`
                              (e.g., `"2026-05-21T13:34:12Z"`). Filename embeds a colon-replaced `<TS>` token
                              `YYYY-MM-DDTHH-MM-SSZ` (e.g., `propose-2026-05-21T13-34-12Z.json`) because
                              colons aren't filesystem-safe across all platforms.
change           string|null  the change name (or null for project-scope commands)
final_assessment string       narrative of what just happened (human-readable)
next_recommended string       verbatim recommendation, suitable for orbit-status best-effort parse
kind             enum         "workflow" | "editorial" | "lifecycle"
```

Per-kind extensions:

- **`kind: "workflow"`** — emitted by `explore`, `propose`, `new`, `continue`, `ff`, `apply`, `verify`. Per-command extensions are defined in the `orbit-run-summary-emit` capability (e.g., `apply.chunk_complete`, `verify.verdict`, `explore.decisions_captured`).
- **`kind: "editorial"`** — emitted by `review`, `address-reviews`, `audit-drift`, `review-external`. Per-command extensions include: `iteration` (when applicable to the command — e.g., review iter-N, address-reviews iter-N), `findings_summary` (counts by severity; included when findings are present — i.e., review/address-reviews/audit-drift completion emits; review-external at T0 emits before external findings return and SHALL omit this field), `finding_titles` (array of brief titles; included with `findings_summary`, omitted in the same cases), plus command-specific fields defined in per-skill schema references at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`.
- **`kind: "lifecycle"`** — emitted by `archive` only. Per-command extensions include: `archive_path`, `audit`, `sync_specs` (transitional — persists from pre-#6 architecture when `/opsx:sync-specs` was a separate command; openspec-orbit#6 will deprecate/remove `/opsx:sync-specs` entirely, at which point this field will be removed or repurposed in a follow-up change), `unresolved_markers`, `user_decision`, plus other fields defined in the archive skill.

**Canonical examples** (one per kind, illustrating spine + per-kind extensions):

Workflow-kind example — `apply-2026-05-21T13-34-12Z.json` written at chunk 2 completion of a chunked apply:

```json
{
  "command": "apply",
  "timestamp": "2026-05-21T13:34:12Z",
  "change": "add-detail-flag",
  "final_assessment": "Completed chunk 2 of 5 (inventory+parsing); 28 of 76 tasks done.",
  "next_recommended": "/opsx:apply add-detail-flag — next chunk: phase+attention+recommendation engine",
  "kind": "workflow",
  "tasks_completed": 28,
  "tasks_remaining": 48,
  "chunk": "2 of 5",
  "chunk_name": "inventory+parsing",
  "chunk_complete": true,
  "tasks_completed_this_session": 16
}
```

Editorial-kind example — `review-proposal-2026-05-21T00-18-14Z.json` written by `/opsx:review --as proposal` iter-1:

```json
{
  "command": "review",
  "timestamp": "2026-05-21T00:18:14Z",
  "change": "emit-run-summary-jsons-from-workflow-commands",
  "final_assessment": "5 CRITICAL findings + 7 WARNING + 7 SUGGESTION. Ready for /opsx:address-reviews.",
  "next_recommended": "/opsx:address-reviews emit-run-summary-jsons-from-workflow-commands --from-file <findings-bridge>",
  "kind": "editorial",
  "mode": "proposal",
  "iteration": 1,
  "depth": "full",
  "passes_run": ["1","2","3","4","5","6","7","8","9"],
  "findings_summary": {
    "critical": 5,
    "warning": 7,
    "suggestion": 7
  }
}
```

Lifecycle-kind example — `archive-2026-05-20T16-31-59Z.json` written after `/opsx:archive bootstrap-orbit-status-cli`:

```json
{
  "command": "archive",
  "timestamp": "2026-05-20T16:31:59Z",
  "change": "bootstrap-orbit-status-cli",
  "final_assessment": "Archived bootstrap-orbit-status-cli to openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/.",
  "next_recommended": "Change archived. Run /opsx:new or /opsx:explore to start the next change, or /opsx:audit-drift for a project-wide drift check.",
  "kind": "lifecycle",
  "archive_path": "openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/",
  "audit": {
    "ran": true,
    "findings_summary": { "critical": 0, "warning": 0, "suggestion": 0 }
  },
  "user_decision": "proceeded_with_no_critical",
  "warnings": []
}
```

#### Scenario: Universal spine present on every emit

- **WHEN** any orbit command (workflow, editorial, or lifecycle) writes a run-summary JSON
- **THEN** the JSON contains the 6 spine fields with valid values: `command`, `timestamp`, `change` (or null), `final_assessment`, `next_recommended`, and `kind`

#### Scenario: Workflow command emit shape

- **WHEN** a workflow command (e.g., `/opsx:propose`, `/opsx:apply`, `/opsx:verify`) writes its JSON
- **THEN** `kind` equals `"workflow"`; per-command extensions are present per the `orbit-run-summary-emit` capability (which defines the specific extension fields per workflow command)

#### Scenario: Editorial command emit shape (with findings)

- **WHEN** an editorial command that has produced findings (`/opsx:review`, `/opsx:address-reviews`, `/opsx:audit-drift`) writes its JSON
- **THEN** `kind` equals `"editorial"`; per-command extensions include `iteration` (when the command tracks iterations), `findings_summary` (counts by severity), `finding_titles` (array of brief titles), plus command-specific fields documented at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`

#### Scenario: Editorial command emit shape (pre-findings T0 case)

- **WHEN** `/opsx:review-external` writes its T0 JSON (prompt packaged, external findings not yet returned)
- **THEN** `kind` equals `"editorial"`; per-command extensions include `mode`, `prompt_path`, `target`, `awaiting_findings: true` — and OMIT `findings_summary` and `finding_titles` because no findings exist yet

#### Scenario: Lifecycle command emit shape

- **WHEN** `/opsx:archive` writes its JSON
- **THEN** `kind` equals `"lifecycle"`; per-command extensions include `archive_path`, `audit`, `sync_specs`, `unresolved_markers`, `user_decision` per the archive skill's emit

#### Scenario: orbit-status tier-1 reader parses next_recommended uniformly across kinds

- **WHEN** orbit-status reads any run-summary JSON (regardless of `kind`) and best-effort parses `next_recommended` per `orbit-status-recommendation/spec.md:7`
- **THEN** the leading `/opsx:<verb> [args]` token (if present) is extracted into `command` and `args`; on parse failure (e.g., prose recommendation), the full string is preserved in `reason`

#### Scenario: Future consumers route by `kind` field rather than parsing filename prefix

- **WHEN** a downstream consumer (dashboard, CI bot, IDE plugin) reads `.orbit-runs/*.json` files
- **THEN** the consumer MAY route by reading the `kind` field directly rather than pattern-matching filename prefixes (e.g., `review-*`, `address-reviews-*`); filename prefix routing remains valid as a fallback

### Requirement: `.orbit-runs/` per-change persistence

The system SHALL use one of the following `.orbit-runs/` persistence locations, scoped by the type of work being persisted:

- **`openspec/changes/<change-name>/.orbit-runs/`** — per-change iteration history for active changes. Used by editorial commands (review, address-reviews, audit-drift inline), workflow commands operating on a named change (propose, new, continue, ff, apply, verify), lifecycle commands (archive), and review-external T0. The primary persistence location.
- **`openspec/explore/<name>/.orbit-runs/`** — per-exploration iteration history for named-mode `/opsx:explore` sessions BEFORE the exploration is promoted to a formal change via `/opsx:propose`. Moves into `openspec/changes/<name>/.orbit-runs/` when `/opsx:propose <name>` consumes the staging directory (per the orbit-propose consume-mode convention).
- **`openspec/.orbit-runs/`** — project-scope iteration history for commands that have no change scope (currently: `/opsx:audit-drift` invoked with no `<change-name>` argument, i.e., project-wide standalone). Created if it doesn't exist.

In all three locations, files are committed (the directory SHOULD NOT be in `.gitignore`) and follow the same naming pattern (`<command>-<ISO-timestamp>.json` for internal-run JSONs; `external-<as>-<ISO-timestamp>.md` for external-review files).

#### Scenario: Active-change emit goes to changes/<name>/.orbit-runs/

- **WHEN** orbit writes a run summary for a named, active change (e.g., `/opsx:review foo`, `/opsx:apply foo`, `/opsx:archive foo`)
- **THEN** the file lands at `openspec/changes/foo/.orbit-runs/<command>-<TS>.json`

#### Scenario: Named-mode exploration emit goes to explore/<name>/.orbit-runs/

- **WHEN** `/opsx:explore foo` (named mode) writes its run-summary JSON at a conversation boundary
- **THEN** the file lands at `openspec/explore/foo/.orbit-runs/explore-<TS>.json` (NOT `openspec/changes/foo/.orbit-runs/`, since `foo` is still in the staging area pre-`/opsx:propose`)

#### Scenario: Project-wide standalone emit goes to openspec/.orbit-runs/

- **WHEN** `/opsx:audit-drift` runs with no change argument and writes its run-summary JSON
- **THEN** the file lands at `openspec/.orbit-runs/audit-drift-<TS>.json` (create the directory if it doesn't exist); the JSON's `change` field is `null`

#### Scenario: Exploration .orbit-runs/ travels with promotion to changes/

- **WHEN** `/opsx:propose foo` (consume mode) moves the staging directory `openspec/explore/foo/` into `openspec/changes/foo/`
- **THEN** the contents of `openspec/explore/foo/.orbit-runs/` (including any prior `explore-<TS>.json` emits) are preserved and now live at `openspec/changes/foo/.orbit-runs/`; subsequent emits go to the new location

#### Scenario: Committed, not gitignored (all three locations)

- **WHEN** orbit writes a run summary or external-review file to any of the three `.orbit-runs/` locations
- **THEN** the file is intended to be committed (none of the three directories should be in `.gitignore`); each represents real iteration history

#### Scenario: Internal run summary file name pattern

- **WHEN** a review, audit, address-reviews, archive, or workflow command writes a summary
- **THEN** the file is named `<command>-<ISO-timestamp>.json` (e.g., `review-proposal-2026-05-21T00-18-14Z.json`, `apply-2026-05-21T13-34-12Z.json`)

#### Scenario: External review file name pattern

- **WHEN** `/opsx:review-external` produces a prompt file (or external AI produces a findings file following the prompt)
- **THEN** the file is named `external-prompt-<as>-<ISO-timestamp>.md` for prompts and `external-<as>-<ISO-timestamp>.md` for findings (e.g., `external-prompt-proposal-2026-05-21T01-32-52Z.md`, `external-proposal-2026-05-21T01-39-56Z.md`)

#### Scenario: Travels with archive

- **WHEN** the change is archived via `/opsx:archive`
- **THEN** the `.orbit-runs/` directory at `openspec/changes/<name>/.orbit-runs/` moves to `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/` as part of the change content (the project-wide `openspec/.orbit-runs/` location is NOT moved — it's project-scope and persists across changes)
