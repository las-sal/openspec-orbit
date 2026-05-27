# orbit-address-reviews

## Purpose

The `/opsx:address-reviews` command (lean v1) — resolves `@review:` markers anywhere in the repo (or external-review findings via `--from-file`) by walking each through pushback → classify → fix → ripple-flag → remove-marker. Pushback is the primary discipline: verify against current state before fixing, suppress stale findings with evidence. Output is a resolution log (NOT a scorecard) with ✓ Resolved / ⚠ Stale / ⏸ Deferred / ✗ Escalated counts. Marker-removal invariant ensures markers don't leak into canonical artifacts.

## Requirements

### Requirement: Address-reviews command available

The system SHALL expose a `/opsx:address-reviews` command that scans for `@review:` markers in the repo (or ingests review findings via `--from-file`) AND walks each through resolution with pushback discipline. When invoked with a change-name positional argument and no markers are found in the change directory, the system SHALL ALSO attempt to auto-discover the most recent internal review JSON OR audit-drift JSON in the change's `.orbit-runs/` directory and walk it as if it had been passed via `--from-file` — closing the "review → address-reviews" convenience loop so users don't need to type `--from-file <path>` or pre-run `/opsx:review --as proposal --mark`.

**Discovery priority order** (normative): (1) `@review:` markers grep in the change directory — if any found, walk them and auto-discovery does NOT fire. (2) If markers grep returns zero AND no `--from-file` flag is specified AND positional is a change name: auto-discover the most-recent candidate JSON in `.orbit-runs/`. (3) Otherwise: clean "no findings" exit. Explicit `--from-file <path>` always overrides auto-discovery regardless of marker/JSON state.

**Positional argument resolution** (normative): a positional argument is treated as a change name if and only if it matches one of these locations, checked in this order: (1) **active**: `openspec/changes/<name>/` (exact directory match); (2) **archived**: `openspec/changes/archive/<YYYY-MM-DD>-<name>/` — enumerate entries in `openspec/changes/archive/` matching the regex `^\d{4}-\d{2}-\d{2}-<name>$`. If multiple archive entries match the same `<name>` suffix (rare; would mean the same change name was archived on different dates), the lexicographically-latest date wins (most-recent archive). If no active match AND no archive match, the positional is treated as a path/pattern scope for the grep, and auto-discovery does NOT apply.

**`--from-file` exclusivity** (normative): when `--from-file <path>` is specified, the parser ingests the file's content and that is the ONLY input source. Marker grep does NOT run; auto-discovery does NOT run; the positional argument (if present) is used only for emit-path resolution (where the resulting `address-reviews-<TS>.json` is written) and does NOT affect discovery. This preserves the baseline behavior where default scan / scoped scan / `--from-file` are 3 alternative input paths, not combinable inputs.

#### Scenario: Default scan

- **WHEN** the user invokes `/opsx:address-reviews` with no arguments
- **THEN** the command greps the whole repo for `@review:` markers, respecting safe exclusions (`.git`, `node_modules`, `dist`, `build`). Auto-discovery does NOT apply — there's no change-directory anchor for `.orbit-runs/` lookup.

#### Scenario: Scoped scan — path or pattern

- **WHEN** the user invokes `/opsx:address-reviews <scope>` with a path or pattern (NOT a recognized change name)
- **THEN** the command restricts the marker scan to the specified scope while keeping safe exclusions. Auto-discovery does NOT apply — path/pattern scopes don't anchor on a change directory.

#### Scenario: Change-name positional scope with markers found

- **WHEN** the user invokes `/opsx:address-reviews <change-name>` (positional resolves to an existing change directory at `openspec/changes/<name>/` or `openspec/changes/archive/<date>-<name>/`) AND grep finds one or more `@review:` markers in the change directory
- **THEN** the command walks the discovered markers (current behavior; auto-discovery does NOT fire because markers already provide work)

#### Scenario: Change-name positional scope with no markers — auto-discovers internal JSON

