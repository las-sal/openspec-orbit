# External Review: emit-run-summary-jsons-from-workflow-commands (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-21

## CRITICAL

### Spec names a non-existent `/opsx:ff-change` command
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:8
**Description**: The change repeatedly treats `/opsx:ff-change` as the user-facing command, but the actual command surface is `/opsx:ff` per `.claude/commands/opsx/fast-forward.md:9` and the onboard command reference. This would make the emit scope, command names, and possibly the `command`/filename prefix wrong for one of the in-scope workflow commands. Replace user-facing `/opsx:ff-change` references with `/opsx:ff`, then explicitly decide whether the JSON filename/`command` value should be `ff-<TS>.json`/`"ff"` or intentionally remain `ff-change-<TS>.json`/`"ff-change"` despite the slash-command alias.

### Standalone audit-drift recommendations assume a change context
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:331
**Description**: The requirement covers standalone `/opsx:audit-drift`, but its recommendations require `<name>`, "the same change", and prior per-change `.orbit-runs/*.json` state. Existing audit-drift supports no-argument project-wide standalone mode and writes to `openspec/.orbit-runs/` when no change context exists (`.claude/skills/openspec-audit-drift/references/run-summary-schema.md:3`). A fresh implementer cannot know what to emit for project-wide standalone audit-drift. Split the requirement into change-scoped standalone and project-scoped standalone behavior, defining `change: null`, the project-level output path, and a no-`<name>` address-reviews recommendation for project-wide findings, or explicitly change audit-drift to require a change scope and update the baseline capability.

## WARNING

### Conversation-boundary emit timing depends on future user behavior
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:86
**Description**: The boundary definition includes "the next user message does not continue the same line of work", which the emit-layer cannot know when the AI is about to return control and write the JSON. The named-explore scenario at lines 107-109 has the same problem. Make this operational: emit only on explicit stop/pause or AI-initiated wrap, emit on every no-question handoff and allow later superseding emits, or define a next-turn catch-up rule that writes the prior boundary before continuing.

### Editorial per-kind fields do not fit review-external T0
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-conventions/spec.md:21
**Description**: The universal editorial extension says `review-external` emits include `findings_summary` and `finding_titles`, but the review-external T0 emit is written before findings return and its own requirement defines only `mode`, `prompt_path`, `target`, and `awaiting_findings`. Either make those fields command-specific/optional for editorial emits, or require review-external T0 to emit explicit empty values such as zero-count `findings_summary` and `finding_titles: []` and add that to the T0 requirement.

### Existing emitter harmonization is both non-goal and required work
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/design.md:25
**Description**: The design says harmonizing existing emit-producing capabilities is a non-goal, but task 1.4 requires changing existing review, address-reviews, audit-drift, and archive emit JSONs to add `kind`, and orbit-conventions now requires every emit to carry the universal spine. An implementer following the non-goal could skip the existing emitters and leave the universal spine false in production. Reword this as in-scope migration work, or explicitly exempt legacy existing emitters and document the fallback/migration window.

### Archive summary schema reference is left out of schema alignment
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:20
**Description**: Task 1.2 updates only the three `run-summary-schema.md` references, but archive has its own schema reference at `.claude/skills/openspec-archive-change/references/archive-summary-schema.md`, and the lifecycle spine now requires `kind`, `final_assessment`, and `next_recommended` across emits. Task 1.4 updates the archive SKILL instructions but not the schema reference, so the authoritative archive schema doc would remain stale. Add the archive schema reference to the alignment tasks or state why it intentionally stays separate.

### Crystallization warning overstates `openspec list` visibility
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:145
**Description**: The warning says crystallization makes the change visible to `openspec list`, but crystallized/named explore creates `openspec/explore/<name>/`, while the baseline propose flow later moves that staging directory to `openspec/changes/<name>/`. Since the upstream CLI is unchanged and `openspec list` reports active changes, this warning appears false unless orbit-status or another consumer separately scans `openspec/explore/`. Remove `openspec list` from the consequence text or add a requirement that the relevant consumer actually lists named explorations.

## SUGGESTION

### README task points at a missing schema document
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:82
**Description**: Task 9.6 says to link README to `openspec/conventions/run-summary-emit-schema.md`, but this path does not exist in the repo and task 1.3 now says the canonical example should live inline in the modified orbit-conventions spec. Update the README task to link to `openspec/specs/orbit-conventions/spec.md` and/or the new `orbit-run-summary-emit` capability, or add an explicit task to create the referenced conventions document.

### Verify fail-mode precedence is not explicit
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:273
**Description**: The verify failure modes are listed, but the spec does not say which mode wins when multiple signals are present, such as unchecked tasks plus an `openspec validate` failure. The design sketch implies an order, but the normative requirement should include it. Add a deterministic precedence rule and one scenario for overlapping failures so two implementers classify the same verify output the same way.

## Notes

`openspec validate emit-run-summary-jsons-from-workflow-commands --strict` passes. A grep for `@review:` under the change directory returned no unresolved markers.
