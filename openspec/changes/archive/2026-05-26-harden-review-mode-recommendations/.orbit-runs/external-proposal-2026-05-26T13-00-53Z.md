# External Review: harden-review-mode-recommendations (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-26

## CRITICAL

### Iteration-aware logic can mark unresolved external findings as converged
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:34
**Description**: The "prior external converged clean" scenario treats convergence as `external-system-*.md` existing plus a later clean `review-system-*.json`, but it never inspects whether the external findings file was clean or whether its findings were resolved by `address-reviews`. The new edge-case scenario also says an `external-system-*.md` counts regardless of content, so a report with CRITICAL/WARNING findings, deferred findings, or escalations can still become "External system review converged clean. Ready to archive." after a later in-context review reports no CRITICALs. That undercuts the core purpose of the change: the recommendation can suppress the external-review nudge without proving the external review actually converged. Tighten the requirement so "converged clean" means either the latest `external-system-*.md` contains no findings, or a later `address-reviews-*.json` whose `source_path` is that external file shows all findings resolved or stale-suppressed with no deferred/escalated unresolved items, followed by the current clean system review. Otherwise keep recommending external or address-reviews before archive.

## WARNING

### Stale-external branch compares the wrong timestamps
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:39
**Description**: The stale case is defined as the latest internal `review-system-*.json` being earlier than the latest `external-system-*.md`, but the prose says the problem is "external ran, then artifacts changed without re-running external, then internal review now reports clean." In that real stale sequence, the current clean internal review would be later than the external file, which line 34 classifies as converged clean. The requirement also says the prior external is "older than the most recent artifact changes" but the WHEN clause never checks artifact-change, apply, or address-reviews timestamps. Define stale against the latest material artifact-changing event after the external review, not against the previous internal review file. At minimum, specify whether the command should compare the external-system filename token to the current run timestamp, latest `apply-*.json`, latest `address-reviews-*.json` for that external file, or git artifact modification evidence.

### Stale warning/suggestion phrasing is not an exact stock phrase
**File**: openspec/changes/harden-review-mode-recommendations/specs/orbit-review/spec.md:40
**Description**: Every other new system-mode sub-case gives exact stock phrasing, but the stale "Only WARNING/SUGGESTION" sub-case only says it "parallels with the warning-count prefix." That leaves implementers to invent the exact final-assessment line, and it conflicts with tasks.md's instruction to replace the SKILL table rows from the spec. The task text also says "4 new rows" while the three no-CRITICAL states split into all-clear and warning/suggestion sub-cases require six rows. Add the exact stale warning/suggestion phrase to the spec and update tasks.md to state the actual row count so `.claude/skills/openspec-review/SKILL.md` and `.claude/commands/opsx/review.md` stay byte-level consistent.

## SUGGESTION

### Scenario counts still describe only the decision-criteria subset
**File**: openspec/changes/harden-review-mode-recommendations/proposal.md:20
**Description**: The proposal says the new `Review mode decision framework` requirement has "3 scenarios", while the actual orbit-conventions delta now has five scenarios: three decision criteria plus README/spec drift and non-enforcement scenarios. design.md uses the same "3 scenarios" shorthand. This is understandable as a distinction between core criteria and ancillary governance, but it reads like stale count drift in the primary proposal. Update proposal.md and design.md to say "three decision-criteria scenarios plus two governance/non-enforcement scenarios" or simply "5 scenarios" to match the spec delta.
