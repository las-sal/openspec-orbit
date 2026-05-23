## MODIFIED Requirements

### Requirement: Distribution model — overlay, not CLI fork

The system SHALL be distributed as a `.claude/` overlay pegged to a specific upstream `@fission-ai/openspec` version, not as a fork of the upstream OpenSpec CLI. Orbit owns the `.claude/` surface (skills, commands, supporting docs); upstream supplies the CLI binary as a pinned engine.

#### Scenario: Overlay shape

- **WHEN** orbit ships
- **THEN** the deliverable is markdown content under `.claude/commands/opsx/` and `.claude/skills/openspec-*/`, plus supporting docs; the upstream `@fission-ai/openspec` CLI binary is unchanged

#### Scenario: Consumers install after openspec init at pinned version

- **WHEN** a user adopts orbit in a project
- **THEN** they first install upstream `@fission-ai/openspec` at orbit's pinned version (see `Upstream version pinning` requirement), run upstream `openspec init` to scaffold the project's `.claude/` and `openspec/` directories, then drop orbit's overrides + new files into the same locations; orbit does not include or replace upstream's CLI

#### Scenario: Pinned upstream version is contractual

- **WHEN** orbit is in use against an upstream version other than the pinned one
- **THEN** orbit's behavior is undefined; users are responsible for verifying their installed upstream version matches orbit's pin (per `Upstream version pinning` requirement)

#### Scenario: Overlay-overwrite behavior is intentional under pegging

