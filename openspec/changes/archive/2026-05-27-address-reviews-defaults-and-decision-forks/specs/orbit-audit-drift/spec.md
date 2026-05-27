## ADDED Requirements

### Requirement: Optional `recommendation_options` field on audit-drift finding entries

The system SHALL emit audit-drift findings whose `recommendation` is genuinely disjunctive — offering the user a choice between two or more concrete remediation paths — with an additional optional structured field `recommendation_options: [{label, body}]` alongside the existing prose `recommendation` field. This field is the producer-side affordance for address-reviews' structured decision-fork detection (per `orbit-address-reviews` `Disjunctive recommendation fields surface as decision forks`).

**Field shape**: identical to `orbit-review`'s `Optional recommendation_options field on finding entries` requirement. The shape is shared across both producers so the consumer (address-reviews) reads it uniformly regardless of which command produced the finding.

```
recommendation_options: [
  { "label": "A", "body": "<concrete option 1 text>" },
  { "label": "B", "body": "<concrete option 2 text>" },
  ...
]
```

**When to emit** in audit-drift specifically:

- Category 1 (Vocabulary residue): typically single-recommendation ("delta this file" or "apply a hotfix commit"). Emit `recommendation_options` only when both options are equally defensible and which to pick depends on the user's context (e.g., "Either (A) delta this file in a future change, or (B) apply a hotfix commit now").
- Category 2 (Lens staleness): typically disjunctive ("Either (A) rename the surface ref, or (B) remove the perspective if obsolete"). Emit `recommendation_options` when the choice between options is genuinely the user's to make.
- Category 3 (Cross-doc consistency): often disjunctive when two docs disagree ("Either (A) update doc X to match doc Y, or (B) update doc Y to match doc X — depending on which is canonical"). Emit `recommendation_options` for these.
- Category 4 (Archive coherence): typically requires running `sync-specs` or applying a hotfix. Emit only when multiple resolution paths are equally defensible.

**Backward compatibility**: optional on every finding; absence means single-recommendation. Existing parsers SHALL ignore the field without erroring.

#### Scenario: Category 2 lens-staleness finding with rename-or-remove options

- **WHEN** audit-drift Category 2 identifies a perspective referencing a capability that no longer exists under that name (potentially renamed)
- **THEN** the finding's JSON entry includes `recommendation_options: [{label: "A", body: "rename the surface ref to match the current capability name <name>"}, {label: "B", body: "remove the perspective if it's obsolete"}]` AND the prose `recommendation` field summarizes the disjunction

#### Scenario: Category 1 single-recommendation residue emit omits options

- **WHEN** audit-drift Category 1 identifies vocabulary residue in a baseline spec and the obvious resolution is to delta the file in a future change (no equally defensible alternative)
- **THEN** the finding's JSON entry contains the prose `recommendation` field but does NOT include `recommendation_options`

#### Scenario: Category 3 disagree-which-is-canonical finding emits options

- **WHEN** audit-drift Category 3 identifies two governing docs disagreeing on a fact, and which doc is canonical depends on user context
- **THEN** the finding's JSON entry includes `recommendation_options` enumerating "update X to match Y" and "update Y to match X" as separate paths

#### Scenario: Field is consumer-shared across orbit-review and orbit-audit-drift

- **WHEN** address-reviews ingests an audit-drift JSON via `--from-file` or auto-discovery
- **THEN** the parser handles `recommendation_options` on audit-drift findings using the same logic it applies to orbit-review findings — uniform consumer-side behavior across both producers

#### Scenario: Pre-archive context preserves the field

- **WHEN** audit-drift runs in `pre-archive` context (invoked by `/opsx:archive`) and produces findings with `recommendation_options`
- **THEN** the field is preserved in the audit-drift JSON written to `.orbit-runs/audit-drift-<TS>.json`; the archive flow's prompt to the user (per `openspec-archive-change` `## NEW Step 3.5`) surfaces the disjunction directly to the user as a fork rather than collapsing to a single line

#### Scenario: Producer-side enforcement — minimum 2 entries

- **WHEN** an audit-drift category pass (C1-C4) would emit a finding with `recommendation_options` containing fewer than 2 entries
- **THEN** the producer SHALL omit the field entirely (single-recommendation findings use prose `recommendation` only); emitting a 0- or 1-entry `recommendation_options` array is a producer bug. Equivalent to the orbit-review producer-side enforcement scenario.

#### Scenario: Producer-side enforcement — every entry has required label and body fields

- **WHEN** an audit-drift category pass would emit a finding with `recommendation_options` where any entry has missing or empty `label` or `body`
- **THEN** the producer SHALL either populate the field with a meaningful value, drop the malformed entry (and re-check the ≥ 2-entry rule), or refuse to emit. Equivalent to the orbit-review producer-side enforcement scenario for entry shape.
