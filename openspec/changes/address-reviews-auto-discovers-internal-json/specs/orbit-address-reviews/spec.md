## MODIFIED Requirements

### Requirement: Address-reviews command available

The system SHALL expose a `/opsx:address-reviews` command that scans for `@review:` markers in the repo (or ingests review findings via `--from-file`) AND walks each through resolution with pushback discipline. When invoked with a change-name positional argument and no markers are found in the change directory, the system SHALL ALSO attempt to auto-discover the most recent internal review JSON OR audit-drift JSON in the change's `.orbit-runs/` directory and walk it as if it had been passed via `--from-file` — closing the "review → address-reviews" convenience loop so users don't need to type `--from-file <path>` or pre-run `/opsx:review --as proposal --mark`.

**Discovery priority order** (normative): (1) `@review:` markers grep in the change directory — if any found, walk them and auto-discovery does NOT fire. (2) If markers grep returns zero AND no `--from-file` flag is specified AND positional is a change name: auto-discover the most-recent candidate JSON in `.orbit-runs/`. (3) Otherwise: clean "no findings" exit. Explicit `--from-file <path>` always overrides auto-discovery regardless of marker/JSON state.

**Positional argument resolution** (normative): a positional argument is treated as a change name if and only if it resolves to a directory at `openspec/changes/<name>/` OR `openspec/changes/archive/<YYYY-MM-DD>-<name>/`. Otherwise the positional is treated as a path/pattern scope for the grep, and auto-discovery does NOT apply.

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
- **THEN** auto-discovery does NOT run; the command uses the explicitly-specified `--from-file <path>` regardless of marker presence, JSON availability, or recency. Explicit user intent wins. (The marker grep still happens — markers found at the change-name scope are still presented in the triage list alongside the `--from-file` virtual markers; this is current behavior unchanged.)

#### Scenario: Multiple JSON candidates resolved by recency

- **WHEN** auto-discovery considers candidate files in `.orbit-runs/` AND multiple matching files exist (e.g., one `review-proposal-*.json`, one `review-system-*.json`, one `audit-drift-*.json`)
- **THEN** the command picks the single most-recent file by filename `<TS>` token across all candidate types — review and audit-drift JSON compete on the same recency axis, NOT a class-preference (i.e., audit-drift CAN win over review if its `<TS>` token is later). If two candidate files share an identical `<TS>` token (rare; possible only if emitted within the same ISO-second), tie-break by stable lexicographic sort of the full filename (e.g., `audit-drift-<TS>.json` sorts before `review-proposal-<TS>.json` before `review-system-<TS>.json`) and the alphabetically-earlier filename wins. The user can override via explicit `--from-file <path>` if they wanted a different (non-most-recent) JSON.

#### Scenario: Auto-discovered JSON walks the standard ingest lifecycle

- **WHEN** auto-discovery selects a candidate JSON and proceeds to walk it
- **THEN** the candidate is fed to the parser per the `--from-file ingest of review findings file` requirement (content-sniff routes to JSON parser based on leading `{`; per-finding virtual markers constructed per the parser contract; lifecycle proceeds via pushback → classify → fix → ripple-flag → no-op marker-removal). The resolution log identifies the source in its `source` field as `"auto-discovered"` (vs `"from-file"` for explicit invocation), preserving the audit trail of how the findings entered the lifecycle.

#### Scenario: `--mark` no longer prerequisite for the address-reviews workflow

- **WHEN** a user runs `/opsx:review <change-name>` (without `--mark`) followed immediately by `/opsx:address-reviews <change-name>`
- **THEN** address-reviews finds the just-written `review-<mode>-*.json` via auto-discovery and walks its findings — `--mark` is NOT required for the canonical 2-command workflow to function. `--mark` remains useful for users who specifically want source-level annotations (e.g., for diff-readability), but it is not a structural prerequisite.

#### Scenario: Auto-discovery respects archive location

- **WHEN** the change-name positional resolves to an archived change at `openspec/changes/archive/<YYYY-MM-DD>-<change-name>/`
- **THEN** auto-discovery looks for JSON candidates in the archived location's `.orbit-runs/` directory (which travelled with the change per the archive flow). Behavior is identical to the active-change case.