- **WHEN** orbit's overlay copies files into `.claude/skills/` or `.claude/commands/opsx/` that share names with upstream-`init`-installed files
- **THEN** the overwrite is the intended distribution mechanism; under pegging there is no "fresh upstream copy to preserve" — the pinned upstream version produces deterministic files, and orbit's overrides are the orbit-owned versions for that pinned upstream

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
- **`kind: "lifecycle"`** — emitted by `archive` only. Per-command extensions include: `archive_path`, `audit`, `sync_specs` (sync results captured from orbit's archive flow invoking the `openspec-sync-specs` upstream primitive; the primitive is retained under pegging strategy per orbit-conventions `Distribution model — overlay, not CLI fork`), `unresolved_markers`, `user_decision`, plus other fields defined in the archive skill.

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

## ADDED Requirements

### Requirement: Upstream version pinning

The system SHALL declare a specific pinned upstream `@fission-ai/openspec` version that orbit is tested against and contractually depends on. Orbit's behavior outside the pinned version is undefined. Version upgrade is a deliberate change-proposal event, not an automatic ingest.

#### Scenario: Pinned version declared in CLAUDE.md

- **WHEN** a user or AI reads `CLAUDE.md` at the orbit repo root
- **THEN** the pinned upstream version is clearly stated in the introductory or installation context

#### Scenario: Pinned version declared in README

- **WHEN** a user reads the orbit `README.md`
- **THEN** the install section names the exact pinned version (e.g., `@fission-ai/openspec@1.3.1`) as a required prerequisite

#### Scenario: Doc-only enforcement is acceptable

- **WHEN** orbit ships without an install script (current state)
- **THEN** documentation-only declaration is sufficient enforcement; install-time version-check is tracked as a separate future enhancement (orbit issue #26)

#### Scenario: Version upgrade is a deliberate change

- **WHEN** orbit's contributors consider upgrading the pinned upstream version
- **THEN** the upgrade is proposed as its own OpenSpec change (with explicit upgrade-and-port scope), not performed as silent maintenance; the new pinned version replaces the prior one in CLAUDE.md + README + this requirement's documented value

#### Scenario: Upstream improvements do not auto-propagate

- **WHEN** upstream releases `@fission-ai/openspec` v1.3.2, v1.4.0, or later while orbit is pinned to an earlier version
- **THEN** orbit users do not receive those improvements until a deliberate upgrade change lands in orbit; this is an accepted trade-off (per design.md D-arch-1)

### Requirement: Overlay file disposition

The system SHALL classify every file orbit ships in `.claude/` into one of four disposition categories. Each category dictates the file's lifecycle in orbit's overlay.

#### Scenario: Orbit-authored — full ownership

- **WHEN** orbit ships a file with no upstream-derived content (e.g., `openspec-review`, `openspec-audit-drift`, `openspec-review-external`, `openspec-address-reviews`, the orbit-authored `openspec-onboard`)
- **THEN** the file is fully owned by orbit; orbit's contributors edit it freely; the file MAY use the `openspec-*` directory naming convention for consistency with siblings even when the body is 100% orbit-authored

#### Scenario: Orbit-modified — upstream body with `# Orbit additions`

- **WHEN** orbit ships a file that bundles upstream's body with an appended `# Orbit additions` section (e.g., `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`)
- **THEN** the upstream body is the pinned-version snapshot; orbit's additions append behavior or constraints; this pattern is transitional and a subject of future Option-2 work (dropping the additions pattern in favor of fully orbit-authored versions of these skills)

#### Scenario: Upstream-required primitive — kept verbatim

- **WHEN** orbit depends on an upstream skill as a callable primitive (e.g., `openspec-sync-specs`, which orbit's archive flow invokes via subagent for delta-spec sync)
- **THEN** orbit ships the upstream skill unchanged; orbit's use of the primitive matches upstream's internal use pattern; no `# Orbit additions` are needed because orbit's interaction is at the invocation layer, not the skill body

#### Scenario: Not shipped — pruned from overlay

- **WHEN** an upstream skill is truly unmodified, not used by orbit, and provides no orbit-mission value (e.g., `feedback`, which sends user feedback to Fission-AI and has no role in orbit's pegged workflow)
- **THEN** orbit does NOT ship the file in its overlay; the skill is simply absent from orbit's `.claude/skills/` directory. NOTE: because the overlay install model is `cp -r` (which does not delete target-only files), absence-in-overlay is necessary but not sufficient to ensure absence-in-user-project. The README install/update sections SHALL document explicit `rm` commands for pruned files so user-project state matches overlay intent. See orbit-onboard `Setup verification section` for the verify-absence scenarios users invoke to check.

#### Scenario: Per-skill disposition is documented

- **WHEN** orbit's contributors evaluate whether to ship a new upstream skill that lands in a future pinned-version upgrade
- **THEN** the disposition decision is recorded in the change proposal that performs the upgrade; the new skill is assigned to one of the four categories with rationale

#### Scenario: Commands follow the same 4-category framework

- **WHEN** orbit ships (or chooses not to ship) a slash-command file in `.claude/commands/opsx/`
- **THEN** the file's disposition is one of the four categories on the same criteria as skills: orbit-authored (e.g., `opsx/review.md`, `opsx/audit-drift.md`, `opsx/review-external.md`, `opsx/address-reviews.md`); orbit-modified (mirrors an upstream-bound capability with orbit-specific behavior, e.g., `opsx/propose.md`, `opsx/explore.md`, `opsx/archive.md`, `opsx/apply.md`, `opsx/verify.md`, `opsx/continue.md`, `opsx/fast-forward.md`, `opsx/new.md`, `opsx/bulk-archive.md`); upstream-required primitive (none currently — orbit does not depend on any upstream command file as a callable primitive); not shipped (e.g., `opsx/sync.md` — corresponds to the `openspec-sync-specs` SKILL that orbit retains as a primitive, but the user-callable command is not part of orbit's user surface under pegging)

#### Scenario: Command-file disposition documented in the same change as a removal or addition

- **WHEN** orbit decides a command should be added to the overlay or pruned from it
- **THEN** the decision is recorded in the change proposal that performs the addition/removal with the disposition category cited; pruned commands must include `rm` step in README install/update documentation (analogous to pruned-skill handling)