- **WHEN** the user invokes `/opsx:address-reviews <change-name>` AND grep finds zero `@review:` markers in the change directory AND no `--from-file` flag is specified
- **THEN** the command looks in `openspec/changes/<change-name>/.orbit-runs/` (or the archive equivalent) for files matching `review-<mode>-*.json` OR `audit-drift-*.json`, picks the single most-recent by filename `<TS>` token (per orbit-conventions `Internal-run JSON summary format`; lexical sort works because tokens are `YYYY-MM-DDTHH-MM-SSZ` — both filename patterns compete on the same recency axis regardless of the file's `command` value), and walks it through the `--from-file` ingest path (per the `--from-file ingest of review findings file` requirement's existing parser contract). The triage step (Step 2) presents the auto-discovered findings as a numbered list before walking begins.

#### Scenario: Change-name positional scope with no markers AND no candidate JSON

- **WHEN** the user invokes `/opsx:address-reviews <change-name>` AND grep finds zero `@review:` markers AND the change's `.orbit-runs/` directory contains no `review-<mode>-*.json` OR `audit-drift-*.json` files (or the directory itself doesn't exist)
- **THEN** the command emits `No @review: markers in scope and no internal review/audit-drift JSON in .orbit-runs/. Nothing to walk.` and exits cleanly with a successful exit code (this is a normal no-work state, not an error)

#### Scenario: Explicit `--from-file` overrides auto-discovery

