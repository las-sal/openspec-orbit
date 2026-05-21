# Internal Review (rendered as findings for `--from-file`): emit-run-summary-jsons-from-workflow-commands (iteration 1)

**Reviewer**: Claude Opus 4.7 (in-context, proposal mode, 9-pass `/opsx:review`)
**Date**: 2026-05-21

> **Note**: This file is a markdown rendering of `.orbit-runs/review-proposal-2026-05-21T00-18-14Z.json` to satisfy the current `--from-file` parser contract (markdown-only per `.claude/skills/openspec-address-reviews/references/external-findings-format.md`). Tracked as a v0.1 bug: [openspec-orbit#4](https://github.com/las-sal/openspec-orbit/issues/4) (address-reviews should accept internal-review JSON natively). Until #4 ships, this file is the bridge.

## CRITICAL

### C1: Universal-emit-shape requirement conflicts with orbit-conventions "Internal-run JSON summary format"
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:34
**Description**: `openspec/specs/orbit-conventions/spec.md:159` already defines an `Internal-run JSON summary format` requirement with minimum fields `command, timestamp, change, iteration (if applicable), findings_summary, finding_titles`. My new `Universal emit shape` requirement requires a different spine: `command, timestamp, change, final_assessment, next_recommended, kind`. Both requirements govern internal-run JSON shape; they require different field sets. The orbit-conventions scenario reads "review/audit/archive command" (doesn't bind workflow commands), but the requirement title implies broader coverage. **Recommendation**: Modify orbit-conventions as part of this change. Add `orbit-conventions` to the proposal's Modified Capabilities section. Define a unified minimum spine with per-kind extensions.

### C2: `openspec/conventions/` path doesn't exist and violates orbit-conventions convention-file location
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:17
**Description**: Tasks 1.1, 1.5, 1.6 reference `openspec/conventions/run-summary-emit-schema.md` and `openspec/conventions/examples/`. (a) The directory `openspec/conventions/` does not exist. (b) `orbit-conventions/spec.md:243` explicitly requires convention files at PROJECT ROOT (e.g., `naming_convention.md`), NOT under `openspec/` or in a subdirectory. The tasks as written violate an existing requirement. **Recommendation**: Two valid options — (1) move the schema doc to project root as `run_summary_emit_convention.md` per orbit-conventions naming pattern, OR (2) document the schema directly in the orbit-conventions spec via a MODIFIED requirement. Option (2) is cleaner — consolidates the spine definition in one place. Either way, remove `openspec/conventions/` references from tasks.

### C3: Existing `run-summary-schema.md` reference files aren't acknowledged or coordinated with
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/proposal.md:23
**Description**: Three schema reference files already exist: `.claude/skills/openspec-review/references/run-summary-schema.md`, `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`, `.claude/skills/openspec-address-reviews/references/run-summary-schema.md`. They define per-skill emit shapes today. My proposal doesn't mention them and claims "no modified capabilities." Either they should be updated to reference the new universal spine, or they should be explicitly noted as "parallel definitions accepted for v1; harmonization deferred." Currently they're invisible — a missed scope item. **Recommendation**: Add a "Schema reference files" subsection to design.md's Migration Plan listing the 3 references and how this change relates to them (update / supersede / leave as-is). Add tasks under group 1 or 9 to either edit or cross-reference these files. If choosing "leave as-is for v1," say so explicitly so reviewers know it's a deliberate scope decision.

### C4: Proposal claims standalone /opsx:audit-drift is a NEW emit, but it already emits per existing schema reference
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/proposal.md:9
**Description**: Proposal line 9 says "/opsx:audit-drift (inline audit-drift during archive is already captured in archive-<TS>.json — unchanged)" and lists standalone audit-drift as "editorial additions" — implying it's a new emit. But `.claude/skills/openspec-audit-drift/references/run-summary-schema.md` documents standalone-mode emit today: "Standalone (no change context): openspec/.orbit-runs/audit-drift-<TS>.json". The actual gap this change addresses for standalone audit-drift is the defer-to-prior-on-clean recommendation logic (D10), NOT the emit itself. **Recommendation**: Revise proposal.md and spec.md to clarify what's actually NEW for standalone audit-drift: the recommendation logic (D10 defer-to-prior-on-clean), the spine field alignment (`kind: 'editorial'`), and possibly path standardization. The emit itself is not new — refine claims accordingly. Also revisit whether standalone audit-drift's path (`openspec/.orbit-runs/`) needs to align with change-scoped path conventions.

### C5: `final_assessment` and `next_recommended` are de facto fields not formally specified anywhere; spec/design imply they're already-canonical
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:42
**Description**: Grep across orbit baseline specs shows `final_assessment` and `next_recommended` have ZERO matches. They exist in actual emitted JSONs (verified via .orbit-runs/ from archived changes) and orbit-status reads them (per `orbit-status-recommendation/spec.md:7`). But the existing schema reference files don't include them, and orbit-conventions doesn't mention them. My spec treats them as established when they're actually de facto extensions. **Recommendation**: Either (a) treat this change as formally introducing `final_assessment` and `next_recommended` as canonical fields (which means modifying orbit-conventions and the 3 existing schema references to add them), OR (b) explicitly say in proposal.md that this change codifies what was previously de facto — and update the existing schema references as part of the work. Option (a) is more honest; either way the proposal needs to acknowledge the gap.

## WARNING

### W1: Proposal claims "no modified capabilities" but the work needs to modify orbit-conventions and likely the 3 existing schema reference files
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/proposal.md:23
**Description**: Proposal lines 23-25 say "Modified Capabilities: None for v1." This contradicts: (a) C1's resolution requires modifying orbit-conventions; (b) C3's resolution likely requires updating existing schema reference files; (c) tasks.md line 21 explicitly mentions "Reference the schema doc from orbit-conventions capability spec where relevant" which is a modification. **Recommendation**: Update proposal.md to list `orbit-conventions` (and any other affected existing capabilities) as Modified Capabilities. Expand tasks.md group 1 to include the modification work explicitly.

### W2: Design's "kind is the only net-new field" claim is incorrect
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/design.md:59
**Description**: design.md line 59 says: "Adding `kind` explicitly is the only net-new field; everything else codifies what existing emits already do." But `final_assessment` and `next_recommended` are NOT in the existing schema reference files (per C5) — they're de facto in actual JSONs but not formally documented. So they're effectively net-new in spec terms even though present in practice. **Recommendation**: Reword the claim to: "The `kind` field is structurally new; `final_assessment` and `next_recommended` codify existing de facto fields into formal spec, requiring updates to the 3 existing schema reference files."

### W3: "Session end" / "session pause" timing semantics are fuzzy across artifacts; underspecified for codegen
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md (multiple) + tasks.md:20
**Description**: tasks.md line 20 mentions "emit at session end" for non-chunked commands. spec.md line 109 (bare-mode explore scenario) says "converses for 30 minutes without crystallizing." design.md line 132 (risk section) says "session pause = AI returns control to user without a chunk_complete: true emit in the same turn." These signals are scattered and not formally defined. An implementer would have to invent semantics. For workflows like /opsx:explore (multi-turn conversation) the boundary is especially fuzzy. **Recommendation**: Add a `Requirement: Session-end and session-pause semantics` to the spec, defining: (1) what "session end" means for one-shot commands (e.g., /opsx:propose, /opsx:verify — natural completion); (2) what it means for multi-turn commands (e.g., /opsx:explore, /opsx:apply — explicit user pause signal or N turns of idle); (3) which signals trigger emit vs which don't. Without this, implementation will be heuristic and inconsistent.

### W4: review-external T0 timing assumes a clean prompt-packaging moment; needs validation against current review-external skill flow
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:240
**Description**: spec.md line 240+ assumes /opsx:review-external has a clean T0 moment when the prompt is packaged, before findings return. But /opsx:review-external is itself a multi-step AI-driven process — the AI might package the prompt as part of a longer conversation, not as a discrete one-shot. The proposal doesn't verify what the actual current skill flow looks like. If T0 is fuzzy, emit timing becomes fuzzy. **Recommendation**: Read `.claude/skills/openspec-review-external/SKILL.md` during apply phase to verify the T0 moment is well-defined. If it's not a discrete moment, add either a Requirement clarifying when emit fires, or a Risk note acknowledging the fuzziness.

### W5: Task 8.4 references /opsx:sync-specs SKILL.md which is being deleted by #6
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:73
**Description**: Task 8.4 says "Add scope-enforcement notes in openspec-bulk-archive-change/SKILL.md, openspec-onboard/SKILL.md, and openspec-sync-specs/SKILL.md." But the proposal (line 15) explicitly excludes sync-specs because it's slated for removal by #6. Editing a file that's about to be deleted is wasted work; if this change ships before #6, the edit will need re-removal. **Recommendation**: Remove openspec-sync-specs from task 8.4. The scope statement in spec.md already documents the exclusion.

### W6: Task 1.6 references `openspec/conventions/examples/` path that doesn't exist and likely shouldn't
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md:22
**Description**: Task 1.6 says "Add a canonical-emit example fixture under openspec/conventions/examples/run-summary-emit-example.json". Same root cause as C2 — `openspec/conventions/` is not a valid orbit path per the convention-file location requirement. Also, the "examples/" subdirectory introduces a new convention pattern not established elsewhere in orbit. **Recommendation**: Drop task 1.6 (the canonical-emit example fixture). Instead, embed the canonical example inline in the convention's documentation (whether that's orbit-conventions spec or a project-root convention file).

### W7: Terminology drift: "run-summary JSON" / "emit JSON" / "internal-run JSON summary" used interchangeably across artifacts
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands (all artifacts)
**Description**: `orbit-conventions/spec.md:159` calls it "Internal-run JSON summary format". Existing schema reference files use "<command> run-summary schema". My capability is named "orbit-run-summary-emit". My spec uses "run-summary JSON" and "emit JSON" interchangeably. design.md uses "emit JSON", "JSON summary", "run-summary JSON". Three+ terms for the same thing. **Recommendation**: Standardize on "run-summary JSON" (matches existing schema reference titles and the new capability name). Update orbit-conventions' "Internal-run JSON summary format" requirement title to "Run-summary JSON format" for consistency.

## SUGGESTION

### S1: Without harmonization, downstream consumers reading `kind` need fallback logic for legacy emits missing the field
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:47
**Description**: Spec scopes universal-emit-shape to commands in scope. Existing emits (review, address-reviews, archive) won't have `kind`. Downstream consumers (orbit-status's tier-1, future dashboards) reading `kind` for routing will need fallback logic: "if kind missing, infer from filename prefix." This fragmentation is workable but not great. **Recommendation**: If keeping the no-harmonization decision: add explicit guidance in the spec for consumers — "kind MAY be absent on JSONs from existing capabilities (review, address-reviews, archive) until a future harmonization change lands; consumers SHOULD fall back to filename prefix routing in that case." Even better: file a follow-up issue for harmonization explicitly tracked.

