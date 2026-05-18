> **Convention**: tasks should reference the spec requirement they implement using the form `[R<#> in <capability>]` at the end of the task description, e.g., `[R3 in orbit-review]`. Tasks without spec references are **operational** (build/release/docs/testing) — they implement orbit's delivery rather than orbit's behavior, and are explicitly distinguished from spec-tied work in the group's introduction line. Annotations are added incrementally as implementation lands; tasks below are seeded without them.

## 1. Repository scaffolding (operational)

- [x] 1.1 Confirm `LICENSE` (MIT), `README.md` (full workflow + command reference), `.gitignore` (excludes personal-transcript files and `settings.local.json`) are in place
- [x] 1.2 Verify `openspec/config.yaml` reflects orbit-coherent settings (schema, optional project context, any orbit-specific rules)
- [x] 1.3 Add a top-level `CLAUDE.md` snippet (or document in README) that adopters can copy as project-level context, including all three execution-discipline reminders: read-before-reference (authoring-time), change completeness (modification-time), pushback (review-time) — per the snippet shipped in the README's "Cross-cutting disciplines" section

## 2. New skill: `/opsx:review` (unified, mode-aware)

- [x] 2.1 Author `.claude/skills/openspec-review/SKILL.md` describing both modes (proposal: 9 passes; system: verify-change Pass 0 + 6 system-wide passes), `--as` flag with mode inference, shared scorecard rollup, flag family, graceful degradation per `specs/orbit-review/spec.md`. **Embed all three execution disciplines** as self-contained reminder sections in the prompt: read-before-reference (when authoring findings that reference specific code/specs), change completeness (if `--mark` modifies artifacts), pushback (verify each finding against current state before reporting).
- [x] 2.2 Author `.claude/commands/opsx/review.md` slash command body that surfaces both modes with concise arg/flag descriptions
- [x] 2.3 Implement `--as proposal|system` flag with state-based mode inference from `tasks.md` (unchecked → proposal; all checked + code → system; ambiguous → `AskUserQuestion`)
- [x] 2.4 Implement proposal-mode passes 1–9 (Structure & Delta, Internal Coherence, Cross-Doc, Archive Consistency, Codegen Readiness, Gap Hunt, Drift Hunt, Inline Review Marker Residue, Pre-Handoff Sweep)
- [x] 2.5 Implement system-mode Pass 0: delegate to upstream `/opsx:verify-change` via the `openspec-verify-change` skill
- [x] 2.6 Implement system-mode passes 1–6 (Baseline Compliance, Cohesion, Surface Walk, Perspective Reviews, Critical-Path Scan, Drift/Residue)
- [x] 2.7 Implement system-mode Pass 6 invocation of `/opsx:audit-drift` as a library function
- [x] 2.8 Implement `.orbit-runs/review-<mode>-<TS>.json` summary writer (per-mode iteration counter, fields per `orbit-conventions`)
- [x] 2.9 Implement iteration-note generation when prior `review-<mode>-<TS>.json` files exist for the same mode
- [x] 2.10 Implement `--mark` writer (proposal mode only) that converts each finding into a `@review:` marker at the finding's file:line; produce a clear note when invoked in system mode (v2 territory per issue #3)
- [x] 2.11 Implement `--skip-verify` handling (system mode only) that bypasses Pass 0 with an explanatory note; silently accepted in proposal mode (no effect)
- [x] 2.12 Implement `--parallel` subagent spawning — proposal mode parallelizes Passes 2, 4, 6; system mode parallelizes Passes 2, 3, 4, 5
- [ ] 2.13 Test happy path against the orbit dev sandbox in proposal mode (change `bootstrap-openspec-orbit`) and system mode (a small applied change)  <!-- deferred: user-driven integration test per chunking plan -->


> **Note**: Group 3 was intentionally merged into Group 2 when `/opsx:review-proposal` and `/opsx:review-system` were unified into the single `/opsx:review` command with `--as proposal|system` modes. The numbering gap (2 → 4) is preserved rather than renumbering downstream groups so that any external references to Group N (e.g., in commit messages, prior review findings) remain valid.

