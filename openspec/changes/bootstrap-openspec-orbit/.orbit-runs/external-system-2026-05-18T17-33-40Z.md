# External Review: bootstrap-openspec-orbit (iteration 1, system mode)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-18

## CRITICAL

None.

## WARNING

### Review command points users at a slash command that does not exist
**File**: .claude/commands/opsx/review.md:45
**Description**: The system-mode review surface says Pass 0 delegates to `/opsx:verify-change`, and the orbit-review spec repeats that invocation at `openspec/changes/bootstrap-openspec-orbit/specs/orbit-review/spec.md:53`. The shipped upstream slash command is `.claude/commands/opsx/verify.md`, whose documented invocation is `/opsx:verify <name>`; there is no `.claude/commands/opsx/verify-change.md`. The review SKILL itself uses the correct form at `.claude/skills/openspec-review/SKILL.md:167`, so this is command/spec drift. Replace `/opsx:verify-change <change-name>` with `/opsx:verify <change-name>` wherever the slash command is meant, and reserve `openspec-verify-change` for the skill name.

### Archive summary paths omit upstream's date-prefixed archive directory
**File**: .claude/skills/openspec-archive-change/SKILL.md:183
**Description**: The orbit archive addition tells the agent to write the summary under `openspec/changes/archive/<change-name>/.orbit-runs/archive-<TS>.json`, but the preserved upstream archive step moves the change to `openspec/changes/archive/YYYY-MM-DD-<name>` at `.claude/skills/openspec-archive-change/SKILL.md:74` and `.claude/skills/openspec-archive-change/SKILL.md:81`. The same non-date-prefixed path appears in the archive spec (`openspec/changes/bootstrap-openspec-orbit/specs/orbit-archive-modifications/spec.md:57`, `:70`, `:93`), README (`README.md:449`, `:681`), and design (`openspec/changes/bootstrap-openspec-orbit/design.md:128`). This can make the archive run summary and already-archived checks target a directory that upstream never creates. Update the orbit docs/contracts to use `openspec/changes/archive/YYYY-MM-DD-<change-name>/.orbit-runs/` or explicitly define how `<change-name>` resolves to the dated archive target after the move.

### Address-reviews flag surface is inconsistent across README, command, SKILL, and spec
**File**: README.md:400
**Description**: The README advertises `/opsx:address-reviews` with `--from-file`, `--list`, `--only`, and `--keep-resolved-markers`. The slash command advertises `--only` and `--keep-resolved-markers` but not `--list` (`.claude/commands/opsx/address-reviews.md:20`), while the SKILL only defines a positional `<scope>` and `--from-file` ingest (`.claude/skills/openspec-address-reviews/SKILL.md:15`, `:50`, `:58`) and never specifies `--only` or `--list` behavior. The spec likewise covers scoped positional scans and `--from-file`, not `--only` or `--list`. Either implement/document `--only` and `--list` in the SKILL and spec, or remove them from the public command reference so users do not pass flags that the command body has no contract to honor.

### External findings template and parser contract do not agree on empty or abbreviated severity sections
**File**: .claude/skills/openspec-review-external/references/prompt-template.md:71
**Description**: The prompt template says the output-format block must be preserved because `address-reviews --from-file` parses it, but it uses `...` under WARNING and SUGGESTION instead of repeating the required `###` / `**File**:` / `**Description**:` fields. The parser contract is strict on those field labels (`.claude/skills/openspec-address-reviews/references/external-findings-format.md:68`) and does not specify how to handle `None.` for an empty severity section, even though prior external findings use `None.` under CRITICAL. This leaves the cross-AI loop dependent on reviewer interpretation rather than a deterministic parse contract. Replace the ellipses with a complete repeated finding shape for every severity and explicitly document the accepted empty-section spelling, e.g. `None.` as a no-finding sentinel.

## SUGGESTION

### README install verification expects metadata that the shipped modified skills do not expose
**File**: README.md:909
**Description**: The verify step says the three modified upstream skills should show `openspec-orbit` in `metadata.author`, but the shipped modified skills still have `metadata.author: openspec` (`.claude/skills/openspec-explore/SKILL.md:7`, `.claude/skills/openspec-propose/SKILL.md:7`, `.claude/skills/openspec-archive-change/SKILL.md:7`). If preserving upstream metadata is intentional, change the README verification to check for the appended "Orbit additions" sections instead. If orbit authorship is desired, update the metadata in the three modified SKILL.md files.

## Notes

Pass 0 structural spot-check: `openspec validate bootstrap-openspec-orbit` passes. The task split matches the prompt's expected shape: 84 checked tasks and 18 unchecked tasks, with the unchecked work annotated as integration/release/user-driven. The eight capability spec directories are present and map to the four new skills plus the three upstream skill additions and distributed conventions surface.

Pass 1 baseline compliance skipped: `openspec/specs/` is empty, which matches the bootstrap context. Passes 4 and 5 skipped: `openspec/lenses/` is absent, also expected for bootstrap. Drift/audit degenerate categories for archived deltas and archive coherence were skipped because `openspec/changes/archive/` is absent; the main useful drift signal is the cross-doc consistency findings above.

The three execution disciplines are embedded in the four new orbit SKILL.md files. The modified upstream SKILL.md files preserve upstream content and append orbit additions after the original content.
