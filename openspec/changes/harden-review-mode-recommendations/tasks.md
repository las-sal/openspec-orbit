<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):   Update openspec-review SKILL body + matching command file
                       (stock-phrasings table + iteration-aware logic prose)
  Chunk 2 (group 2):   Update README — add "Choosing a review mode" section
  Chunk 3 (group 3):   Validation + user-validation handoff

Total: 3 implementation chunks. No sandbox verification needed: this is
prose + spec text only (no CLI behavior change, no overlay surface change).
Sync-specs at archive applies the MODIFIED + ADDED requirements to baseline.
-->

## 1. Update openspec-review SKILL body + matching command file

- [ ] 1.1 Update `.claude/skills/openspec-review/SKILL.md`'s `### 9. Emit the final assessment` section. Replace the system-mode rows in the stock-phrasings table with the 10 new rows from the MODIFIED `Final assessment phrasings depend on mode` requirement — 5 system-mode "no CRITICAL" convergence states (no prior external / Path A clean / Path B resolved / unresolved findings / stale post-apply), each with "all clear" and "Only WARNING/SUGGESTION" sub-cases = 10 total stock-phrasing rows. Add the iteration-aware logic prose: how to (a) check `external-system-*.md` presence, (b) parse the external markdown for Path A clean detection (count `### <title>` entries under each `## SEVERITY` heading), (c) inspect `address-reviews-*.json` whose `source_path` references the external for Path B resolution detection, (d) compare filename `<TS>` tokens against the most recent `apply-*.json` for stale detection, and (e) apply the convergence-state precedence rules per the spec's `Convergence-state precedence when multiple states apply` scenario.

- [ ] 1.2 Update the worked example in `.claude/skills/openspec-review/SKILL.md` (the partial system-mode review example) to demonstrate the new final-assessment phrasing. If a worked example doesn't yet exist for system mode in the current SKILL, add a short illustrative one showing the "All checks passed. Recommend /opsx:review-external..." case.

- [ ] 1.3 Update `.claude/commands/opsx/review.md` to mirror SKILL changes from 1.1 + 1.2. Per orbit-modified command-file convention, the body sections mirror the SKILL (excluding frontmatter); the iteration-aware logic + new stock phrasings appear in both.

- [ ] 1.4 Verify both files cite the empirical evidence consistently: `bootstrap-orbit-status-cli` 3-of-3 finding should appear in the same form (e.g., "in-context system review missed 3 of 3 real bugs in bootstrap-orbit-status-cli's first archived cycle") in both SKILL.md + opsx/review.md. Grep both files for "3 of 3" — should appear at least once each.

- [ ] 1.5 Re-grep both files for stale phrasings to confirm removal: `grep -n "All checks passed. Ready to archive." .claude/skills/openspec-review/SKILL.md .claude/commands/opsx/review.md` should NOT match the system-mode "all clear" row (the new phrasing is `All checks passed. Recommend /opsx:review-external...` or `All checks passed. External system review converged clean. Ready to archive.`). Proposal-mode "Ready to apply." phrasings are unchanged and should still appear.

## 2. Update README — add "Choosing a review mode" section

- [ ] 2.1 Add a new top-level section to `README.md` titled `## Choosing a review mode`. Place it immediately after `## Command reference` (currently L238-481) and immediately before `## The external review cycle` (currently L482) — adjacent to the cycle docs the new section cross-references. Update README's TOC (L9-40) to include the new section in the correct position. Verify line numbers at edit time (README may have shifted); the section anchors are stable even if line numbers move.

- [ ] 2.2 Section content (target ~50 lines, lighter guidance than the spec scenarios):
  - Three-mode framework summary (in-context / `--fresh` / external) with 1-paragraph each describing what it is + when to use
  - Recommended cycle patterns by change size (small / medium / substantial / high-stakes) — as guidance, NOT as normative scenarios
  - Cite the bootstrap-orbit-status-cli 3-of-3 empirical evidence as the basis for the system-mode external default
  - Cross-reference the orbit-conventions baseline `Review mode decision framework` requirement for normative criteria
  - Cross-reference the orbit-review `Final assessment phrasings depend on mode` requirement for runtime recommendation behavior

- [ ] 2.3 Verify the README section cross-references resolve: `grep -n "Review mode decision framework\|Final assessment phrasings depend on mode" README.md` — both should appear in the new section.

- [ ] 2.4 Re-run `/opsx:audit-drift` Category 3 (cross-doc consistency) check mentally (or invoke if convenient): does README's framework match the spec scenarios? Any divergence would be caught by audit-drift on next run; manual sanity-check here catches it pre-archive.

## 3. Validation + user-validation handoff

- [ ] 3.1 Run `openspec validate harden-review-mode-recommendations --strict`; resolve any validation findings.

- [ ] 3.2 (User-validation) User reads the rewritten final-assessment table + iteration-aware logic in `.claude/skills/openspec-review/SKILL.md` and confirms:
  - (a) New system-mode phrasings are clear about when to expect each variant (no prior external / converged clean / stale)
  - (b) Iteration-aware logic prose is implementable (the file-existence check + timestamp comparison are unambiguous)
  - (c) Empirical citation reads naturally (not gratuitous or distracting)
  - (d) Recommendation prose mentions both `/opsx:review-external` and `/opsx:review --fresh` per design D-mention-fresh

- [ ] 3.3 (User-validation) User reads the new README "Choosing a review mode" section and confirms:
  - (a) Three-mode framework is parseable for a cold reader
  - (b) Recommended cycle patterns by change size are useful, not over-prescriptive
  - (c) Cross-references to orbit-conventions + orbit-review requirements work as expected
  - (d) Section doesn't duplicate too much from existing docs (the framework references existing prose where possible)

- [ ] 3.4 (User-validation) User reads the new `Review mode decision framework` scenarios in the orbit-conventions delta and confirms each scenario's WHEN/THEN is testable (i.e., a future drift-audit run could surface drift between docs and spec).

- [ ] 3.5 If user-validation surfaces no blocking findings, the change is ready for `/opsx:review` (proposal-mode pre-apply review per the canonical flow). Then `/opsx:apply` → `/opsx:review --as system` (which under the new logic will recommend external next, dogfooding this change's own behavior change) → `/opsx:archive`.