### S2: tasks.md preamble parsing for chunk detection — format not formally specified
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:161
**Description**: spec.md line 161+ says apply emits when "chunks are explicitly declared in the change's tasks.md preamble." Task 4.1 says "parse the tasks.md preamble comment block for `Chunk N (groups X-Y): <name>` lines". But this preamble format isn't formally specified anywhere — neither in orbit-conventions nor in this spec. An implementer would have to reverse-engineer the format from existing examples (this change's own tasks.md preamble being one). **Recommendation**: Add a `Requirement: tasks.md chunk preamble format` to either this spec or orbit-conventions, defining the exact regex/structure. Without it, implementation is brittle.

### S3: explore.md has structural cruft from incremental edits during exploration (D15 misplaced under "Open questions" header; orphaned Q1/Q3 text)
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/explore.md:256
**Description**: explore.md line 256 has "## Open questions" header but contains D15 (a decision, not a question). Lines 268-289 have leftover text from earlier Q1/Q3 sections that were closed but not fully cleaned. Doesn't change substance but creates confusion for future readers. **Recommendation**: Quick cleanup pass on explore.md before apply: move D15 to be with the other D-decisions; remove orphaned closed-Q text. ~5-minute fix. Not blocking but worth doing while context is fresh.

### S4: Missing scenario: explore named-mode at session end with 0 decisions captured
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:120
**Description**: Named-mode explore maturity rules cover 0-1 / 2-3 / 4+. The 0-decision case is included in "Early" but has no explicit scenario. If a user starts /opsx:explore foo and the session ends without capturing any decisions (just discussion), the recommendation behavior should be tested. **Recommendation**: Add a scenario: "WHEN named-mode explore emits with decisions_captured: 0; THEN next_recommended begins with /opsx:explore <name> and contains language acknowledging that thinking is just beginning."