- **WHEN** the user invokes `/opsx:address-reviews <change-name> --from-file <path>` with both a change-name positional AND an explicit `--from-file <path>`
- **THEN** auto-discovery does NOT run AND marker grep does NOT run; the command uses the explicitly-specified `--from-file <path>` as the ONLY input source regardless of marker presence, JSON availability, or recency. Explicit user intent wins. The positional `<change-name>` is used only for emit-path resolution (the resulting `address-reviews-<TS>.json` is written to that change's `.orbit-runs/` directory).

#### Scenario: Multiple JSON candidates resolved by recency

- **WHEN** auto-discovery considers candidate files in `.orbit-runs/` AND multiple matching files exist (e.g., one `review-proposal-*.json`, one `review-system-*.json`, one `audit-drift-*.json`)
- **THEN** the command picks the single most-recent file by filename `<TS>` token across all candidate types — review and audit-drift JSON compete on the same recency axis, NOT a class-preference (i.e., audit-drift CAN win over review if its `<TS>` token is later). If two candidate files share an identical `<TS>` token (rare; possible only if emitted within the same ISO-second), tie-break by stable lexicographic sort of the full filename (ASCII order: `a` < `r`; within `review-`, `proposal` < `system` → so `audit-drift-<TS>.json` sorts before `review-proposal-<TS>.json` sorts before `review-system-<TS>.json`) and the alphabetically-earliest filename wins. The user can override via explicit `--from-file <path>` if they wanted a different (non-most-recent) JSON.

#### Scenario: Auto-discovered JSON walks the standard ingest lifecycle

- **WHEN** auto-discovery selects a candidate JSON and proceeds to walk it
- **THEN** the candidate is fed to the parser per the `--from-file ingest of review findings file` requirement (content-sniff routes to JSON parser based on leading `{`; per-finding virtual markers constructed per the parser contract; lifecycle proceeds via pushback → classify → fix → ripple-flag → no-op marker-removal).

#### Scenario: Auto-discovery resolution log captures audit-trail evidence

- **WHEN** auto-discovery succeeds (a candidate JSON is selected and walked) and the resolution log is emitted to `.orbit-runs/address-reviews-<TS>.json`
- **THEN** the log SHALL include the following audit-trail fields documenting the discovery decision: (a) `source: "auto-discovered"` (the new enum value distinguishing this from explicit `from-file` / `whole-repo` / `scope`); (b) `source_path`: the repo-relative path to the selected JSON file (e.g., `openspec/changes/<name>/.orbit-runs/review-system-2026-05-27T00-43-10Z.json`); (c) `source_command`: the `command` value read from the selected JSON's top-level field (`"review"` or `"audit-drift"`); (d) `source_token`: the filename `<TS>` token of the selected JSON; (e) `latest_apply_token`: the filename `<TS>` token of the most-recent `apply-*.json` in the same `.orbit-runs/`, OR `null` if no apply JSON exists. The `latest_apply_token` field exists because D-no-stale-detection delegates stale-detection to lifecycle pushback rather than refusing the walk; recording the apply token in the resolution log gives downstream auditors the evidence to assess whether the JSON was stale relative to artifacts at walk-time. If the recency comparison invoked tie-break (identical `<TS>` tokens), an optional (f) `tie_break_rationale`: short string explaining which file won and why (e.g., `"audit-drift-<TS>.json won by stable lexicographic sort over review-proposal-<TS>.json"`).

#### Scenario: `--mark` no longer prerequisite for the address-reviews workflow

- **WHEN** a user runs `/opsx:review <change-name>` (without `--mark`) followed immediately by `/opsx:address-reviews <change-name>`
- **THEN** address-reviews finds the just-written `review-<mode>-*.json` via auto-discovery and walks its findings — `--mark` is NOT required for the canonical 2-command workflow to function. `--mark` remains useful for users who specifically want source-level annotations (e.g., for diff-readability), but it is not a structural prerequisite.

#### Scenario: Auto-discovery respects archive location

- **WHEN** the change-name positional resolves to an archived change at `openspec/changes/archive/<YYYY-MM-DD>-<change-name>/`
- **THEN** auto-discovery looks for JSON candidates in the archived location's `.orbit-runs/` directory (which travelled with the change per the archive flow). Behavior is identical to the active-change case.

### Requirement: `--from-file` ingest of review findings file

The system SHALL accept a `--from-file <path>` flag and parse the file's content into virtual markers for resolution. The parser SHALL auto-detect the file's format via content sniff and support TWO format families: (a) external-review markdown (produced by external AIs per `references/external-findings-format.md`), (b) internal findings JSON (produced by `/opsx:review` or `/opsx:audit-drift`, per `references/internal-findings-format.md`). V1 accepts JSON files whose top-level `command` field is either `"review"` or `"audit-drift"`; other `command` values are rejected with a clean error. Each finding becomes a virtual marker that walks the same lifecycle as inline markers, with marker-removal as a no-op (no source-file marker text exists to remove for either virtual-marker type).

#### Scenario: Parse external findings markdown

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace prefix is `# External Review:` (markdown header) and the body follows the orbit external-review markdown format
- **THEN** the parser extracts each finding (severity, title, file:line, description) as a virtual marker with `source: "external"` and walks it through the same lifecycle as inline markers, except no source-file marker text exists to remove

#### Scenario: Parse internal review JSON

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace character is `{` AND the file parses as valid JSON AND the top-level `command` field is `"review"` AND the JSON shape matches the orbit internal-run review summary (per orbit-conventions `Internal-run JSON summary format` + the review-specific extensions documented at `.claude/skills/openspec-review/references/run-summary-schema.md`)
- **THEN** the parser extracts each entry in the JSON's `findings[]` array as a virtual marker — mapping `severity` directly, `title` directly, `file:line` from `file` + `line` fields, `description` from the entry's `recommendation` field — tags `source: "internal-review"` on each, and walks each through the same lifecycle as inline markers, with marker-removal as a no-op

#### Scenario: Parse internal audit-drift JSON

- **WHEN** the command runs with `--from-file <path>` and the file's first non-whitespace character is `{` AND the file parses as valid JSON AND the top-level `command` field is `"audit-drift"` AND the JSON shape matches the orbit audit-drift summary (per orbit-conventions `Internal-run JSON summary format` + the audit-drift-specific extensions documented at `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`)
- **THEN** the parser extracts each entry in the JSON's `findings[]` array as a virtual marker — mapping `severity` directly, `title` directly, `file:line` from `file` + `line` fields, `description` from the entry's `recommendation` field, with the audit-drift `category` field (`"1"`–`"4"`) used as the provenance/pass-slot detail in the virtual marker — tags `source: "audit-drift"` on each, and walks each through the same lifecycle as inline markers, with marker-removal as a no-op. The audit-drift virtual-marker lifecycle is identical to internal-review except for the `source` tag value and the provenance detail (`category` rather than `pass`); the resolution log and address-reviews emit are otherwise indistinguishable.

#### Scenario: Auto-detect content sniff routes to correct parser

- **WHEN** the command runs with `--from-file <path>` and the file's content needs format routing
- **THEN** the parser inspects only the first non-whitespace token to route: leading `{` routes to JSON parser; leading `# External Review:` routes to markdown parser (the only markdown format orbit produces); anything else triggers the format-mismatch error. The sniff does NOT depend on file extension or pathname.

#### Scenario: Unsupported JSON command field

- **WHEN** `--from-file <path>` resolves to a JSON file whose top-level `command` field is anything other than `"review"` or `"audit-drift"` (e.g., `"address-reviews"`, `"apply"`, `"archive"`, `"propose"`, `"explore"`, `"new"`, `"continue"`, `"ff"`, `"review-external"`)
- **THEN** the parser emits a clean error message naming the supported `command` values (`"review"`, `"audit-drift"`) and the unsupported value detected, and exits without acting on any findings. The message references both supported format reference files so the user can self-diagnose. Notably, `command: "address-reviews"` is rejected to avoid recursive cycles (a walk that produces its own input would loop).

#### Scenario: Internal JSON missing `findings[]` field

- **WHEN** `--from-file <path>` resolves to a JSON file with a supported `command` (`"review"` or `"audit-drift"`) AND valid JSON parse AND no `findings` field at the top level (or `findings` present but not an array)
- **THEN** the parser treats this as a malformed-input parse error — emits a clean error message naming the missing/malformed `findings[]` requirement and references `references/internal-findings-format.md` for the expected shape; exits without acting

#### Scenario: Internal JSON with empty `findings[]`

- **WHEN** `--from-file <path>` resolves to a JSON file with a supported `command` (`"review"` or `"audit-drift"`) AND `findings: []` (empty array — the source command ran cleanly and surfaced no findings)
- **THEN** the parser succeeds with zero virtual markers; the resolution log reports an empty walk (✓ Resolved: 0, ⚠ Stale: 0, ⏸ Deferred: 0, ✗ Escalated: 0) with a one-line note `Source JSON had no findings to walk; resolution log is informational.` and exits cleanly. This is NOT an error — a clean review or clean audit-drift run with `findings: []` is the expected state for an all-clear pass.

#### Scenario: Malformed input — neither format matches

- **WHEN** the `--from-file` path resolves to a file whose first non-whitespace prefix is neither `{` nor `# External Review:`
- **THEN** the command emits a clean error message naming both supported formats (referencing `references/external-findings-format.md` and `references/internal-findings-format.md`) plus the observed leading-content snippet, and exits without acting on partial input

#### Scenario: Malformed input — JSON parse failure

- **WHEN** the `--from-file` path looks like JSON (leading `{`) but fails to parse as valid JSON
- **THEN** the command emits a clean parse-error message naming the file and the JSON parse-error position (best-effort), and exits without acting on partial input. The user fixes the file and re-runs.

#### Scenario: Malformed input — markdown missing required sections

- **WHEN** the `--from-file` path looks like markdown (leading `# External Review:`) but is missing required sections OR has broken field labels OR otherwise can't be parsed cleanly per the external-findings parser contract
- **THEN** the command reports a parse error with format guidance (referencing `references/external-findings-format.md`) and exits without acting on partial input

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

### Requirement: Pushback discipline applied per marker

The system SHALL verify each marker against current state before fixing — regardless of whether the marker is inline (grep-found), markdown-virtual (`--from-file` external), or JSON-virtual (`--from-file` internal review or audit-drift JSON).

#### Scenario: Already fixed at HEAD

- **WHEN** the marker's claim references something already fixed in the current state (verified via `grep`, `git log`, or file inspection)
- **THEN** the marker is classified as "stale, suppressed"; current state and evidence (commit hash or grep result) are reported; the marker is removed without further edit

#### Scenario: Still applies

- **WHEN** the marker's claim is still valid against current state
- **THEN** the resolution proceeds to classification

#### Scenario: Pushback decision procedure

- **WHEN** the AI verifies a marker against current state
- **THEN** it follows this procedure in order: (1) identify the marker's referenced symbol, name, or concept; (2) `grep -rn` for current presence in expected locations (the file the marker is in, related files, baseline specs); (3) if the symbol is absent where the marker expects it, run `git log -S "<symbol>" --since=<reasonable-window>` (default: since the marker file was last modified) to confirm intentional removal; (4) read the relevant file's current content; (5) compare to the marker's claim and decide: still applies / already fixed / partially applies; (6) on already-fixed, report the commit hash or current-content evidence as part of suppression

#### Scenario: Internal JSON virtual markers receive fresh pushback

- **WHEN** the command ingests an internal review JSON OR audit-drift JSON via `--from-file` and walks its `findings[]` entries as virtual markers
- **THEN** each entry receives fresh pushback against current state — the source command (`/opsx:review` or `/opsx:audit-drift`) that produced the JSON already filtered findings into its own `stale_suppressed[]` array at source-run time, BUT pushback is re-applied at address-reviews time because state may have changed between the source run and the resolve (e.g., user fixed an issue ad-hoc; another commit landed). The lifecycle does NOT skip pushback for any JSON-virtual marker regardless of source `command`.

### Requirement: Classification of each marker

The system SHALL classify each marker as one of: trivial fix, decision required, stale, or unresolvable.

#### Scenario: Classification heuristics

- **WHEN** the AI classifies a marker after pushback
- **THEN** it applies these heuristics in order: (a) **stale** — pushback determined the issue is already resolved at HEAD; (b) **trivial fix** — the resolution is a single-line edit or a few-line localized edit with one obvious correct answer (no design implication, no scope question, no ambiguity in intent); (c) **decision required** — the resolution requires ambiguity resolution, a design choice between defensible alternatives, a scope decision, or has implications beyond the immediate location; (d) **unresolvable** — the resolution needs information not currently available (e.g., depends on a deferred decision, requires a future capability, blocked on external input)

#### Scenario: Trivial fix

- **WHEN** a marker is classified as trivial fix
- **THEN** the command proposes the edit, applies it, and removes the marker (no `AskUserQuestion` needed)

#### Scenario: Decision required

- **WHEN** a marker is classified as decision required
- **THEN** the command surfaces 2-4 concrete options via `AskUserQuestion` (rather than asking open-ended); applies the user's choice; removes the marker

#### Scenario: Unresolvable — default file as task

- **WHEN** the marker can't be resolved now and the user has not specified an alternative
- **THEN** the command files a follow-up task in `tasks.md` and removes the marker

#### Scenario: Unresolvable — convert to permanent TODO

- **WHEN** the user chooses to convert an unresolvable marker
- **THEN** the marker text is replaced with `@todo: <content>` and the resolution log notes the conversion

#### Scenario: Unresolvable — escalate

- **WHEN** the user chooses to escalate
- **THEN** the marker is replaced with `@review(escalated): <content with explanation>` and the resolution log notes the escalation

### Requirement: Marker removal invariant

The system SHALL remove the original `@review:` marker on resolution unless `--keep-resolved-markers` is set.

#### Scenario: Marker removed by default

- **WHEN** a marker has been resolved (trivial fix applied, decision applied, or stale-suppression)
- **THEN** the marker text is deleted from its source file

#### Scenario: `--keep-resolved-markers` debug flag

- **WHEN** the command runs with `--keep-resolved-markers`
- **THEN** markers remain in their source files even after resolution (debug use)

### Requirement: Ripple cascade by default

The system SHALL automatically apply ripple-affected edits to in-scope files when a finding resolves, unless `--no-cascade` is specified. Ripple-flagged files are always recorded in the resolution log; the difference between default and `--no-cascade` modes is whether the system edits them or merely lists them.

**Cascade scope** (normative — defines which files cascade refuses to edit; everything else ripple-flagged is fair game regardless of file extension):

The cascade IN set is defined by exclusion. The OUT list captures four lifecycle-invariant categories — each entry is OUT for a structural workflow reason, not for a "this might be unsafe" heuristic. Files NOT matching any OUT category are IN, regardless of extension (`.py`, `.swift`, `.c`, `.sh`, `.md`, configs, dotfiles — all eligible if ripple-flagged).

- **OUT — change-dir audit trail**: `openspec/changes/<name>/.orbit-runs/*` and `openspec/.orbit-runs/*`. Audit-trail files; editing would corrupt the workflow's own record. NEVER edited by cascade.
- **OUT — baseline specs**: `openspec/specs/<capability>/spec.md`. Baseline mutations flow through the proposal cycle (`/opsx:propose` → delta spec → `/opsx:archive` triggers `sync-specs` to propagate). Cascade refuses direct baseline edits; the appropriate action is to add a delta in the current change's `specs/<capability>/spec.md`.
- **OUT — cross-change directories (active and archived)**: `openspec/changes/<other-name>/*` (other active changes' directories) AND `openspec/changes/archive/*` (any archived change, regardless of date prefix). Change-isolation invariant for active changes; immutable-history invariant for archived changes. Cascade never edits either, even when the current change's ripple-flag set surfaces a path inside one. The corresponding scenario `Cascade respects current-change scope only` documents both cases.
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

- **WHEN** a finding's ripple-flag set includes any path under exactly one of the four safe-exclusion prefixes: `.git/`, `node_modules/`, `dist/`, `build/`
- **THEN** the cascade SKIPS that file (universal "never edit" set); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `safe-exclusion path; never edited`. Such paths typically should not appear in a ripple-flag set at all — if one does, it's likely a ripple-flag-analysis bug worth surfacing in the resolution log so the user can investigate. The safe-exclusion prefix set is EXACT — implementers MUST NOT extend it ad-hoc to other paths during codegen. Expanding the set requires a spec change to amend this list explicitly; the Option D framing depends on the OUT list being closed and predictable.

#### Scenario: --no-cascade flag suppresses all cascade

- **WHEN** the user invokes `/opsx:address-reviews --no-cascade <scope>` or `/opsx:address-reviews <change-name> --no-cascade`
- **THEN** NO cascade edits are made regardless of file type or scope; ALL ripple-flagged files are recorded under `ripple_cascade.flagged_not_applied`. The resolution log's per-resolution audit-trail entries are identical to v1 list-only behavior — the cascade output is preserved as an audit record without acting.

#### Scenario: Cascade respects current-change scope only

- **WHEN** a finding's ripple-flag set includes a path under another active change (e.g., `openspec/changes/<other-name>/proposal.md`) or a path in the archive
- **THEN** the cascade SKIPS that file (cross-change ripples are explicitly out-of-scope); the resolution log records the path under `ripple_cascade.flagged_not_applied` with reason `cross-change ripple; cascade scope is current change only`

#### Scenario: Cascade preserves marker-removal invariant

- **WHEN** a finding resolves with cascade-applied ripples in walk-mode
- **THEN** the `@review:` marker on the original finding is removed at the end of the per-finding cycle (per `Marker removal invariant`); cascaded files do NOT carry new markers (cascade is a fix, not a flagging operation)

### Requirement: Resolution log output

The system SHALL emit a resolution log (not a 3-dimension scorecard) summarizing per-item outcomes.

#### Scenario: Output structure

- **WHEN** the command completes
- **THEN** the output includes a summary table with counts of ✓ Resolved, ⚠ Stale, ⏸ Deferred, ✗ Escalated; followed by per-section listings of each marker's file:line, brief description, action taken, and any related files flagged

#### Scenario: Final assessment

- **WHEN** the resolution log is complete
- **THEN** the final assessment reports remaining markers in scope (0 if clean, plus count of any deliberately persisted escalated markers) and suggests next step (e.g., "re-run /opsx:review --as proposal to confirm clean baseline")

### Requirement: Marker convention across file types

The system SHALL recognize `@review:` markers uniformly across file types: bare in markdown; inside the file type's comment syntax in source code and configs.

#### Scenario: Markdown marker

- **WHEN** a markdown file contains `@review: <text>` (bare)
- **THEN** the marker is discovered and resolution applies

#### Scenario: Source code marker (C-style comment)

- **WHEN** a source file contains `// @review: <text>` or `/* @review: <text> */`
- **THEN** the marker is discovered; resolution can either remove the marker text alone (leaving the comment) or remove the comment entirely if it contained only the marker

#### Scenario: Source code marker (hash comment)

- **WHEN** a source file or config contains `# @review: <text>`
- **THEN** the marker is discovered with the same removal behavior

### Requirement: No new markers without user consent

The system SHALL NOT create new `@review:` markers as part of its own operation.

#### Scenario: address-reviews does not write markers

- **WHEN** address-reviews runs
- **THEN** the only marker creation possible is by `/opsx:review --as proposal --mark` (a different command); address-reviews never writes new markers

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
   - "Options:" prefix followed by a list or numbered enumeration, including both bold variants (`**Options**:` with the colon outside the bold span, AND `**Options:**` with the colon inside the bold span) — both are accepted equivalently after markdown normalization. Implementers SHOULD normalize the input's markdown emphasis before pattern-matching this trigger so the detector behaves consistently across the two conventions.
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
