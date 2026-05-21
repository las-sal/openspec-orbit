# External Review: emit-run-summary-jsons-from-workflow-commands (system mode, iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-21

## CRITICAL

None.

## WARNING

### `/opsx:new` recommendation contract is split between premature review and continue
**File**: .claude/skills/openspec-new-change/SKILL.md:105
**Description**: The implementation emits `next_recommended: "/opsx:continue <name>..."`, while the new `orbit-run-summary-emit` spec says `/opsx:new` is propose-shaped and SHALL emit a leading `/opsx:review <name>` recommendation (`openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:199`). The skill also records `artifacts_created: ["proposal"]` even though the upstream `/opsx:new` command stops after scaffolding and showing the first artifact template; it does not guarantee proposal/design/tasks/specs exist. This leaves future implementers with incompatible contracts: follow the spec and recommend review before artifacts exist, or follow the skill and violate the spec. Recommendation: split `/opsx:new` out of the "propose-shaped always review" rule, define its recommendation from `openspec status --change <name> --json` like `/opsx:continue`, and update the JSON example/artifacts fields to match scaffold-only default behavior.

### Change-scoped `/opsx:audit-drift <name>` has no public command surface
**File**: .claude/commands/opsx/audit-drift.md:16
**Description**: The new emit spec defines two standalone audit-drift modes, including change-scoped `/opsx:audit-drift <name>` with a change-local emit path and recommendation (`openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:365`). The public command file still declares only `/opsx:audit-drift [flags]` and immediately says "No positional arguments." The top of `.claude/skills/openspec-audit-drift/SKILL.md` likewise describes optional flags only, so a user or fresh AI following the shipped command surface will not know that `<name>` is accepted. Recommendation: update the command body and the skill's primary Input/context-resolution sections to accept optional `[<change-name>] [flags]`, then route no-arg standalone to project-wide behavior and named standalone to the change-scoped emit/recommendation path.

### Verify fail-mode 2 recommends system `--mark`, but system `--mark` is ignored
**File**: .claude/skills/openspec-verify-change/SKILL.md:232
**Description**: The verify emit says impl-vs-spec failures should recommend `/opsx:review --as system <name> --mark` and promises that system review will surface markers for `/opsx:address-reviews`. The archived `orbit-review` contract says `--mark` is meaningful only in proposal mode, and that `--as system --mark` is accepted but ignored (`openspec/specs/orbit-review/spec.md:183`). The current review skill repeats that system-mode `--mark` is ignored. This means the recommended path does not create the markers it says it will create, so address-reviews has nothing marker-based to consume. Recommendation: either implement and spec system-mode marker writing in this change, or remove `--mark` from the verify recommendation and route users to a supported resolution path, such as review-system findings plus a parser-supported `--from-file` bridge when available.

### Final apply emit marks the final chunk complete while tasks remain
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/apply-2026-05-21T04-03-28Z.json:12
**Description**: The final apply dogfood emit has `tasks_completed: 38`, `tasks_remaining: 6`, and `chunk_complete: true`, then recommends `/opsx:verify`. The apply emit spec defines `chunk_complete: true` as the state reached "when the last task in chunk N is checked" and advances to verify on apply-complete (`openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:241`). The JSON explains that the six remaining tasks are user-validation tasks, but the schema has no machine-readable exception for "AI-doable tasks complete." Downstream consumers reading `chunk_complete` and `tasks_remaining` get contradictory state. Recommendation: either keep `chunk_complete: false` until all tasks are checked, or explicitly extend the apply schema with a field such as `user_validation_remaining` / `ai_applicable_tasks_complete` and document how such tasks affect `next_recommended`.

### Review-external T0 dogfood emit violates the no issue-reference rule
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/review-external-2026-05-21T04-15-40Z.json:5
**Description**: The new `No forward-look at other-issue future behavior changes` requirement says that when any run-summary JSON is inspected, `next_recommended` and `final_assessment` contain no issue numbers, with `#20` called out as an example (`openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:424`). The newly written review-external T0 JSON is the in-change dogfood emit for this feature, but its `final_assessment` says "per #20 evidence." Recommendation: rewrite this emitted `final_assessment` to remove the issue number/future-issue reference, and add a lightweight validation grep before committing new run-summary JSONs so the first dogfood artifact satisfies the rule it is proving.

## SUGGESTION

### Timestamp field documentation mixes JSON ISO time with filename-safe time
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-conventions/spec.md:11
**Description**: The universal spine says the JSON `timestamp` field is ISO-8601 UTC with format `YYYY-MM-DDTHH-MM-SSZ`, but the canonical JSON examples use colon-separated ISO values such as `2026-05-21T13:34:12Z` at line 31 and existing emitted JSONs follow that same colon format. The hyphenated `HH-MM-SSZ` form is the filename-safe `<TS>` token, not the JSON timestamp. Recommendation: explicitly separate the two concepts: JSON `timestamp` should be `YYYY-MM-DDTHH:MM:SSZ`, while filenames use a colon-replaced `<TS>` such as `YYYY-MM-DDTHH-MM-SSZ`; update the per-skill schema references that repeat the same wording.

### Design still rejects the implementation mechanism now used by the change
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/design.md:44
**Description**: D-arch-1 says the rejected alternative is "Modify each upstream skill's `## Orbit additions` section to embed emit logic," specifically because apply and verify had no such sections. The completed tasks and implementation now add exactly those `## Orbit additions` sections to upstream-derived skills including new/continue/ff/apply/verify. The spec now permits this pattern as long as the upstream body above the marker remains untouched, so the design rationale is stale rather than intentionally different. Recommendation: revise D-arch-1 to state that `## Orbit additions` are the wrapper boundary and the upstream-authored body remains unchanged, or add a real separate wrapper artifact if the rejected alternative is still meant to be rejected.

## Notes

Validation run during this review:

- `openspec validate emit-run-summary-jsons-from-workflow-commands --strict` passed.
- `openspec validate --all --strict` passed: 9 items, 0 failed.
- `git diff --check 0de2de9..HEAD -- .claude/ openspec/changes/emit-run-summary-jsons-from-workflow-commands/` passed.
