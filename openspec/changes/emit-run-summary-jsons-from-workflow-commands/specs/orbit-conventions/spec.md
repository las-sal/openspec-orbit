## MODIFIED Requirements

### Requirement: Internal-run JSON summary format

The system SHALL use a consistent JSON shape for run-summary emissions across **all orbit commands that mutate artifacts** — workflow, editorial, and lifecycle alike. Every run-summary JSON SHALL include a universal spine; commands extend the spine with per-kind and per-command extensions.

The universal spine (6 required fields):

```
command          string       identifies which command emitted (matches filename prefix)
timestamp        string       ISO-8601 UTC, format YYYY-MM-DDTHH-MM-SSZ, also embedded in filename
change           string|null  the change name (or null for project-scope commands)
final_assessment string       narrative of what just happened (human-readable)
next_recommended string       verbatim recommendation, suitable for orbit-status best-effort parse
kind             enum         "workflow" | "editorial" | "lifecycle"
```

Per-kind extensions:

- **`kind: "workflow"`** — emitted by `explore`, `propose`, `new`, `continue`, `ff-change`, `apply`, `verify`. Per-command extensions are defined in the `orbit-run-summary-emit` capability (e.g., `apply.chunk_complete`, `verify.verdict`, `explore.decisions_captured`).
- **`kind: "editorial"`** — emitted by `review`, `address-reviews`, `audit-drift`, `review-external`. Per-command extensions include: `iteration` (when applicable to the command — e.g., review iter-N, address-reviews iter-N), `findings_summary` (counts by severity, optionally by pass/category), `finding_titles` (array of brief titles), plus command-specific fields defined in per-skill schema references at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`.
- **`kind: "lifecycle"`** — emitted by `archive` only. Per-command extensions include: `archive_path`, `audit`, `sync_specs`, `unresolved_markers`, `user_decision`, plus other fields defined in the archive skill.

#### Scenario: Universal spine present on every emit

- **WHEN** any orbit command (workflow, editorial, or lifecycle) writes a run-summary JSON
- **THEN** the JSON contains the 6 spine fields with valid values: `command`, `timestamp`, `change` (or null), `final_assessment`, `next_recommended`, and `kind`

#### Scenario: Workflow command emit shape

- **WHEN** a workflow command (e.g., `/opsx:propose`, `/opsx:apply`, `/opsx:verify`) writes its JSON
- **THEN** `kind` equals `"workflow"`; per-command extensions are present per the `orbit-run-summary-emit` capability (which defines the specific extension fields per workflow command)

#### Scenario: Editorial command emit shape

- **WHEN** an editorial command (e.g., `/opsx:review`, `/opsx:address-reviews`, `/opsx:audit-drift`, `/opsx:review-external`) writes its JSON
- **THEN** `kind` equals `"editorial"`; per-command extensions include `iteration` (when the command tracks iterations), `findings_summary` (counts by severity), `finding_titles` (array of brief titles), plus command-specific fields documented at `.claude/skills/openspec-<skill>/references/run-summary-schema.md`

#### Scenario: Lifecycle command emit shape

- **WHEN** `/opsx:archive` writes its JSON
- **THEN** `kind` equals `"lifecycle"`; per-command extensions include `archive_path`, `audit`, `sync_specs`, `unresolved_markers`, `user_decision` per the archive skill's emit

#### Scenario: orbit-status tier-1 reader parses next_recommended uniformly across kinds

- **WHEN** orbit-status reads any run-summary JSON (regardless of `kind`) and best-effort parses `next_recommended` per `orbit-status-recommendation/spec.md:7`
- **THEN** the leading `/opsx:<verb> [args]` token (if present) is extracted into `command` and `args`; on parse failure (e.g., prose recommendation), the full string is preserved in `reason`

#### Scenario: Future consumers route by `kind` field rather than parsing filename prefix

- **WHEN** a downstream consumer (dashboard, CI bot, IDE plugin) reads `.orbit-runs/*.json` files
- **THEN** the consumer MAY route by reading the `kind` field directly rather than pattern-matching filename prefixes (e.g., `review-*`, `address-reviews-*`); filename prefix routing remains valid as a fallback
