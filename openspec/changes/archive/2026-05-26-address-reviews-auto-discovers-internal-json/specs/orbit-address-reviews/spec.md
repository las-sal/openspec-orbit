## MODIFIED Requirements

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
