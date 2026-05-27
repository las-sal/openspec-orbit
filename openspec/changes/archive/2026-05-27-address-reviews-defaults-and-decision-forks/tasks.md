<!--
Implementation chunks:
  Chunk 1 (groups 1-2):  Cascade behavior + walk-mode default (#14 + #11)
  Chunk 2 (groups 3-4):  Decision-fork producer-side affordance + consumer-side detection (#18)
  Chunk 3 (groups 5-6):  Resolution-log JSON shape + reference doc updates
  Chunk 4 (groups 7):    Documentation surface
  Chunk 5 (groups 8):    User-validation handoff
-->

## 1. Cascade-by-default behavior (#14)

- [x] 1.1 Update `.claude/skills/openspec-address-reviews/SKILL.md` Step 3e (Ripple cascade) — change from "list, don't edit" to "edit IN-set files; record OUT-set in `flagged_not_applied`"; document the four lifecycle-invariant OUT categories per D7 (no file-extension discrimination)
- [x] 1.2 Mirror Step 3e changes in `.claude/commands/opsx/address-reviews.md`
- [x] 1.3 Add `--no-cascade` flag handling to the SKILL.md flag table + flag-resolution logic; mirror in command file
- [x] 1.4 Document the `flagged_not_applied[]` entry reason codes in SKILL.md and command file — these come from TWO distinct sources: (a) structural OUT-category reasons (`audit-trail file; cascade skipped by policy`, `baseline spec; add a delta to your current change's specs/<capability>/spec.md to capture this ripple`, `cross-change ripple; cascade scope is current change only`, `safe-exclusion path; never edited`) which apply when cascade was on but a file matched an OUT prefix; (b) mode-suppression reason (`--no-cascade suppressed`) which applies uniformly to ALL ripple-flagged files when `--no-cascade` is set, regardless of IN/OUT classification. The two source categories are structurally distinct per the spec's `Ripple cascade by default` mode invariants.
- [x] 1.5 Update the SKILL.md frontmatter `description` line if cascade-default reframing requires it

## 2. Walk-mode-by-default behavior (#11)

- [x] 2.1 Update `.claude/skills/openspec-address-reviews/SKILL.md` Step 3 lifecycle prose — change from "all findings together" to "per-finding default with `--batch` opt-in"
- [x] 2.2 Add `--batch` flag handling to SKILL.md flag table + flag-resolution logic; mirror in command file
- [x] 2.3 Document the verbal-trigger detection rule (invocation-message-only; explicit phrases listed) in SKILL.md
- [x] 2.4 Document the mid-walk command-shape interruption recognition rule + bare-message criterion + 1-indexed `walk_mode_shifted_at_finding` capture
- [x] 2.5 Update Step 3 worked example in SKILL.md (show a 3-finding walk-mode trace)
- [x] 2.6 Mirror all Step 3 changes in `.claude/commands/opsx/address-reviews.md`

## 3. Decision-fork: producer-side affordance (orbit-review + orbit-audit-drift)

- [x] 3.1 Update `.claude/skills/openspec-review/SKILL.md` — document the optional `recommendation_options: [{label, body}]` field on disjunctive findings; when to emit; prose-recommendation-still-summarizes rule; require ≥ 2 entries (producer-side enforcement per S4)
- [x] 3.2 Mirror in `.claude/commands/opsx/review.md`
- [x] 3.3 Update `.claude/skills/openspec-review/references/run-summary-schema.md` to add the optional `recommendation_options[]` field to the finding shape (file documents finding shape at line ~59)
- [x] 3.4 Update `.claude/skills/openspec-audit-drift/SKILL.md` — same optional field; per-category emit guidance (C1 typically single-rec; C2 + C3 often disjunctive; C4 typically single); require ≥ 2 entries
- [x] 3.5 Mirror in `.claude/commands/opsx/audit-drift.md`
- [x] 3.6 Update `.claude/skills/openspec-audit-drift/references/run-summary-schema.md` to add the optional `recommendation_options[]` field (file documents finding shape at line ~61)

## 4. Decision-fork detection: consumer side (#18 in address-reviews)

- [x] 4.1 Add new section `Step 3c.5 — Decision-fork detection` to `.claude/skills/openspec-address-reviews/SKILL.md`: positioned after classify, before fix, gated on `classify == "decision required"`
- [x] 4.2 Document the hybrid detection rule: try structured (`recommendation_options[]` in JSON-virtual marker), fall back to heuristic over `**Description**:` / `recommendation` text
- [x] 4.3 Document the conservative heuristic triggers (numbered alternatives, "either…or" with clause-level branches, `**Options:**` prefix) and explicit non-triggers (loose "or" in prose)
- [x] 4.4 Document the `[discuss]` escape hatch as a mandatory option on every fork prompt; describe the tradeoff-analysis-then-re-prompt UX
- [x] 4.5 Document malformed-array fallback (zero entries, single entry, missing label/body) → heuristic fallback + stderr warning + `structured_path_skipped_reason` field in resolution log
- [x] 4.6 Update Step 3 worked example in SKILL.md to include a fork-prompt walkthrough
- [x] 4.7 Mirror all Step 3c.5 changes in `.claude/commands/opsx/address-reviews.md`

## 5. Resolution-log JSON shape updates

- [x] 5.1 Update `.claude/skills/openspec-address-reviews/references/run-summary-schema.md` — replace `ripple_flagged_files_aggregate` (v1) with `ripple_cascade.applied / flagged_not_applied` (v2); add top-level `walk_mode`, `walk_mode_source`, `walk_mode_shifted_at_finding`; add per-resolution `recommendation_fork` object
- [x] 5.2 Document the v1 → v2 reader migration rule in the same reference doc (readers MUST treat presence of `ripple_cascade` as v2; absence + `ripple_flagged_files_aggregate` as v1)
- [x] 5.3 Add worked example JSON snippets showing: (a) walk-mode clean run; (b) batch-mode with verbal source; (c) decision-fork with `[discuss]` invoked; (d) `--no-cascade` with all ripples flagged-not-applied
- [x] 5.4 Update `.claude/skills/openspec-address-reviews/references/external-findings-format.md` (if it touches resolution-log shape) — likely no change since external findings format is input-side, but verify

## 6. Internal findings format reference

- [x] 6.1 Update `.claude/skills/openspec-address-reviews/references/internal-findings-format.md` — document the optional `recommendation_options[]` field as part of the review-JSON and audit-drift-JSON shape sections; describe how the parser reads it for the structured-detection path
- [x] 6.2 Cross-reference the parser contract to the consumer-side spec scenario in `orbit-address-reviews`'s `Disjunctive recommendation fields surface as decision forks` requirement
- [x] 6.3 Update the "Strictness" section if it currently asserts the absence of optional fields

## 7. Documentation surface

- [x] 7.1 Update `README.md` `/opsx:address-reviews` section — document new defaults (walk-mode, cascade); document `--batch` and `--no-cascade` flags; document decision-fork prompts
- [x] 7.2 Update README workflow box / ASCII diagram if it shows the address-reviews flow
- [x] 7.3 Update `.claude/skills/openspec-onboard/SKILL.md` command table row for `/opsx:address-reviews` to reflect new defaults (1-2 lines max)
- [x] 7.4 Mirror in `.claude/commands/opsx/onboard.md`

## 8. User-validation (intentionally unchecked at apply-time)

- [ ] 8.1 (User-validation handoff) — Live-test walk-mode default on a real change with multiple findings; verify each finding gets its own checkpoint; verify cascade applies to IN-set files (mix of markdown and non-markdown if available)
- [ ] 8.2 (User-validation handoff) — Live-test `--batch` flag; verify batch-mode runs through all findings without per-finding prompts; verify cascade applies as a single end-step
- [ ] 8.3 (User-validation handoff) — Live-test verbal `--batch` in invocation message (e.g., `/opsx:address-reviews foo, fix them all`); verify mode-shift recognized with `walk_mode_source: "verbal"`
- [ ] 8.4 (User-validation handoff) — Live-test mid-walk command-shape interruption (send bare "go batch" message between findings); verify `walk_mode_source: "command-shape-interruption"` + `walk_mode_shifted_at_finding` populated
- [ ] 8.5 (User-validation handoff) — Live-test decision-fork detection on a finding with numbered-alternative recommendation; verify fork prompt surfaces; verify `[discuss]` escape works
- [ ] 8.6 (User-validation handoff) — Live-test structured `recommendation_options[]` path: emit a review JSON with the field populated, run address-reviews via auto-discovery; verify structured-path detection fires
- [ ] 8.7 (User-validation handoff) — Live-test `--no-cascade` flag; verify all ripples land in `flagged_not_applied[]` with `--no-cascade suppressed` reason
- [ ] 8.8 (User-validation handoff) — Verify resolution-log JSON contains all new fields per schema; pipe through `jq` for sanity check