## 4. New skill: `/opsx:review-external`

- [x] 4.1 Author `.claude/skills/openspec-review-external/SKILL.md` describing handoff prompt generation
- [x] 4.2 Author `.claude/commands/opsx/review-external.md` slash command body
- [x] 4.3 Implement `--as proposal|system` flag with state-based inference from `tasks.md`
- [x] 4.4 Implement iteration counting per mode (count of matching `external-<as>-*.md` files in `.orbit-runs/`)
- [x] 4.5 Implement cycle-context population from prior `.orbit-runs/` files (open findings, resolved since last)
- [x] 4.6 Author the proposal-mode prompt skeleton (lists the 9 proposal-mode passes)
- [x] 4.7 Author the system-mode prompt skeleton (lists the 7 system-mode passes)
- [x] 4.8 Include uncommitted-changes warning in chat output
- [ ] 4.9 Test prompt generation for both modes; manually validate that codex can follow it  <!-- deferred: user-driven integration test per chunking plan -->

## 5. New skill: `/opsx:audit-drift`

- [x] 5.1 Author `.claude/skills/openspec-audit-drift/SKILL.md` describing the 4 scan categories, scorecard rollup, flag family, three invocation paths (standalone, library, pre-archive). **Embed all three execution disciplines** as self-contained reminder sections in the prompt: read-before-reference (when citing specific paths or symbols in findings), change completeness (not applicable for audit-only, but document for consistency), pushback (verify each potential finding against current state before reporting).
- [x] 5.2 Author `.claude/commands/opsx/audit-drift.md` slash command body
- [x] 5.3 Implement Category 1 (Vocabulary residue) — extract residue patterns from archived deltas; grep target docs
- [x] 5.4 Implement Category 2 (Lens staleness) — resolve perspective surface refs against `openspec/specs/`; check critical-path touchpoints
- [x] 5.5 Implement Category 3 (Cross-doc consistency) — extract structured claims; pairwise compare
- [x] 5.6 Implement Category 4 (Archive coherence) — walk archived changes within window; verify ADDED/REMOVED propagation to baseline
- [x] 5.7 Implement `--focus <area>` and `--since <ref>` flags
- [x] 5.8 Implement context-aware final-assessment phrasings (standalone / pre-archive / library)
- [x] 5.9 Implement summary writer for `.orbit-runs/audit-drift-<TS>.json` (change-scoped) and a project-level fallback location for standalone runs
- [ ] 5.10 Test happy path on this very repo (vocabulary residue, lens staleness, doc consistency, archive coherence)  <!-- deferred: user-driven integration test per chunking plan -->

## 6. New skill: `/opsx:address-reviews` (lean v1)

