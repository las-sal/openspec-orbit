# Findings — lean-overlay-and-add-orbit-onboard (proposal-mode review iter-2, fresh-context Claude)

Bridge file converting `review-proposal-2026-05-23T14-38-05Z.json` to markdown format for `/opsx:address-reviews` ingest. Persists openspec-orbit#4 (JSON-to-markdown bridge workaround).

**Source**: fresh-context Claude subagent review on 2026-05-23T14:38:05Z (proposal mode, iteration 2).
**Counts**: 1 CRITICAL, 3 WARNING, 3 SUGGESTION, 0 stale.

---

## CRITICAL

### FF1 — Stale `#6 will deprecate /opsx:sync-specs` survives in archive-summary-schema.md

**File**: `.claude/skills/openspec-archive-change/references/archive-summary-schema.md:18`
**Dimension**: coherence

Codex EW1 implicitly named THREE locations carrying stale #6-removal framing. iter-2 swept `orbit-conventions` and `orbit-run-summary-emit` but missed this third location. Once this change archives, baseline orbit-conventions says sync_specs is permanent-under-pegging while archive-skill's own schema reference still predicts its removal.

**Recommendation**: Add task to update `archive-summary-schema.md:18` with post-pegging framing. Explicitly grep `.claude/` for `openspec-orbit#6` / `slated for removal` to confirm no further residue.

---

## WARNING

### FF2 — MODIFIED requirement title contradicts post-pegging body

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-conventions/spec.md:3`
**Dimension**: coherence

MODIFY-by-name match rule requires keeping baseline title verbatim, so `### Requirement: Distribution model — overlay, not CLI fork` is structurally correct. But the body now describes pegging. Design D-conventions-1 item 1 mentioned a rename ("Rename to `Distribution model — pegged engine + orbit-owned surface`") but the delta spec didn't perform it.

**Recommendation**: Either (a) add RENAMED Requirements section to do the rename; or (b) explicitly note in design.md that rename was rejected and update cross-references. Pick one.

### FF3 — Setup verification distinction unclear

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md:45-48`
**Dimension**: correctness

Scenario `Verify pruned upstream files are absent` says "a correct install has these absent after the user runs the documented post-overlay-copy prune step" — conditional. Adjacent `Verify overlay applied via stable post-install artifacts` asserts unconditional presence. Implementation must distinguish "fail because user didn't rm" (warning) from "fail because overlay is incomplete" (error) — spec gives no guidance.

**Recommendation**: One-line addition to `Verification failures are informational` scenario clarifying the distinction.

### FF4 — Task 3.4 `if possible / best-known` leaves verification ambiguous

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:33`
**Dimension**: completeness

Task 3.4 escape hatch: "Verify counts against a fresh sandbox install if possible; otherwise mark as best-known and note the verification gap." Combined with task 5.4 user-validation, no firm forcing function: 3.4 may ship with best-known counts; 5.4 may validate without true sandbox check.

**Recommendation**: Either make 3.4 unconditional sandbox-install OR add an explicit task that performs the sandbox install as the verification gate.

---

## SUGGESTION

### FF5 — Task 3.6 doesn't specify README anchor points for `rm` commands

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:35`
**Dimension**: coherence

Chunk 2 (narrative sweep) runs before chunk 3 (README rm-step docs). Author may delete install-section `feedback` references without knowing chunk 3 needs anchors there. Task 3.6 doesn't reference anchor points.

**Recommendation**: Task 3.6 should specify WHERE rm commands land in README. Also note in task 2.3 that install-section feedback references will get explicit rm lines in chunk 3.

### FF6 — orbit-onboard quick-reference table should explicitly exclude pruned `sync.md`

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md:102-105`
**Dimension**: coherence

Scenario says "every command currently shipped in .claude/commands/opsx/ appears in table". After chunk 1.2 prunes sync.md, "currently shipped" excludes it — but implementer might wonder if /opsx:sync should appear with deprecation note.

**Recommendation**: Add one sentence to scenario explicitly excluding pruned commands.

### FF7 — User-validation tasks 5.3-5.5 don't reflect iter-2 expanded scope

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:56-58`
**Dimension**: correctness

iter-2 added sync.md pruning, EC1 rm steps, EW3 command-disposition, EW2 stable-artifact verification rewrite — user-validation tasks didn't grow.

**Recommendation**: Add sub-items 5.3a/5.4a/5.5a covering the iter-2 expansions.
