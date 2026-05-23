# Findings — lean-overlay-and-add-orbit-onboard (proposal-mode review iter-1)

Bridge file converting `review-proposal-2026-05-23T13-42-44Z.json` to markdown format for `/opsx:address-reviews` ingest. Persists openspec-orbit#4 (JSON-to-markdown bridge workaround until address-reviews accepts JSON directly).

**Source**: in-context review on 2026-05-23T13:42:44Z (proposal mode, iteration 1).
**Counts**: 3 CRITICAL, 4 WARNING, 3 SUGGESTION, 1 stale suppressed.

---

## CRITICAL

### F1 — /opsx:onboard non-emission scope-enforcement may be lost in rewrite

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md` (chunk 4.1)
**Dimension**: completeness

Current `.claude/skills/openspec-onboard/SKILL.md` has `# Orbit additions` section asserting `/opsx:onboard` does NOT emit (per orbit-run-summary-emit Emit scope). Task 4.1 deletes existing content; new 100% orbit-authored body has no provision for re-stating this.

**Recommendation**: add task to chunk 4 to preserve non-emission note in new SKILL.md OR document in design.md that non-emission is enforced solely by orbit-run-summary-emit spec.

### F2 — README pre-pegging framings not swept by current tasks

**File**: `README.md:915,931,943,969`
**Dimension**: completeness

README contains "upstream skills + feedback" / "11 upstream + 4 orbit" / similar pre-pegging accounting. Tasks 2.3-2.5 handle install-order/table-counts/sync-specs from #6 body but don't sweep these other framings.

**Recommendation**: add task to chunk 2 (or expand 1.3) — sweep README for all references to "upstream skills" / pre-pegging accounting, rewrite under post-pegging framing per orbit-conventions Overlay file disposition.

### F3 — Cross-chunk coupling (chunks 1-2 → chunk 3) not declared in tasks.md

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md` (chunking comment + chunks 1-3)
**Dimension**: coherence

Chunks 1-2 update README narrative containing references to "feedback" skill that chunk 3 deletes. Current chunking implies strict 1→2→3 DAG; actual dependency is 3-before-references-to-3.

**Recommendation**: either (a) reorder chunks to 3→1→2, (b) explicitly note in chunk 1 preamble that README updates must account for chunk 3 deletion, or (c) add cross-chunk verification task at chunk 2 end.

---

## WARNING

### F4 — Task 2.4 too narrow — table-counts is symptom of bigger framing issue

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:2.4`
**Dimension**: correctness

Table-counts issue is symptomatic of pre-pegging "upstream-vs-orbit" framing (F2). Treating as counts-only misses structural reframe.

**Recommendation**: rewrite 2.4 to focus on framing reframe (orbit overlay surface, per orbit-conventions Overlay file disposition); count correction is consequent.

### F5 — Task 4.4 (canonical-flow walkthrough) too concentrated

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:4.4`
**Dimension**: coherence

9 phases + lens intro (2 touches) + external-review demo packed into one task. High writing volume risks skipped phases or inconsistent paragraph depth.

**Recommendation**: split into 4.4a (phases 1-4), 4.4b (phases 5-9), 4.4c (lens intro + external-review demo woven). OR add explicit per-phase checklist within the single task.

### F6 — Upstream gap (non-interactive expanded-profile) not tracked

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:2.3`
**Dimension**: correctness

Task references the upstream-gap but no follow-up issue exists on Fission-AI/OpenSpec yet. #6 body said "happy to file separately" — no such issue.

**Recommendation**: add task to chunk 5 — file issue on Fission-AI/OpenSpec for non-interactive expanded-profile gap; reference in README as known limitation with upstream issue link.

### F7 — Spec couples quick-reference table to Overlay file disposition (maintenance burden)

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md` (Quick-reference command table section requirement)
**Dimension**: correctness

Spec requires table show origin (orbit-authored / orbit-modified / upstream-primitive). Couples SKILL.md to orbit-conventions Overlay file disposition. Every reclassification (e.g., Option 2 work) requires lockstep table update.

**Recommendation**: consider whether origin column is necessary OR could be a link to orbit-conventions (single source of truth) rather than duplicated content.

---

## SUGGESTION

### F9 — Cross-reference verification passed

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md` (Setup verification section requirement)
**Dimension**: correctness

orbit-onboard spec references orbit-conventions Upstream version pinning + Distribution model. Names match exactly. Logged for completeness.

**Recommendation**: no action; flagged so future-edits know to re-verify if requirement names change.

### F10 — Proposal Impact list omits README troubleshooting/install-verification scope

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/proposal.md` (Impact section)
**Dimension**: completeness

Impact says "Documentation: CLAUDE.md, README, and orbit narrative docs require coordinated rewriting" but doesn't enumerate specific README sections (troubleshooting around L969, install verification table) needing updates per the reframe.

**Recommendation**: add sub-bullet enumerating: README install instructions, install verification table, troubleshooting section, any prose referencing pre-pegging upstream/orbit accounting.

### F11 — No explicit openspec validate --strict task before user-validation

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md` (chunk 5)
**Dimension**: coherence

Chunk 5 has issue closures (5.1, 5.2) + user-validation (5.3-5.5). No explicit openspec validate --strict step. /opsx:verify runs this naturally but explicit task helps document the gate.

**Recommendation**: optional — add 5.0 "Run openspec validate lean-overlay-and-add-orbit-onboard --strict; resolve any validation findings before user-validation handoff."

---

## Stale (suppressed)

### F8 — feedback skill refs in onboard files (suppressed)

**File**: `.claude/commands/opsx/onboard.md:237, .claude/skills/openspec-onboard/SKILL.md:241`

Original concern: onboard.md and SKILL.md reference "feedback" suggesting cleanup needed when feedback skill deleted.
**INVESTIGATION**: both references are the prose word "approval/feedback", NOT references to the feedback SKILL. No cleanup needed.
**Status**: suppressed. Evidence: `grep -n 'feedback' <file>` shows only the "approval/feedback" prose context.