- [x] 6.1 Author `.claude/skills/openspec-address-reviews/SKILL.md` describing the lean v1 lifecycle (discover → triage → walk → ripple flag → report). **Embed all three execution disciplines** as self-contained reminder sections in the prompt: read-before-reference (when applying fixes that reference existing code/specs), change completeness (when a marker's resolution touches related artifacts), pushback (verify each marker against current state before fixing — the primary discipline for this command).
- [x] 6.2 Author `.claude/commands/opsx/address-reviews.md` slash command body
- [x] 6.3 Implement default whole-repo `@review:` marker discovery with safe exclusions
- [x] 6.4 Implement `--only <pattern>` scoping
- [x] 6.5 Implement pushback discipline step (verify against current state before fixing)
- [x] 6.6 Implement classification flow (trivial fix / decision / stale / unresolvable) with `AskUserQuestion` integration
- [x] 6.7 Implement marker removal invariant (and `--keep-resolved-markers` debug override)
- [x] 6.8 Implement ripple flag (list affected related files without auto-cascading)
- [x] 6.9 Implement unresolvable-marker handling (default file as task in `tasks.md`; alternatives: `@todo:`, `@review(escalated):`)
- [x] 6.10 Implement `--from-file <path>` parsing of external-review markdown into virtual markers (lean v1 inclusion; required to close the cross-AI loop with `/opsx:review-external` without per-finding copy-paste)
- [x] 6.11 Implement resolution-log output (Resolved/Stale/Deferred/Escalated sections)
- [ ] 6.12 Test happy path: drop `@review:` markers in this very repo and run the command  <!-- deferred: user-driven integration test per chunking plan -->

## 7. Modified skill: `/opsx:explore`

- [x] 7.1 Update `.claude/skills/openspec-explore/SKILL.md` to add orbit additions (capture affordances, explore.md authoring, three invocation modes) while preserving upstream's "stance, not workflow" character
- [x] 7.2 Update `.claude/commands/opsx/explore.md` slash command body accordingly
- [x] 7.3 Implement Mode A behavior (bare invocation — pure think; no file)
- [x] 7.4 Implement Mode B behavior (named — create or resume `openspec/explore/<name>/explore.md`)
- [x] 7.5 Implement Mode C behavior (bare-then-crystallized — prompt for name after 2+ substantive decisions emerge)
- [x] 7.6 Implement convention-capture trigger and `<topic>_convention.md` writer
- [x] 7.7 Implement perspective-capture trigger and `openspec/lenses/perspectives.md` appender (uses orbit-conventions entry shape)
- [x] 7.8 Implement critical-path-capture trigger and `openspec/lenses/critical-paths.md` appender
- [x] 7.9 Implement decision capture (proactive; with brief chat acknowledgment) into `explore.md` Decisions section
- [x] 7.10 Implement reference capture into `explore.md` References section
- [x] 7.11 Implement decline-tracking within the conversation (don't double-offer)
- [x] 7.12 Implement group offers for related captures (e.g., three conventions in succession)

## 8. Modified skill: `/opsx:propose`

- [x] 8.1 Update `.claude/skills/openspec-propose/SKILL.md` to add orbit additions (consume mode) while preserving upstream's standalone-mode behavior
- [x] 8.2 Update `.claude/commands/opsx/propose.md` slash command body accordingly
- [x] 8.3 Implement `openspec/explore/<name>/` detection; branch to consume mode when present
- [x] 8.4 Implement section mapping (Premise → proposal motivation; Decisions → spec deltas + design + tasks; Considered & out → design alternatives; References → contextual reads)
- [x] 8.5 Implement Open-question handling flow (per-question AskUserQuestion: resolve / defer-as-`@review:` / abandon)
- [x] 8.6 Implement bulk-handle UX ("resolve all / defer all / walk each") for many Open questions
- [x] 8.7 Implement directory move (`openspec/explore/<name>/` → `openspec/changes/<name>/`) after artifact generation
- [x] 8.8 Implement conflict-detection (both staging and change dirs exist) with the three-way prompt (regenerate / continue / abort)
- [x] 8.9 Implement structural validation of `explore.md` (warn on missing sections)
- [x] 8.10 Implement naming inference (single recent staging → propose name to user)
- [ ] 8.11 Test consume mode end-to-end on a fresh exploration in the orbit dev sandbox  <!-- deferred: user-driven integration test per chunking plan -->

## 9. Modified skill: `/opsx:archive`

- [x] 9.1 Update `.claude/skills/openspec-archive-change/SKILL.md` to add orbit additions (pre-archive audit-drift hook, `--skip-audit`) while preserving upstream archive behavior
- [x] 9.2 Update `.claude/commands/opsx/archive.md` slash command body accordingly
- [x] 9.3 Implement pre-archive `/opsx:audit-drift` invocation as a library call
- [x] 9.4 Implement critical-drift three-way prompt (address now / proceed / abort) via `AskUserQuestion`
- [x] 9.5 Implement `--skip-audit` flag handling
- [x] 9.6 Implement archive-run-summary writer at `openspec/changes/archive/<name>/.orbit-runs/archive-<TS>.json` with audit and sync-specs results plus user decision
- [x] 9.7 Implement `.orbit-runs/` directory move alongside the change content
- [x] 9.8 Implement unresolved-`@review:`-marker warning before completing archive
- [x] 9.9 Implement edge cases: already-archived halt; audit-drift failure → proceed with warning
- [ ] 9.10 Test pre-archive sweep on the bootstrap change after this work is applied  <!-- deferred: user-driven integration test per chunking plan -->

## 10. Conventions (cross-cutting)

- [x] 10.1 Document the `@review:` / `@review(escalated):` / `@todo:` marker conventions in `README.md` and a project-level doc (referenced from `CLAUDE.md`)
- [x] 10.2 Document the `openspec/lenses/{perspectives,critical-paths}.md` entry shapes and example content in the README
- [x] 10.3 Document `.orbit-runs/` directory contents and lifecycle (committed, dot-prefixed, travels with archive)
- [x] 10.4 Document the internal-run JSON summary format (one consistent shape across all commands)
- [x] 10.5 Document the external-review markdown findings format (consumed by `address-reviews --from-file`)
- [x] 10.6 Confirm 3-dimension scorecard usage is consistent across all review/audit commands (final review of generated SKILL.md files)
- [x] 10.7 Confirm severity ladder (CRITICAL/WARNING/SUGGESTION) and false-positive bias are consistent
- [x] 10.8 Confirm final-assessment phrasings match the stock forms across commands

## 11. Integration testing (operational)

> **All tasks in Group 11 are user-driven end-to-end tests against a fresh sandbox project, deferred from the apply chunking plan. Self-application against `bootstrap-openspec-orbit` would be circular (the change being tested IS the change implementing the commands). Run these in a separate fresh openspec-init'd project to validate orbit's behavior.**

- [ ] 11.1 End-to-end: run `/opsx:explore` to crystallization, capture conventions/perspectives/critical-paths, write `explore.md`  <!-- user-driven; fresh sandbox -->
- [ ] 11.2 End-to-end: run `/opsx:propose <name>` to consume the exploration; verify directory move and artifact generation  <!-- user-driven; fresh sandbox -->
- [ ] 11.3 End-to-end: run `/opsx:review <name> --as proposal` and confirm 9 passes execute with the scorecard  <!-- user-driven; fresh sandbox -->
- [ ] 11.4 End-to-end: run `/opsx:review-external <name> --as proposal`; manually paste prompt into codex; ingest findings via `/opsx:address-reviews --from-file <path>`  <!-- user-driven; fresh sandbox -->
- [ ] 11.5 End-to-end: run `/opsx:apply <name>` (upstream behavior, no orbit changes)  <!-- user-driven; fresh sandbox -->
- [ ] 11.6 End-to-end: run `/opsx:review <name> --as system` and confirm Pass 0 + Passes 1–6 execute  <!-- user-driven; fresh sandbox -->
- [ ] 11.7 End-to-end: run `/opsx:review-external <name> --as system`; ingest external findings  <!-- user-driven; fresh sandbox -->
- [ ] 11.8 End-to-end: run `/opsx:archive <name>` and confirm pre-archive audit-drift sweep runs; verify archive run summary written  <!-- user-driven; fresh sandbox -->
- [ ] 11.9 End-to-end: run `/opsx:audit-drift` standalone; verify findings across all four categories  <!-- user-driven; fresh sandbox -->

## 12. Documentation and release (operational)

- [x] 12.1 Final pass on `README.md` to ensure command reference matches shipped SKILL.md content
- [x] 12.2 Document install pattern (after `openspec init`, copy `.claude/commands/opsx/` and `.claude/skills/openspec-*/` into target project)
- [ ] 12.3 Update GitHub issues #1, #2, #3 with implementation status and links to landed PRs  <!-- user-driven; requires gh access to las-sal/openspec-orbit -->
- [ ] 12.4 Tag a v0.1.0 release on `las-sal/openspec-orbit` once integration tests pass  <!-- user-driven; depends on Group 11 passing -->
- [ ] 12.5 Make the repo public if the user decides it's ready for external adopters (currently private during v1)  <!-- user-driven; user decision -->
