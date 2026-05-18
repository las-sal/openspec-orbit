## ADDED Requirements

### Requirement: Review-external command available

The system SHALL expose a `/opsx:review-external` command that packages a review request for an external AI (codex, fresh Claude, GPT, etc.) to perform a second-opinion review.

#### Scenario: Invoke with change name and mode

- **WHEN** the user invokes `/opsx:review-external <change-name> --as proposal` (or `--as system`)
- **THEN** the command generates a self-contained markdown prompt and emits it to chat for the user to copy

#### Scenario: Invoke without change name

- **WHEN** the user invokes `/opsx:review-external` with no argument
- **THEN** the command runs `openspec list --json` and prompts via `AskUserQuestion` to pick a change

### Requirement: Mode flag with state-based inference default

The system SHALL accept a `--as proposal|system` flag and, when omitted, infer the mode from the change's `tasks.md` state.

#### Scenario: Tasks unchecked → infer proposal mode

- **WHEN** `--as` is omitted and the change's `tasks.md` has any unchecked task boxes
- **THEN** the command proceeds as `--as proposal` and the chat output includes "Generating external-review prompt as `proposal` (inferred from tasks state)."

#### Scenario: All tasks checked + code exists → infer system mode

- **WHEN** `--as` is omitted, all tasks in `tasks.md` are checked, and the codebase shows changes consistent with apply having run
- **THEN** the command proceeds as `--as system` with an analogous inference note

#### Scenario: Ambiguous state → prompt user

- **WHEN** the inference cannot determine mode unambiguously (e.g., partial implementation)
- **THEN** the command prompts the user via `AskUserQuestion` to choose proposal or system

### Requirement: Self-contained prompt emitted to chat

The system SHALL emit the handoff prompt to chat output (not to a file). The prompt SHALL be complete and copy-paste-ready.

#### Scenario: Prompt structure

- **WHEN** the prompt is emitted
- **THEN** it includes: role description, repo URL or path, project context file pointers (`CLAUDE.md`, `openspec/project.md`, `*_convention.md`, `openspec/lenses/`), iteration history pointer (`.orbit-runs/`), cycle context (iteration N, prior findings count, resolved-since-last), mode-specific "what to read" and "what to look for" sections, output format specification

#### Scenario: Mode-specific sections

- **WHEN** the prompt is emitted in `--as proposal` mode
- **THEN** the "what to look for" section enumerates the 9 review-proposal passes; in `--as system` mode it enumerates the 7 review-system passes

#### Scenario: Reference prompt template

- **WHEN** an implementer generates the handoff prompt
- **THEN** it follows the reference template structure below (the implementer may adjust phrasing for clarity but MUST preserve the named sections and the output-format block verbatim so external-review parsers ingest cleanly):

```markdown
# External Review: <change-name> (iteration <N>)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is
your independent take — be thorough; flag anything that looks wrong,
inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

<repo URL or path>

## Project context (read first)

- `CLAUDE.md` — handoff orientation (if present)
- `openspec/project.md` — project goals + stack (if present)
- `*_convention.md` at repo root — naming, error handling, etc. (if present)
- `openspec/lenses/perspectives.md` — named callers worth validating from (if present)
- `openspec/lenses/critical-paths.md` — user flows worth walking end-to-end (if present)
- `openspec/changes/<change-name>/.orbit-runs/` — iteration history; see what's
  already been addressed in prior cycles

## Cycle context

- Iteration: <N>
- Prior internal findings still open: <count + brief list>
- Prior external findings still open: <count + brief list>
- Resolved since last review: <brief list>

Do not push back on stale findings — pushback discipline is enforced on
resolution, not review. Just flag what you observe.

## What to read for THIS review (<as proposal | system>)

<mode-specific file list>

## What to look for

<mode-specific pass list — 9 passes for proposal mode, 7 for system mode>

## Output format — write to:

`openspec/changes/<change-name>/.orbit-runs/external-<as>-<TS>.md`

(Where <TS> is today's timestamp in ISO format. Pick a fresh timestamp so
this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: <change-name> (iteration <N>)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

## WARNING
...

## SUGGESTION
...

## Notes

<Optional: overall impression, broader concerns.>
```

If your environment doesn't support file writes (chat-only interface), output
the markdown directly and the user will save it.
```

#### Scenario: Output-format block must be verbatim

- **WHEN** the prompt is generated
- **THEN** the "Output format" block (the inner markdown showing the expected findings structure) is included verbatim, including section headers `## CRITICAL` / `## WARNING` / `## SUGGESTION` and the `**File**:` / `**Description**:` field labels; deviations would break `--from-file` parsing

### Requirement: Output format specification in prompt

The system SHALL instruct the external AI to write findings using a defined markdown structure with severity sections.

#### Scenario: Required format

- **WHEN** the prompt is generated
- **THEN** it specifies a markdown structure with `# External Review: <change> (iteration N)` header, `**Reviewer**:` and `**Date**:` fields, and severity sections `## CRITICAL` / `## WARNING` / `## SUGGESTION`, each containing `### <Title>` entries with `**File**: <path>:<line>` and `**Description**: <text>` fields, plus an optional `## Notes` section

#### Scenario: Output file path specified

- **WHEN** the prompt is generated
- **THEN** it instructs the external AI to write findings to `openspec/changes/<change-name>/.orbit-runs/external-<as>-<TS>.md` where `<TS>` is an ISO timestamp

#### Scenario: Chat-only fallback

- **WHEN** the external AI lacks file-write capability
- **THEN** the prompt instructs the AI to output the markdown directly so the user can save it manually

### Requirement: Iteration counting per mode

The system SHALL count iterations separately for proposal-mode and system-mode reviews.

#### Scenario: Iteration N reported

- **WHEN** the command runs and the change's `.orbit-runs/` already contains some `external-<as>-*.md` files for the specified mode
- **THEN** the prompt reports iteration N = (count of matching files) + 1

#### Scenario: First-iteration for mode

- **WHEN** no prior `external-<as>-*.md` files exist for the specified mode
- **THEN** the prompt reports iteration 1 and notes "first external review for this change in <as> mode"

### Requirement: Cycle context populated from prior runs

The system SHALL include a "Cycle context" section in the prompt with iteration number, prior findings (open and resolved), populated from `.orbit-runs/` if it contains prior runs.

#### Scenario: Prior internal review summary exists

- **WHEN** the change has a prior `review-proposal-<TS>.json` or `review-system-<TS>.json` summary in `.orbit-runs/`
- **THEN** the cycle context section includes counts and short titles of those findings

#### Scenario: Prior external review exists

- **WHEN** the change has a prior `external-<as>-<TS>.md` in `.orbit-runs/`
- **THEN** the cycle context section summarizes those findings so the external AI knows what was previously flagged

### Requirement: Repo state validation

The system SHALL warn if the repo has uncommitted changes when `/opsx:review-external` is invoked.

#### Scenario: Uncommitted changes present

- **WHEN** the command runs and `git status` reports uncommitted changes
- **THEN** the chat output includes a warning "Repo has uncommitted changes; external review will be against committed state."

### Requirement: Command does not run the review or ingest findings

The system SHALL NOT execute the review itself nor ingest the external AI's findings.

#### Scenario: Out of scope

- **WHEN** the command completes
- **THEN** only a prompt is emitted; no review passes execute and no findings are processed; ingestion is the responsibility of `/opsx:address-reviews --from-file <path>`