### S5: Missing scenarios for verify-fail mode-② error paths (e.g., concurrent task modification, partial verify run)
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md:215
**Description**: spec mode-② requires classifying impl-vs-spec gap. But what if verify partially completes (e.g., timeout, transient failure)? The recommendation logic only covers clean pass/fail/warn — not "verify aborted before completing classification." Edge case worth specifying. **Recommendation**: Add a scenario for partial-verify failure or note as out-of-scope. Current spec is silent.

### S6: Task count (35) and chunk count (7) — verify chunk groupings match implementation reality
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md (preamble)
**Description**: tasks.md has 9 task groups across 7 chunks (per preamble). Chunk 2 spans groups 2-3, chunk 5 spans groups 6-7. The chunk-to-group mapping is a design choice that affects how D7's chunk-end emits behave when applying. Worth verifying the mapping is intentional (some chunks span multiple groups; that's fine but worth understanding). **Recommendation**: Confirm during apply that chunk boundaries trigger emits correctly when crossing group boundaries (e.g., chunk 2 ends after group 3 is complete, not after group 2). Add this to task 4.1 or 4.2 as an explicit implementation note.

### S7: Design doesn't reference the bootstrap-openspec-orbit archived change's design.md or sketches
**File**: openspec/changes/emit-run-summary-jsons-from-workflow-commands/design.md
**Description**: The 2026-05-18-bootstrap-openspec-orbit archive established orbit-conventions, the schema reference files, and the broader emit architecture. My design.md treats this work as if there's no architectural prior art, except for cross-referencing orbit-status's spec. Worth referencing the prior change for context. **Recommendation**: Add a "Prior art" or "Related work" note in design.md Context section linking to the bootstrap archive. Helps future readers understand the lineage.

## Notes

This file is a one-time markdown rendering of `.orbit-runs/review-proposal-2026-05-21T00-18-14Z.json` for `--from-file` compatibility. Once [openspec-orbit#4](https://github.com/las-sal/openspec-orbit/issues/4) (address-reviews accepts JSON natively) ships, this conversion step won't be needed and `/opsx:address-reviews <change>` (with #10 auto-discovery) becomes the canonical close-the-loop command.

Self-honest framing on review depth: 5 CRITICAL is a lot for one review pass on work I just authored. The findings are evidence-based (each verified via grep/file-read), but a fresh-context external review may surface additional anchoring misses. Worth considering `/opsx:review-external --as proposal` after this address-reviews walk concludes — or before, to surface anything this pass missed.
