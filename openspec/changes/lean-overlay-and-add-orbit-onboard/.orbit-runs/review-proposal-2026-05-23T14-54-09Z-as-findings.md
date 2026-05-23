# Findings — lean-overlay-and-add-orbit-onboard (proposal-mode --thorough review iter-3, in-context Claude)

Bridge file converting `review-proposal-2026-05-23T14-54-09Z.json` to markdown for `/opsx:address-reviews` ingest. Persists openspec-orbit#4.

**Source**: in-context Claude --thorough constraint-probe pass on 2026-05-23T14:54:09Z (proposal mode, iteration 3).
**Counts**: 0 CRITICAL, 2 WARNING, 2 SUGGESTION, 0 stale.

---

## WARNING

### TR1 — Task 3.4 mandates "fresh-sandbox install" but never defines what "sandbox" is

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/tasks.md:34`
**Dimension**: correctness

After FF4 stripped the escape hatch, task 3.4 reads "Mandatory: run a fresh-sandbox install per the documented README sequence". But "fresh sandbox" is undefined — temp dir? Docker? fresh git clone? specific Node version? Implementing AI may default to escalation (defeating FF4 intent) or interpret loosely (re-running in dev tree — not actually fresh).

**Recommendation**: Define "fresh sandbox" inline. Minimum: temp directory + same Node version as project's package.json + run openspec init from clean state. One parenthetical sentence in task 3.4.

### TR2 — orbit's archive dependency on openspec-sync-specs primitive not anchored as normative requirement

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-conventions/spec.md` (Overlay file disposition, Upstream-required primitive scenario)
**Dimension**: correctness

Overlay file disposition says "kept verbatim because orbit depends on it as a callable primitive" but doesn't anchor WHICH orbit code calls it. Implementation evidence (openspec-archive-change/SKILL.md:65,112,123) confirms archive flow uses it, but only as prose, not normative spec. If Option 2 (#27) work later rewrites archive flow without re-reading those invocation specs, sync-specs could be quietly dropped.

**Recommendation**: Either (a) add normative requirement capturing the dependency, or (b) cross-reference openspec-archive-change SKILL.md invocation pattern from the Upstream-required primitive scenario for traceability.

---

## SUGGESTION

### TR3 — "post-pegging forward-looking framing" prescription is subjective

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md` (Identity statement section requirement)
**Dimension**: correctness

Requirement says Identity SHALL convey "post-pegging forward-looking framing". Scenario gives one example but constraint is loose — two implementing AIs could produce semantically-different content that both technically satisfy.

**Recommendation**: Add negative test scenario: "framing MUST NOT use augmentation language (e.g., 'augments cleanly', 'overlay on top of')". Negative test more useful than positive — goal is preventing old framing, not enforcing specific new vocabulary.

### TR4 — Try-it nudge assumes user has a real project idea

**File**: `openspec/changes/lean-overlay-and-add-orbit-onboard/specs/orbit-onboard/spec.md` (Try-it nudge section requirement)
**Dimension**: coherence

Nudge prescribes /opsx:explore <name> 'on a real project idea they have'. Orientation-only users (collaborators new to orbit, AI sessions getting context cold) have no concrete idea yet.

**Recommendation**: Relax to: 'recommend /opsx:explore <name> when reader has a real idea, OR /opsx:explore (bare mode) to think out loud if no idea ready yet'. Preserves no-demo-change discipline; surfaces bare-mode as the no-decision-yet entry point.
