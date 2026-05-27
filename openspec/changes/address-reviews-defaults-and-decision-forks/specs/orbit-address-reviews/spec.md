## MODIFIED Requirements

### Requirement: Discover → triage → walk → ripple flag → report lifecycle

The system SHALL execute five lifecycle stages: (1) discover markers (or load from file), (2) triage with user (numbered list, optional scoping), (3) walk each marker per-finding (walk-mode default; `--batch` opts INTO legacy batch-mode), (4) ripple cascade (auto-apply ripples to in-scope files by default; `--no-cascade` opts out), (5) report a resolution log.

**Walk-mode is the default**: each finding receives its own pushback → classify → fix → ripple-cascade → remove-marker cycle. Batch-mode (all findings resolved together before any ripple) is opt-in via the `--batch` flag or by verbal batch-intent in the user's invocation message (per `Walk-mode by default with --batch opt-in`).

**Cascade is the default**: when a finding resolves, ripple-flagged files are automatically updated unless they fall into one of four lifecycle-invariant OUT categories (audit trail, baseline specs, cross-change directories, safe-exclusions) — see `Ripple cascade by default`. `--no-cascade` reverts to v1 list-only behavior; the resolution log records ripple-flagged files in `flagged_not_applied` regardless of mode.

**Decision-fork detection** fires within Step 3's walk, after classify, before fix, and only when classify == "decision required" (per `Disjunctive recommendation fields surface as decision forks`).

#### Scenario: Triage scoping

- **WHEN** markers are discovered and presented as a numbered list
- **THEN** the user can specify subset scoping (e.g., "just 1-3") and only those markers are walked

#### Scenario: Walk-mode lifecycle order per finding

- **WHEN** the lifecycle runs in walk-mode (default) with multiple findings in scope
- **THEN** each finding completes its full inner cycle (pushback → classify → fix → ripple-cascade → remove-marker) before the next finding's pushback begins; the user can interrupt between findings, and the resolution log captures partial completion if interrupted

#### Scenario: Batch-mode lifecycle order

- **WHEN** the lifecycle runs in batch-mode (`--batch` flag or verbal trigger)
- **THEN** all findings complete pushback + classify + fix in one pass before any ripple-cascade fires; ripples are applied once at the end as a single aggregated step. The resolution log records `walk_mode: "batch"`.

### Requirement: Ripple cascade by default

**RENAMED FROM**: `Ripple flag without auto-cascade`

The system SHALL automatically apply ripple-affected edits to in-scope files when a finding resolves, unless `--no-cascade` is specified. Ripple-flagged files are always recorded in the resolution log; the difference between default and `--no-cascade` modes is whether the system edits them or merely lists them.

**Cascade scope** (normative — defines which files cascade refuses to edit; everything else ripple-flagged is fair game regardless of file extension):

The cascade IN set is defined by exclusion. The OUT list captures four lifecycle-invariant categories — each entry is OUT for a structural workflow reason, not for a "this might be unsafe" heuristic. Files NOT matching any OUT category are IN, regardless of extension (`.py`, `.swift`, `.c`, `.sh`, `.md`, configs, dotfiles — all eligible if ripple-flagged).

