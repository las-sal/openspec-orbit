# External Review: address-reviews-auto-discovers-internal-json (iteration 1, proposal mode)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-27

## CRITICAL

None.

## WARNING

### Archived change-name resolution is underspecified
**File**: openspec/changes/address-reviews-auto-discovers-internal-json/specs/orbit-address-reviews/spec.md:9
**Description**: The spec says a positional argument is a change name if it resolves to `openspec/changes/archive/<YYYY-MM-DD>-<name>/`, but the user invokes `/opsx:address-reviews <change-name>` without knowing the archive date prefix. The proposal/design/tasks never define the resolver algorithm for archived changes: whether active exact matches win over archive matches, how to scan date-prefixed archive directories, what happens if multiple archived directories match the same suffix, and what error/no-discovery behavior applies when the match is ambiguous. As written, a fresh implementer can only implement active-change discovery deterministically, while the archived-change scenario at lines 56-59 depends on an unstated lookup rule. Add a normative resolver rule and mirror it in task 1.1/SKILL prose, for example: check `openspec/changes/<name>/` first; then enumerate `openspec/changes/archive/` entries matching `^\d{4}-\d{2}-\d{2}-<name>$`; define latest-date vs ambiguity behavior; then treat non-matches as path/pattern scope.

### Resolution-log schema does not require the promised auto-discovery evidence
**File**: openspec/changes/address-reviews-auto-discovers-internal-json/tasks.md:42
**Description**: The design promises that the resolution log captures source path, source command, and recency-comparison evidence, and also says stale JSON is acceptable because the log records source-JSON and latest-apply timestamps. The implementation task here only adds `"auto-discovered"` to the existing `source` enum and field-note prose. The delta spec likewise only requires `source: "auto-discovered"`, so an implementer can satisfy the spec/tasks while omitting the selected JSON path, the selected file's `command`, the filename timestamp that won, the compared candidates, and the latest `apply-*.json` timestamp. That breaks the audit-trail mitigation used to justify both D-recency-1 and D-no-stale-detection. Extend the spec/tasks/run-summary schema with explicit fields or prose requirements for `source_path` as the selected JSON path, the selected source command, the winner timestamp/tie-break rationale, and the latest apply timestamp when present.

### Internal findings reference still says only review JSON is valid
**File**: .claude/skills/openspec-address-reviews/references/internal-findings-format.md:119
**Description**: The current parser reference already says audit-drift JSON is accepted in its top matter and command discriminator, but the strictness section still says the top-level `command` field "must be exactly `"review"` (case-sensitive) for v1." Task 2.1 only asks the implementer to add a new top sentence saying auto-discovery also consumes this parser contract, so this stale strictness bullet can survive the change. Because auto-discovery explicitly chooses among both `review-<mode>-*.json` and `audit-drift-*.json`, leaving this contradiction in the parser contract can make a fresh implementer reject the very audit-drift candidate the spec says to discover. Broaden task 2.1 to fix the strictness bullet to `"review"` OR `"audit-drift"` and sweep the reference for similar v1-only-review residue.

### `--from-file` plus change scope semantics are not byte-identical or unambiguous
**File**: openspec/changes/address-reviews-auto-discovers-internal-json/specs/orbit-address-reviews/spec.md:39
**Description**: This scenario says that when a user supplies both `<change-name>` and `--from-file <path>`, auto-discovery does not run, but marker grep still happens and any change-scope markers are presented alongside the `--from-file` virtual markers. The existing baseline and skill describe discovery as scanning markers OR ingesting `--from-file`; the design decision says explicit `--from-file` uses that path verbatim and preserves existing behavior. The parenthetical therefore either changes behavior by making explicit file ingest additive with scope markers, or it documents a current behavior that is not actually stated in the baseline contracts. Clarify whether `--from-file` is exclusive or additive when combined with a scope. If additive is intended, add a baseline-aligned scenario and SKILL/task updates for mixed marker plus file input; if exclusive is intended, remove the parenthetical so explicit user intent really wins.

## SUGGESTION

None.

## Notes

`openspec validate address-reviews-auto-discovers-internal-json --strict` passed. `openspec validate --all --strict` also passed. The change is structurally valid; the warnings above are codegen-readiness and contract-coherence gaps that validation will not catch.
