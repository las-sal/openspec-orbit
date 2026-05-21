<!--
Implementation chunks (per D7 / orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (groups 1):    Schema codification + shared spine reference
  Chunk 2 (groups 2-3):  Workflow command emits — explore + propose-shaped
  Chunk 3 (groups 4):    Apply with chunk-end behavior
  Chunk 4 (groups 5):    Verify with mode classification
  Chunk 5 (groups 6-7):  Editorial additions — review-external T0 + standalone audit-drift
  Chunk 6 (groups 8):    Bare-mode explore non-emission + crystallization warning
  Chunk 7 (groups 9):    Integration verification + docs

Total: 9 groups across 7 implementation chunks.
-->

## 1. Schema verification + per-skill schema reference alignment

(Most original schema-codification tasks were resolved by **C1**'s orbit-conventions modification during the proposal review's address-reviews walk. The work remaining in this group is verification + alignment with existing per-skill schema references + a canonical-example embed.)

- [ ] 1.1 Verify `orbit-run-summary-emit` capability's `Workflow-kind emit shape` requirement properly defers to `orbit-conventions`'s MODIFIED `Internal-run JSON summary format` for the universal spine (no redundant redefinition); add cross-references where useful
- [ ] 1.2 (Per **C3** resolution) Update the 3 existing per-skill schema reference files — `.claude/skills/openspec-review/references/run-summary-schema.md`, `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`, `.claude/skills/openspec-address-reviews/references/run-summary-schema.md` — to acknowledge inheritance from orbit-conventions's universal spine. Per-command extension documentation remains per-skill (`findings`, `passes_run`, etc.).
- [ ] 1.3 Embed a canonical example JSON inline in the `orbit-conventions` MODIFIED spec (no separate file) showing the universal spine + sample per-kind extensions for one each of workflow / editorial / lifecycle commands
- [ ] 1.4 (Per **S1** resolution) Update existing emit-producing skill SKILL.md instructions (or `## Orbit additions` sections) for `openspec-review`, `openspec-address-reviews`, `openspec-archive-change`, and inline `openspec-audit-drift` (during archive) to include the `kind` field in their actual emit JSONs (`kind: "editorial"` for review / address-reviews / audit-drift; `kind: "lifecycle"` for archive). Without this, downstream consumers reading `kind` need fallback logic for legacy emits.

## 2. Workflow command emits — explore + propose-shaped (extend existing additions)

- [ ] 2.1 Extend `openspec-explore/SKILL.md`'s existing `## Orbit additions` section with emit instructions for named-mode explore (write `openspec/explore/<name>/.orbit-runs/explore-<TS>.json` at session end with spine + extensions `mode`, `decisions_captured`, `open_questions_count`, `crystallized_to_name`)
- [ ] 2.2 In the same SKILL.md, document the maturity-based `next_recommended` rule (D11: early/mid/mature recommendations based on `decisions_captured` count and open-question count)
- [ ] 2.3 Extend `openspec-propose/SKILL.md`'s `## Orbit additions` with emit instructions (write `propose-<TS>.json` at session end with spine + extensions `artifacts_created`, `delta_count`, `from`) and the D12 recommendation rule (next: `/opsx:review <name>`)

## 3. Workflow command emits — propose-shaped variants (add new orbit-additions)

- [ ] 3.1 Add a new `## Orbit additions` section to `openspec-new-change/SKILL.md` with emit instructions (write `new-<TS>.json`) and D12 recommendation
- [ ] 3.2 Add `## Orbit additions` to `openspec-continue-change/SKILL.md` with emit instructions (write `continue-<TS>.json`) and the artifact-completion-aware D12 recommendation (artifacts complete → review; incomplete → continue with next missing artifact identified)
- [ ] 3.3 Add `## Orbit additions` to `openspec-ff-change/SKILL.md` with emit instructions (write `ff-change-<TS>.json`) and D12 recommendation
- [ ] 3.4 Verify per-variant filename convention is honored (each variant uses its own command name prefix, NOT `propose-<TS>.json`)

## 4. Apply with chunk-end behavior

- [ ] 4.1 Add `## Orbit additions` to `openspec-apply-change/SKILL.md` with the chunk-detection rule (parse the tasks.md preamble comment block for `Chunk N (groups X[-Y]): <name>` lines per the format specified in `Requirement: Apply per-chunk-end emission`). Note: chunks MAY span multiple groups (e.g., `Chunk 2 (groups 2-3)`); the emit fires when the LAST group in the chunk completes its tasks, not on every group boundary (per **S6**).
- [ ] 4.2 Document chunk-end emit logic (D7 rule 1: emit when last task in chunk N is checked, with `chunk: "N of M"`, `chunk_complete: true`, `chunk_name: <name>`)
- [ ] 4.3 Document mid-chunk session-pause emit logic (D7 rule 2: emit on explicit pause signals with `chunk_complete: false` for resumability)
- [ ] 4.4 Document no-chunking apply emit logic (D7 rule 3: emit once at session end with `chunk: null`)
- [ ] 4.5 Document per-command extensions (`tasks_completed`, `tasks_remaining`, `chunk`, `chunk_name`, `chunk_complete`, `tasks_completed_this_session`)
- [ ] 4.6 Document the `next_recommended` advancement rules (next chunk on N<M, `/opsx:verify <name>` on N==M, resume current chunk on mid-chunk pause)

## 5. Verify with mode classification

- [ ] 5.1 Add `## Orbit additions` to `openspec-verify-change/SKILL.md` with emit instructions (write `verify-<TS>.json` with spine + extensions `verdict`, `findings_count`, `findings_summary`)
- [ ] 5.2 Document the verify-pass recommendation (D9: leading `/opsx:review --as system <name>` with `/opsx:archive <name>` surfaced in reason text)
- [ ] 5.3 Document the verify fail-mode classification logic that lives in the emit-layer (NOT in verify itself): tasks-incomplete → mode ①, impl-vs-spec gap → mode ②, openspec-validate failure → mode ③
- [ ] 5.4 Document the per-mode recommendations (D13: mode ① → `/opsx:apply`, mode ② → `/opsx:review --as system --mark`, mode ③ → verbatim validator message)
- [ ] 5.5 Document the warn-state recommendation (`/opsx:review --as system` with warning count in reason)
- [ ] 5.6 Explicitly note in the SKILL.md additions that verify's upstream behavior is unchanged — emit-layer is observability-only (cross-reference `Requirement: Emit-layer wraps upstream skills without modifying them`)

## 6. Editorial: review-external T0 emit

- [ ] 6.1 Extend `openspec-review-external/SKILL.md` with T0 emit instructions (write `review-external-<TS>.json` at the moment the prompt is packaged, before findings return)
- [ ] 6.2 Document the per-command extensions for review-external (`mode`, `prompt_path`, `target`, `awaiting_findings`)
- [ ] 6.3 Document the multi-step prose `next_recommended` (D14: "Paste ... save ... then /opsx:address-reviews <name> --from-file <path>") — note that orbit-status's tier-1 parse will find no leading slash command and leave `command`/`args` null, preserving full prose in `reason`

## 7. Editorial: standalone audit-drift emit

- [ ] 7.1 Extend `openspec-audit-drift/SKILL.md` with standalone-mode emit instructions (write `audit-drift-<TS>.json` when the command runs standalone — NOT for the inline-during-archive case, which remains captured in `archive-<TS>.json`)
- [ ] 7.2 Document the per-command extensions (`categories_run`, `findings_by_category`, `findings_total`)
- [ ] 7.3 Document the with-findings recommendation (D10: `/opsx:address-reviews <name> --from-file <this-json>`)
- [ ] 7.4 Document the clean-findings defer-to-prior recommendation (D10: copy `next_recommended` verbatim from the most recent prior `.orbit-runs/*.json` for the same change, excluding the just-written audit-drift JSON; set `final_assessment` to note the deferral)

## 8. Bare-mode explore non-emission + crystallization warning

- [ ] 8.1 Refine `openspec-explore/SKILL.md`'s `## Orbit additions` to explicitly state that bare-mode (no name argument) does NOT emit any `.orbit-runs/` JSON
- [ ] 8.2 Codify the crystallization warning text verbatim in the SKILL.md additions, including all 4 consequences (new directory created, audit trail starts, visibility to consumers, abandonment requires formal archive)
- [ ] 8.3 Document the confirmation gate (AI MUST wait for explicit user confirmation before creating `openspec/explore/<name>/explore.md` or emitting the first `explore-<TS>.json`)
- [ ] 8.4 Add scope-enforcement notes in `openspec-bulk-archive-change/SKILL.md` and `openspec-onboard/SKILL.md` clarifying these commands do NOT emit run-summary JSONs (defensive — explicit scope statement prevents future confusion). `openspec-sync-specs` is excluded from scope here because it is slated for removal by [openspec-orbit#6](https://github.com/las-sal/openspec-orbit/issues/6); editing it would be wasted work.

## 9. Integration verification + docs

- [ ] 9.1 Live-test end-to-end: run `/opsx:propose foo` on a throwaway test change; verify `propose-<TS>.json` appears in `openspec/changes/foo/.orbit-runs/` with all 6 spine fields populated and `next_recommended` parseable to `/opsx:review foo`
- [ ] 9.2 Live-test end-to-end: run `/opsx:apply foo` on a chunked change; verify per-chunk-end emits appear with correct `chunk` values and recommendations
- [ ] 9.3 Live-test end-to-end: run `/opsx:verify foo` (pass case); verify the JSON's `next_recommended` parses correctly and `/opsx:archive` is mentioned in reason text
- [ ] 9.4 Cross-repo integration test: in `~/code/orbit-status`, run `opsx-status` against a change with new workflow-command JSONs present; verify tier-1 reader picks them up and surfaces recommendations correctly (no orbit-status code change required)
- [ ] 9.5 Backward-compatibility check: run `/opsx:review`, `/opsx:address-reviews`, `/opsx:archive` (existing emitters); verify their existing JSON shapes are unchanged (no regressions from this change)
- [ ] 9.6 Update orbit's `README.md` to document the new emit behavior (one-paragraph summary + link to `openspec/conventions/run-summary-emit-schema.md`)
- [ ] 9.7 Assess whether `CLAUDE.md` needs a new convention note about the emit pattern; add if useful
- [ ] 9.8 Run `openspec validate emit-run-summary-jsons-from-workflow-commands --strict` to confirm spec validity
- [ ] 9.9 (User-validation handoff) User runs `/opsx:verify emit-run-summary-jsons-from-workflow-commands` and confirms all artifacts match implementation