- **OUT — change-dir audit trail**: `openspec/changes/<name>/.orbit-runs/*` and `openspec/.orbit-runs/*`. Audit-trail files; editing would corrupt the workflow's own record. NEVER edited by cascade.
- **OUT — baseline specs**: `openspec/specs/<capability>/spec.md`. Baseline mutations flow through the proposal cycle (`/opsx:propose` → delta spec → `/opsx:archive` triggers `sync-specs` to propagate). Cascade refuses direct baseline edits; the appropriate action is to add a delta in the current change's `specs/<capability>/spec.md`.
- **OUT — cross-change directories**: `openspec/changes/<other-name>/*` (other active changes' directories). Change-isolation invariant; each change's authoring is its own context.
- **OUT — safe-exclusions**: `.git/`, `node_modules/`, `dist/`, `build/`. Universal "never edit anywhere" set.

**Mode invariants**:

- Default mode (cascade on): IN ripples are edited; OUT ripples are recorded in `flagged_not_applied` with a reason naming the OUT category (preserving the v1 ripple-flag audit trail for files cascade refused to touch).
- `--no-cascade` mode: NO ripples are edited regardless of IN/OUT classification; all ripple-flagged files are recorded in `flagged_not_applied` with reason `--no-cascade suppressed`.
- Both modes: cascaded files are recorded in `ripple_cascade.applied`; flagged-not-applied files are recorded in `ripple_cascade.flagged_not_applied`.

**Cascade trusts the ripple-flag analysis**: the cascade step does NOT make independent "should this file be touched?" judgments beyond the four-category OUT check. Its job is to apply the fix to ripple-flagged files in the IN set. The contextual scope judgment ("is this file relevant to my finding?") lives in the earlier ripple-flag step, where the AI derives which files a fix affects. If ripple-flag analysis didn't surface a file, cascade doesn't touch it.

#### Scenario: Cascade applies to in-scope change-dir markdown

- **WHEN** a finding resolves and its ripple-flag set includes `openspec/changes/<name>/design.md` (a change-dir markdown file)
- **THEN** the cascade edits `design.md` consistent with the primary fix; the resolution log records the path under `ripple_cascade.applied`

#### Scenario: Cascade applies to non-markdown project files

- **WHEN** a finding resolves and its ripple-flag set includes a non-markdown project file (e.g., `src/handlers.py`, `bin/deploy.sh`, `Dockerfile`, `package.json`, a Swift / Go / C source file, or any dotfile in the project root)
- **THEN** the cascade edits the file consistent with the primary fix; the resolution log records the path under `ripple_cascade.applied`. File extension is NOT a discriminator — cascade applies the same pushback + classify + fix lifecycle to any file in the IN set. Pushback discipline + decision-fork prompts handle ambiguous edits regardless of file type.

#### Scenario: Cascade applies to orbit framework files when developing orbit itself

- **WHEN** the current repo IS the orbit framework's source (i.e., orbit develops itself; the change's content includes orbit's own `.claude/skills/`, `.claude/commands/`, README, etc.) AND a finding's ripple-flag set includes one of these files
- **THEN** the cascade edits the file consistent with the primary fix; the resolution log records the path under `ripple_cascade.applied`. Orbit's framework files have no special status when they ARE the project being developed — they're in scope because they're not in any OUT category. The mutation-via-delta rule applies only to baseline `openspec/specs/<capability>/spec.md`, not to the framework's behavioral surface (skill prompts, command mirrors, prose docs).

#### Scenario: Cascade skips .orbit-runs/ audit trail

- **WHEN** a finding's ripple-flag set includes any path under `openspec/changes/<name>/.orbit-runs/` or `openspec/.orbit-runs/`
- **THEN** the cascade SKIPS that file (it's audit-trail; editing would corrupt the workflow's own record); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `audit-trail file; cascade skipped by policy`

#### Scenario: Cascade skips baseline openspec/specs/ — guides user to add a delta

- **WHEN** a finding's ripple-flag set includes any path under `openspec/specs/<capability>/`
- **THEN** the cascade SKIPS that file (baseline mutations flow through the proposal cycle, not via direct edits during address-reviews); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `baseline spec; add a delta to your current change's specs/<capability>/spec.md to capture this ripple`. The reason text explicitly guides the user to the correct mutation path (delta operation in the change-dir, which sync-specs propagates at archive).

#### Scenario: Cascade skips safe-exclusion paths

- **WHEN** a finding's ripple-flag set includes any path under `.git/`, `node_modules/`, `dist/`, `build/`, or another safe-exclusion path
- **THEN** the cascade SKIPS that file (universal "never edit" set); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `safe-exclusion path; never edited`. Such paths typically should not appear in a ripple-flag set at all — if one does, it's likely a ripple-flag-analysis bug worth surfacing in the resolution log so the user can investigate.

#### Scenario: --no-cascade flag suppresses all cascade

- **WHEN** the user invokes `/opsx:address-reviews --no-cascade <scope>` or `/opsx:address-reviews <change-name> --no-cascade`
- **THEN** NO cascade edits are made regardless of file type or scope; ALL ripple-flagged files are recorded under `ripple_cascade.flagged_not_applied`. The resolution log's per-resolution audit-trail entries are identical to v1 list-only behavior — the cascade output is preserved as an audit record without acting.

#### Scenario: Cascade respects current-change scope only

- **WHEN** a finding's ripple-flag set includes a path under another active change (e.g., `openspec/changes/<other-name>/proposal.md`) or a path in the archive
- **THEN** the cascade SKIPS that file (cross-change ripples are explicitly out-of-scope); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `cross-change ripple; cascade scope is current change only`

#### Scenario: Cascade preserves marker-removal invariant

- **WHEN** a finding resolves with cascade-applied ripples in walk-mode
- **THEN** the `@review:` marker on the original finding is removed at the end of the per-finding cycle (per `Marker removal invariant`); cascaded files do NOT carry new markers (cascade is a fix, not a flagging operation)

## ADDED Requirements

### Requirement: Walk-mode by default with --batch opt-in

The system SHALL execute Step 3's lifecycle in walk-mode by default — each finding completes its full inner cycle (pushback → classify → fix → ripple-cascade → remove-marker) before the next finding begins. Batch-mode (all findings resolved together before any ripple-cascade fires) is opt-in.

**Opt-in triggers** (mutually-exclusive):

1. **`--batch` flag** in the invocation: `/opsx:address-reviews <scope> --batch`. Canonical signal.
2. **Verbal `--batch` in the invocation message only**: if the user's invocation message (the message that triggered the command) contains a clearly batch-intent phrase ("fix them all", "batch them", "go ahead with all", "address all at once", or similar phrases conveying batch-resolution intent), the AI SHALL treat this as a verbal `--batch` and proceed in batch-mode. The verbal phrase MUST be in the invocation message; phrases inside subsequent walk-step responses do NOT shift mode unless they are clearly command-shaped (e.g., a bare "go batch" or "switch to batch" message).
3. **Command-shaped mid-walk interruption**: during a walk, if the user sends a bare message that is unambiguously a mode-switch command ("go batch", "switch to batch", "batch the rest"), the AI SHALL treat this as a verbal `--batch` for the remainder of the walk. Ambiguous mid-walk phrases ("just fix them all" said in passing, "let's keep going") do NOT shift mode.

**Walk-mode UX**: at each per-finding boundary, the AI surfaces the finding's title + classification + proposed fix to the user via `AskUserQuestion`, applies the resolution, then proceeds to the next finding. The user can interrupt between findings.

**Batch-mode UX**: the AI completes pushback + classify + fix for all findings without per-finding `AskUserQuestion` prompts, then applies all ripple-cascades together as a final step. Decision-fork prompts (per `Disjunctive recommendation fields surface as decision forks`) still fire for findings classified as "decision required" — batch-mode does NOT suppress user decisions on disjunctive recommendations; it only suppresses per-finding-completion checkpoints for unambiguous fixes.

**Resolution log capture**: the top-level `walk_mode` field in `address-reviews-<TS>.json` SHALL be set to `"per_finding"` in walk-mode and `"batch"` in batch-mode.

#### Scenario: Default invocation runs in walk-mode

- **WHEN** the user invokes `/opsx:address-reviews <change-name>` (no `--batch` flag, no batch-intent verbal phrase in the invocation message) and three findings are discovered
- **THEN** the lifecycle walks each finding individually: pushback finding 1 → classify → fix → cascade → remove-marker → pushback finding 2 → classify → fix → cascade → remove-marker → pushback finding 3 → … The resolution log records `walk_mode: "per_finding"`.

#### Scenario: --batch flag triggers batch-mode

- **WHEN** the user invokes `/opsx:address-reviews <change-name> --batch`
- **THEN** the lifecycle completes all findings' pushback + classify + fix without per-finding checkpoints, then applies all ripple-cascades together. The resolution log records `walk_mode: "batch"`.

#### Scenario: Verbal batch-intent in invocation message triggers batch-mode

- **WHEN** the user's invocation message reads `/opsx:address-reviews <change-name> — fix them all` or `/opsx:address-reviews <change-name>` accompanied by a paragraph saying "batch them; I just want everything resolved"
- **THEN** the AI recognizes the verbal batch-intent and proceeds in batch-mode. The resolution log records `walk_mode: "batch"` AND includes a top-level `walk_mode_source` field with value `"verbal"` (vs. `"flag"` when `--batch` was passed; `"command-shape-interruption"` when triggered mid-walk).

#### Scenario: Mid-walk command-shaped interruption shifts to batch

- **WHEN** during a walk, after finding 1 is resolved, the user sends the bare message "go batch" or "switch to batch"
- **THEN** the AI shifts to batch-mode for findings 2, 3, …, N. The resolution log records `walk_mode: "batch"` with `walk_mode_source: "command-shape-interruption"` and an additional field `walk_mode_shifted_at_finding: 2` (the finding number at which the mode-shift occurred).

#### Scenario: Ambiguous mid-walk phrases do NOT shift mode

- **WHEN** during a walk, between finding 2 and finding 3, the user sends a response like "yeah let's just keep going" or "fine, fix them all" in the context of a back-and-forth conversation
- **THEN** the AI does NOT shift to batch-mode; the walk continues. The threshold for command-shape recognition is a bare, unambiguous mode-switch message; conversational continuations are not triggers.

#### Scenario: Batch-mode preserves decision-fork prompts

- **WHEN** batch-mode is active AND one of the findings is classified as "decision required" AND its `recommendation` field is disjunctive (triggering `Disjunctive recommendation fields surface as decision forks`)
- **THEN** the decision-fork prompt STILL fires for that finding even in batch-mode; only unambiguous "trivial fix" and "stale" findings short-circuit user interaction in batch-mode

### Requirement: Disjunctive recommendation fields surface as decision forks

When a finding's recommendation contains a disjunctive structure (multiple options the user must choose between, NOT just a single recommended action), the system SHALL surface the options as a decision fork via `AskUserQuestion` rather than collapsing to a passive line. The fork prompt fires within Step 3's walk, **after classify, before fix**, and **only when classify == "decision required"** — replacing the generic 2–4 option prompt in that path. Other classifications (`stale`, `trivial fix`, `unresolvable`) do not surface forks.

**Detection** (hybrid — structured path preferred, heuristic fallback):

1. **Structured path** (orbit-emit pipelines): when `--from-file` ingests an internal review JSON or audit-drift JSON whose finding entries include the optional `recommendation_options: [{label, body}, …]` field (per `orbit-review` and `orbit-audit-drift` finding-emit specs), the parser SHALL use this array directly as the fork options.
2. **Heuristic path** (external markdown findings, JSON findings without `recommendation_options`): the parser SHALL scan the finding's `**Description**:` (markdown) or `recommendation` (JSON) field for STRICT disjunctive signals:
   - Numbered alternatives: `(A) … (B)`, `1. … 2.`, `[A] … [B]`
   - "Either … or …" with clause-level branches (each clause is a recognizable recommendation)
   - "**Options**:" or "Options:" prefix followed by a list or numbered enumeration
3. **NOT** triggers: loose "or" in prose like "fix it now or later", "X or Y could happen", "the issue is A or B and we should address one of them." These prose patterns do NOT trigger fork detection.

**Source recording**: the resolution-log entry's `recommendation_fork.source` field SHALL be `"structured"` when the structured path fired, `"heuristic"` when the heuristic path fired.

**Fork prompt UX**: the system surfaces the options via `AskUserQuestion` with the finding title as the question and each option as a button. The system SHALL include a `[discuss]` option in every fork prompt as the escape hatch — selecting `[discuss]` causes the AI to surface tradeoff analysis on the options and re-prompt with the same fork (or a refined fork if the discussion clarifies a different option set).

**No global opt-out**: there is no flag to suppress fork prompts. `[discuss]` is the per-prompt escape; the user retains decision control without needing a flag.

**Capture in resolution log**: each resolved finding with a triggered fork SHALL emit a `recommendation_fork` object in its resolution-log entry (per `Resolution-log JSON shape extensions`). Findings without a triggered fork omit the field (or set `"detected": false`).

#### Scenario: Numbered-alternative finding surfaces fork

- **WHEN** address-reviews walks a finding (classified as "decision required") whose `recommendation` field reads `"Either (A) file a follow-up issue tracking the v2 polish work, or (B) add Group 19 to tasks.md extending this change's scope"`
- **THEN** the heuristic detector recognizes the `(A) … (B)` pattern; the system surfaces `AskUserQuestion` with two option buttons labeled `(A) file a follow-up issue tracking the v2 polish work` and `(B) add Group 19 to tasks.md extending this change's scope`, plus a `[discuss]` option

#### Scenario: Structured recommendation_options drives fork

- **WHEN** address-reviews ingests an internal review JSON via `--from-file` and one finding's entry includes `recommendation_options: [{label: "A", body: "file a follow-up issue"}, {label: "B", body: "extend scope to tasks.md"}]`
- **THEN** the parser uses `recommendation_options` directly (skipping heuristic scan of the `recommendation` string); the fork prompt surfaces buttons labeled `(A) file a follow-up issue` and `(B) extend scope to tasks.md` plus `[discuss]`; the resolution log records `recommendation_fork.source: "structured"`

#### Scenario: Single-recommendation finding does NOT surface fork

- **WHEN** address-reviews walks a finding whose `recommendation` field is a single concrete action (e.g., `"Replace the import path to match the renamed module"`) — no disjunctive structure
- **THEN** no fork is detected; the finding proceeds through the regular "decision required" path (generic 2–4 option prompt if needed) OR falls through to "trivial fix" if it qualifies. The resolution log omits `recommendation_fork` for this entry (or sets `recommendation_fork.detected: false`).

#### Scenario: Loose "or" in prose does NOT trigger fork

- **WHEN** address-reviews walks a finding whose `recommendation` field reads `"This will break if X or Y change; consider tightening the type signature"`
- **THEN** the heuristic detector does NOT trigger (no numbered alternatives, no clause-level "either…or", no "Options:" prefix); the finding proceeds through the regular "decision required" path

#### Scenario: Stale classification short-circuits before fork detection

- **WHEN** a finding's pushback classifies it as "stale" (already fixed at HEAD)
- **THEN** the lifecycle short-circuits to stale-suppression; fork detection does NOT run; the resolution log records the stale suppression without a `recommendation_fork` field

#### Scenario: Trivial fix classification short-circuits before fork detection

- **WHEN** a finding's pushback + classify yields "trivial fix" (single-line edit, one obvious correct answer)
- **THEN** the lifecycle applies the trivial fix without surfacing a fork prompt; fork detection does NOT run; the resolution log omits `recommendation_fork`

#### Scenario: Unresolvable classification short-circuits before fork detection

- **WHEN** a finding's pushback + classify yields "unresolvable" (needs information not currently available; will be filed as task / converted to TODO / escalated)
- **THEN** the lifecycle proceeds to the unresolvable-path UX; fork detection does NOT run; the resolution log omits `recommendation_fork`

#### Scenario: [discuss] surfaces tradeoff analysis and re-prompts

- **WHEN** the user selects `[discuss]` from a fork prompt
- **THEN** the AI surfaces a tradeoff analysis (pros/cons of each option, relevant context from the finding's pushback evidence, any related findings); after the discussion, the AI re-presents the fork (possibly with refined options if the discussion surfaced a third path) for the user to make a final choice. The `recommendation_fork.discuss_invoked` flag is set to `true` in the resolution log.

#### Scenario: Fork prompt fires in both walk-mode and batch-mode

- **WHEN** batch-mode is active AND a finding's classification is "decision required" AND the recommendation is disjunctive
- **THEN** the fork prompt still fires within the batch run (batch-mode does not suppress user decisions); only the per-finding completion checkpoints for unambiguous fixes are suppressed in batch-mode

#### Scenario: Malformed `recommendation_options` array falls back to heuristic detection

- **WHEN** the parser ingests a finding whose `recommendation_options` field is present but malformed — zero entries, exactly one entry, or any entry missing the required `label` or `body` keys
- **THEN** the consumer SHALL log a warning to stderr (e.g., `"warning: finding <title> at <file>:<line> has malformed recommendation_options (<reason>); falling back to heuristic detection over the recommendation prose"`) AND proceed via the heuristic-detection path over the finding's `recommendation` string. The malformed array does NOT abort the walk; the consumer treats the structured path as if it were absent and applies heuristic detection. The resolution log records `recommendation_fork.source: "heuristic"` (if detection fires) and includes a `recommendation_fork.structured_path_skipped_reason` field naming why the structured path was skipped (e.g., `"only 1 entry in array"`, `"missing label on entry index 2"`).

### Requirement: Resolution-log JSON shape extensions for walk-mode / decision-forks / cascade

The system SHALL extend the `address-reviews-<TS>.json` resolution-log shape with three additions: a top-level `walk_mode` field, a per-resolution optional `recommendation_fork` object, and a per-resolution `ripple_cascade.applied / flagged_not_applied` split.

**Field semantics**:

- **`walk_mode`** (top-level, required): `"per_finding"` (walk-mode, the default) or `"batch"`.
- **`walk_mode_source`** (top-level, required when `walk_mode == "batch"`): `"flag"` (via `--batch`), `"verbal"` (via verbal trigger in invocation message), or `"command-shape-interruption"` (via mid-walk command-shaped message). Omitted when `walk_mode == "per_finding"`.
- **`walk_mode_shifted_at_finding`** (top-level, present only when `walk_mode_source == "command-shape-interruption"`): the 1-indexed finding number at which the mode shifted.
- **`recommendation_fork`** (per-resolution, optional): present only when a fork was detected and triggered. Object shape:
  - `detected: true` — always true when present
  - `source: "structured" | "heuristic"` — which detection path fired
  - `options_presented: [{label, body}, …]` — structured options as shown to the user; shape is identical to the producer-side `recommendation_options` field on the input finding, so downstream consumers parse one shape across producer JSON and resolution-log JSON
  - `chosen: "A" | "B" | …` — the option label the user picked (matches one of the `options_presented[].label` values)
  - `discuss_invoked: bool` — whether `[discuss]` was invoked before the final choice
  - `structured_path_skipped_reason: string` — present only when the structured detection path was attempted but skipped due to malformed input (per the `Malformed recommendation_options array falls back to heuristic detection` scenario). Examples: `"only 1 entry in array"`, `"missing label on entry index 2"`, `"empty body on entry index 0"`. Omitted when the structured path either succeeded OR was simply absent (no `recommendation_options` field on the input).
- **`ripple_cascade`** (per-resolution, required): replaces the v1 `ripple_flagged_files_aggregate` array. Object shape:
  - `applied: [path, …]` — files cascade-edited consistent with the primary fix
  - `flagged_not_applied: [{path, reason}, …]` — files identified as ripple-related but NOT edited (either out-of-scope by policy, OR `--no-cascade` suppressed all)

**Backward compatibility — structural change, not just rename**: the v1 `ripple_flagged_files_aggregate` field is a top-level flat array of strings (aggregating ripple-flagged paths across ALL resolutions, with no per-resolution attribution). The v2 `ripple_cascade.applied / flagged_not_applied` fields are per-resolution (each entry in `resolutions[]` carries its own `ripple_cascade` object). The shapes are NOT bijective — v1 archived JSONs cannot be losslessly upconverted to v2 (per-resolution attribution is lost in v1). Reader guidance:

- **Reading v1**: presence of top-level `ripple_flagged_files_aggregate` (and absence of per-resolution `ripple_cascade`) signals v1 format. Treat the aggregate as the union of all per-resolution ripple sets across the run; attribution to specific findings is not available.
- **Reading v2**: presence of per-resolution `ripple_cascade` is the canonical format going forward. The top-level `ripple_flagged_files_aggregate` field is NOT emitted in v2 (removed, not renamed).

Existing archived v1 JSONs (e.g., `openspec/changes/archive/2026-05-24-lean-overlay-and-add-orbit-onboard/.orbit-runs/address-reviews-2026-05-23T18-03-25Z.json:59-66`) retain their v1 shape; readers handling both formats must check for which field is present.

#### Scenario: Walk-mode resolution log structure

- **WHEN** address-reviews completes a walk-mode run with 3 findings (all "decision required", no disjunctive recommendations) and 2 ripple-flagged files (both in the IN set; can be any extension)
- **THEN** the emitted JSON contains `walk_mode: "per_finding"` (top-level), 3 entries in `resolutions[]` each WITHOUT a `recommendation_fork` field, and each entry's `ripple_cascade` shows the applicable cascaded paths in `applied` and any policy-skipped paths in `flagged_not_applied`

#### Scenario: Batch-mode resolution log structure

- **WHEN** address-reviews completes a batch-mode run (triggered by `--batch`)
- **THEN** the emitted JSON contains `walk_mode: "batch"` and `walk_mode_source: "flag"`; per-resolution `ripple_cascade` fields are populated only at the end of the batch (each resolution's `applied`/`flagged_not_applied` reflects the final aggregated cascade pass)

#### Scenario: Recommendation_fork captured when disjunctive triggered

- **WHEN** address-reviews walks a finding with a disjunctive recommendation; user chooses option A without invoking `[discuss]`
- **THEN** the resolution-log entry for that finding contains `recommendation_fork: {detected: true, source: "heuristic", options_presented: [{label: "A", body: "..."}, {label: "B", body: "..."}], chosen: "A", discuss_invoked: false}`

#### Scenario: Recommendation_fork captures discuss invocation

- **WHEN** the user invokes `[discuss]`, the AI presents tradeoffs, the user then chooses option B
- **THEN** the resolution-log entry contains `recommendation_fork: {…, chosen: "B", discuss_invoked: true}`

#### Scenario: --no-cascade resolution log records all ripples as flagged_not_applied

- **WHEN** address-reviews runs with `--no-cascade` and the finding's ripple-flag set includes 3 files (all IN-set files that would normally cascade)
- **THEN** the resolution-log entry contains `ripple_cascade.applied: []` (empty) AND `ripple_cascade.flagged_not_applied: [{path: <path1>, reason: "--no-cascade suppressed"}, …]` (all 3 paths). Note: `--no-cascade` does not classify by IN/OUT — it suppresses cascade entirely; the `--no-cascade suppressed` reason applies uniformly. Per-OUT-category reasons (audit-trail, baseline-spec, etc.) only appear in the default mode when cascade DID try to apply and was structurally blocked.

#### Scenario: Resolution log omits recommendation_fork for findings without forks

- **WHEN** address-reviews walks 5 findings: 1 stale, 2 trivial fix, 1 decision-required-with-fork, 1 decision-required-without-fork
- **THEN** only the 1 fork-triggered finding's resolution entry contains `recommendation_fork`; the other 4 entries omit the field entirely (consumers MUST treat absence as "no fork")
