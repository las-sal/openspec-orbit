## MODIFIED Requirements

### Requirement: Audit-drift standalone recommendations

Standalone `/opsx:audit-drift` already emits `audit-drift-<TS>.json` today per `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`. The emit SHALL include the universal spine from `orbit-conventions`'s `Internal-run JSON summary format` (with `kind: "editorial"`). The emit-layer recognizes TWO standalone invocation modes — **change-scoped** (`/opsx:audit-drift <name>`) and **project-wide** (`/opsx:audit-drift` with no argument) — each with distinct emit path, `change` value, and recommendation logic:

**Change-scoped standalone** (`/opsx:audit-drift <name>`):
- Emit path: `openspec/changes/<name>/.orbit-runs/audit-drift-<TS>.json`
- `change` field: the change name
- `next_recommended` on findings: `"/opsx:address-reviews <name> — N drift(s) detected; resolve before next workflow step"`. The `--from-file <this-json>` argument is NOT required: `/opsx:address-reviews` auto-discovers the most-recent `audit-drift-*.json` in the change's `.orbit-runs/` per the orbit-address-reviews `Address-reviews command available` requirement's auto-discovery fallback. Address-reviews picks the audit-drift JSON via the recency rule (single global most-recent across `review-<mode>-*.json` + `audit-drift-*.json`); the just-written `audit-drift-<TS>.json` will be the most recent at recommendation-emit time, so auto-discovery resolves to it cleanly.
- `next_recommended` on clean (zero findings): copy from the most recent prior `.orbit-runs/*.json` for the same change, excluding the just-written audit-drift JSON itself. `final_assessment` SHALL note "drift check clean; deferring to prior workflow state."

**Project-wide standalone** (`/opsx:audit-drift` with no argument):
- Emit path: `openspec/.orbit-runs/audit-drift-<TS>.json` (create the directory if it doesn't exist)
- `change` field: `null` (no change scope)
- `next_recommended` on findings: `"/opsx:address-reviews --from-file <this-json> — N drift(s) detected project-wide; resolve before next workflow step"`. The `--from-file <path>` argument IS required here: project-wide audit-drift has no change-directory anchor for `.orbit-runs/` lookup, so auto-discovery does NOT apply per the orbit-address-reviews spec's positional-resolution algorithm (no positional → bare invocation → grep-only; bare invocation skips auto-discovery).
- `next_recommended` on clean (zero findings): `"No project-wide drifts detected. Run /opsx:audit-drift periodically to catch new drift."` (no prior-workflow defer because project-wide audit-drift has no per-change workflow narrative to defer to). `final_assessment` SHALL note "project-wide drift check clean."

(Historical note: prior to the `address-reviews-auto-discovers-internal-json` change landing, the change-scoped recommendation also included `--from-file <this-json>` because address-reviews v1 did not auto-discover. Now that auto-discovery handles change-scoped invocations, the `--from-file` argument is dropped for change-scoped recommendations to give users the cleaner 2-command workflow. Project-wide invocations still require explicit `--from-file` because they have no change-directory anchor.)

Per-command extensions for audit-drift SHALL include:

```
categories_run        object   { vocab_residue, lens_staleness, cross_doc_consistency, archive_coherence } — bools
findings_by_category  object   counts per category
findings_total        int
```

This requirement applies only to **standalone** `/opsx:audit-drift` invocations (both change-scoped and project-wide). Inline audit-drift during `/opsx:archive` is captured in `archive-<TS>.json` per the existing archive emit convention (unchanged).

#### Scenario: Change-scoped standalone audit-drift with findings recommends address-reviews

- **WHEN** `/opsx:audit-drift foo` runs and detects 3 drifts
- **THEN** `openspec/changes/foo/.orbit-runs/audit-drift-<TS>.json` is written with `change: "foo"`, `findings_total: 3`, and `next_recommended` beginning with `"/opsx:address-reviews foo —"` (no `--from-file` argument; auto-discovery picks up the just-written audit-drift JSON via the recency rule)

#### Scenario: Change-scoped clean audit-drift defers to prior workflow recommendation

- **WHEN** `/opsx:audit-drift foo` runs and detects 0 drifts, and the prior latest `.orbit-runs/*.json` for `foo` is `apply-2026-05-21T10-00-00Z.json` with `next_recommended: "/opsx:apply foo — chunk 3 of 5 done"`
- **THEN** the new `audit-drift-<TS>.json` has `findings_total: 0` and `next_recommended` equal to the verbatim string `"/opsx:apply foo — chunk 3 of 5 done"`, and `final_assessment` notes drift-check-clean + deferral

#### Scenario: Project-wide standalone audit-drift with findings

- **WHEN** `/opsx:audit-drift` runs with no change argument and detects 2 project-wide drifts
- **THEN** `openspec/.orbit-runs/audit-drift-<TS>.json` is written with `change: null`, `findings_total: 2`, and `next_recommended` beginning with `"/opsx:address-reviews --from-file"` (still requires `--from-file` because project-wide invocations have no change-directory anchor; auto-discovery does not apply to bare `/opsx:address-reviews` invocations)

#### Scenario: Project-wide clean audit-drift (no prior workflow to defer to)

- **WHEN** `/opsx:audit-drift` runs with no change argument and detects 0 drifts
- **THEN** `openspec/.orbit-runs/audit-drift-<TS>.json` is written with `change: null`, `findings_total: 0`, and `next_recommended` reads `"No project-wide drifts detected. Run /opsx:audit-drift periodically to catch new drift."`

#### Scenario: Inline audit-drift during archive unchanged

- **WHEN** `/opsx:archive foo` runs and the inline audit-drift step produces findings
- **THEN** the findings are captured in `archive-<TS>.json` per existing archive emit behavior; no separate `audit-drift-<TS>.json` is written for the inline pass
