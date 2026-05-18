> **Convention**: tasks should reference the spec requirement they implement using the form `[R<#> in <capability>]` at the end of the task description, e.g., `[R3 in orbit-review-proposal]`. Tasks without spec references are **operational** (build/release/docs/testing) — they implement orbit's delivery rather than orbit's behavior, and are explicitly distinguished from spec-tied work in the group's introduction line. Annotations are added incrementally as implementation lands; tasks below are seeded without them.

## 1. Repository scaffolding (operational)

- [ ] 1.1 Confirm `LICENSE` (MIT), `README.md` (full workflow + command reference), `.gitignore` (excludes personal-transcript files and `settings.local.json`) are in place
- [ ] 1.2 Verify `openspec/config.yaml` reflects orbit-coherent settings (schema, optional project context, any orbit-specific rules)
- [ ] 1.3 Add a top-level `CLAUDE.md` snippet (or document in README) that adopters can copy as project-level context, including the pushback-discipline reminder

## 2. New skill: `/opsx:review-proposal`

- [ ] 2.1 Author `.claude/skills/openspec-review-proposal/SKILL.md` describing the 9 passes, scorecard rollup, flag family, pushback discipline, and graceful degradation per `specs/orbit-review-proposal/spec.md`
- [ ] 2.2 Author `.claude/commands/opsx/review-proposal.md` slash command body that surfaces the skill behavior to users with concise arg/flag descriptions
- [ ] 2.3 Implement `.orbit-runs/review-proposal-<TS>.json` summary writer (per-command summary fields per `orbit-conventions`)
- [ ] 2.4 Implement iteration-note generation when prior `review-proposal-<TS>.json` files exist
- [ ] 2.5 Implement `--mark` writer that converts each finding into a `@review:` marker at the finding's file:line
- [ ] 2.6 Implement `--parallel` subagent spawning for Passes 2, 4, 6
- [ ] 2.7 Test happy path against the orbit dev sandbox (this very repo) for change `bootstrap-openspec-orbit`

## 3. New skill: `/opsx:review-system`

- [ ] 3.1 Author `.claude/skills/openspec-review-system/SKILL.md` describing Pass 0 (verify-change delegation), Passes 1–6 (system-wide), scorecard rollup, flag family
- [ ] 3.2 Author `.claude/commands/opsx/review-system.md` slash command body
- [ ] 3.3 Implement Pass 0 delegation to upstream `/opsx:verify-change` via the openspec-verify-change skill
- [ ] 3.4 Implement `.orbit-runs/review-system-<TS>.json` summary writer
- [ ] 3.5 Implement iteration-note generation when prior `review-system-<TS>.json` files exist
- [ ] 3.6 Implement `--skip-verify` handling
- [ ] 3.7 Implement `--parallel` subagent spawning for Passes 2, 3, 4, 5
- [ ] 3.8 Implement Pass 6 invocation of `/opsx:audit-drift` as a library function
- [ ] 3.9 Test happy path on a small applied change in the orbit dev sandbox

## 4. New skill: `/opsx:review-external`

- [ ] 4.1 Author `.claude/skills/openspec-review-external/SKILL.md` describing handoff prompt generation
- [ ] 4.2 Author `.claude/commands/opsx/review-external.md` slash command body
- [ ] 4.3 Implement `--as proposal|system` flag with state-based inference from `tasks.md`
- [ ] 4.4 Implement iteration counting per mode (count of matching `external-<as>-*.md` files in `.orbit-runs/`)
- [ ] 4.5 Implement cycle-context population from prior `.orbit-runs/` files (open findings, resolved since last)
- [ ] 4.6 Author the proposal-mode prompt skeleton (lists the 9 review-proposal passes)
- [ ] 4.7 Author the system-mode prompt skeleton (lists the 7 review-system passes)
- [ ] 4.8 Include uncommitted-changes warning in chat output
- [ ] 4.9 Test prompt generation for both modes; manually validate that codex can follow it

## 5. New skill: `/opsx:audit-drift`

- [ ] 5.1 Author `.claude/skills/openspec-audit-drift/SKILL.md` describing the 4 scan categories, scorecard rollup, flag family, three invocation paths (standalone, library, pre-archive)
- [ ] 5.2 Author `.claude/commands/opsx/audit-drift.md` slash command body
- [ ] 5.3 Implement Category 1 (Vocabulary residue) — extract residue patterns from archived deltas; grep target docs
- [ ] 5.4 Implement Category 2 (Lens staleness) — resolve perspective surface refs against `openspec/specs/`; check critical-path touchpoints
- [ ] 5.5 Implement Category 3 (Cross-doc consistency) — extract structured claims; pairwise compare
- [ ] 5.6 Implement Category 4 (Archive coherence) — walk archived changes within window; verify ADDED/REMOVED propagation to baseline
- [ ] 5.7 Implement `--focus <area>` and `--since <ref>` flags
- [ ] 5.8 Implement context-aware final-assessment phrasings (standalone / pre-archive / library)
- [ ] 5.9 Implement summary writer for `.orbit-runs/audit-drift-<TS>.json` (change-scoped) and a project-level fallback location for standalone runs
- [ ] 5.10 Test happy path on this very repo (vocabulary residue, lens staleness, doc consistency, archive coherence)

## 6. New skill: `/opsx:address-reviews` (lean v1)

- [ ] 6.1 Author `.claude/skills/openspec-address-reviews/SKILL.md` describing the lean v1 lifecycle (discover → triage → walk → ripple flag → report)
- [ ] 6.2 Author `.claude/commands/opsx/address-reviews.md` slash command body
- [ ] 6.3 Implement default whole-repo `@review:` marker discovery with safe exclusions
- [ ] 6.4 Implement `--only <pattern>` scoping
- [ ] 6.5 Implement pushback discipline step (verify against current state before fixing)
- [ ] 6.6 Implement classification flow (trivial fix / decision / stale / unresolvable) with `AskUserQuestion` integration
- [ ] 6.7 Implement marker removal invariant (and `--keep-resolved-markers` debug override)
- [ ] 6.8 Implement ripple flag (list affected related files without auto-cascading)
- [ ] 6.9 Implement unresolvable-marker handling (default file as task in `tasks.md`; alternatives: `@todo:`, `@review(escalated):`)
- [ ] 6.10 Implement `--from-file <path>` parsing of external-review markdown into virtual markers
- [ ] 6.11 Implement resolution-log output (Resolved/Stale/Deferred/Escalated sections)
- [ ] 6.12 Test happy path: drop `@review:` markers in this very repo and run the command

## 7. Modified skill: `/opsx:explore`

- [ ] 7.1 Update `.claude/skills/openspec-explore/SKILL.md` to add orbit additions (capture affordances, explore.md authoring, three invocation modes) while preserving upstream's "stance, not workflow" character
- [ ] 7.2 Update `.claude/commands/opsx/explore.md` slash command body accordingly
- [ ] 7.3 Implement Mode A behavior (bare invocation — pure think; no file)
- [ ] 7.4 Implement Mode B behavior (named — create or resume `openspec/explore/<name>/explore.md`)
- [ ] 7.5 Implement Mode C behavior (bare-then-crystallized — prompt for name after ~2 substantive decisions emerge)
- [ ] 7.6 Implement convention-capture trigger and `<topic>_convention.md` writer
- [ ] 7.7 Implement perspective-capture trigger and `openspec/lenses/perspectives.md` appender (uses orbit-conventions entry shape)
- [ ] 7.8 Implement critical-path-capture trigger and `openspec/lenses/critical-paths.md` appender
- [ ] 7.9 Implement decision capture (proactive; with brief chat acknowledgment) into `explore.md` Decisions section
- [ ] 7.10 Implement reference capture into `explore.md` References section
- [ ] 7.11 Implement decline-tracking within the conversation (don't double-offer)
- [ ] 7.12 Implement group offers for related captures (e.g., three conventions in succession)

## 8. Modified skill: `/opsx:propose`

- [ ] 8.1 Update `.claude/skills/openspec-propose/SKILL.md` to add orbit additions (consume mode) while preserving upstream's standalone-mode behavior
- [ ] 8.2 Update `.claude/commands/opsx/propose.md` slash command body accordingly
- [ ] 8.3 Implement `openspec/explore/<name>/` detection; branch to consume mode when present
- [ ] 8.4 Implement section mapping (Premise → proposal motivation; Decisions → spec deltas + design + tasks; Considered & out → design alternatives; References → contextual reads)
- [ ] 8.5 Implement Open-question handling flow (per-question AskUserQuestion: resolve / defer-as-`@review:` / abandon)
- [ ] 8.6 Implement bulk-handle UX ("resolve all / defer all / walk each") for many Open questions
- [ ] 8.7 Implement directory move (`openspec/explore/<name>/` → `openspec/changes/<name>/`) after artifact generation
- [ ] 8.8 Implement conflict-detection (both staging and change dirs exist) with the three-way prompt (regenerate / continue / abort)
- [ ] 8.9 Implement structural validation of `explore.md` (warn on missing sections)
- [ ] 8.10 Implement naming inference (single recent staging → propose name to user)
- [ ] 8.11 Test consume mode end-to-end on a fresh exploration in the orbit dev sandbox

## 9. Modified skill: `/opsx:archive`

- [ ] 9.1 Update `.claude/skills/openspec-archive-change/SKILL.md` to add orbit additions (pre-archive audit-drift hook, `--skip-audit`) while preserving upstream archive behavior
- [ ] 9.2 Update `.claude/commands/opsx/archive.md` slash command body accordingly
- [ ] 9.3 Implement pre-archive `/opsx:audit-drift` invocation as a library call
- [ ] 9.4 Implement critical-drift three-way prompt (address now / proceed / abort) via `AskUserQuestion`
- [ ] 9.5 Implement `--skip-audit` flag handling
- [ ] 9.6 Implement archive-run-summary writer at `openspec/changes/archive/<name>/.orbit-runs/archive-<TS>.json` with audit and sync-specs results plus user decision
- [ ] 9.7 Implement `.orbit-runs/` directory move alongside the change content
- [ ] 9.8 Implement unresolved-`@review:`-marker warning before completing archive
- [ ] 9.9 Implement edge cases: already-archived halt; audit-drift failure → proceed with warning
- [ ] 9.10 Test pre-archive sweep on the bootstrap change after this work is applied

## 10. Conventions (cross-cutting)

- [ ] 10.1 Document the `@review:` / `@review(escalated):` / `@todo:` marker conventions in `README.md` and a project-level doc (referenced from `CLAUDE.md`)
- [ ] 10.2 Document the `openspec/lenses/{perspectives,critical-paths}.md` entry shapes and example content in the README
- [ ] 10.3 Document `.orbit-runs/` directory contents and lifecycle (committed, dot-prefixed, travels with archive)
- [ ] 10.4 Document the internal-run JSON summary format (one consistent shape across all commands)
- [ ] 10.5 Document the external-review markdown findings format (consumed by `address-reviews --from-file`)
- [ ] 10.6 Confirm 3-dimension scorecard usage is consistent across all review/audit commands (final review of generated SKILL.md files)
- [ ] 10.7 Confirm severity ladder (CRITICAL/WARNING/SUGGESTION) and false-positive bias are consistent
- [ ] 10.8 Confirm final-assessment phrasings match the stock forms across commands

## 11. Integration testing (operational)

- [ ] 11.1 End-to-end: run `/opsx:explore` to crystallization, capture conventions/perspectives/critical-paths, write `explore.md`
- [ ] 11.2 End-to-end: run `/opsx:propose <name>` to consume the exploration; verify directory move and artifact generation
- [ ] 11.3 End-to-end: run `/opsx:review-proposal <name>` and confirm 9 passes execute with the scorecard
- [ ] 11.4 End-to-end: run `/opsx:review-external <name> --as proposal`; manually paste prompt into codex; ingest findings via `/opsx:address-reviews --from-file <path>`
- [ ] 11.5 End-to-end: run `/opsx:apply <name>` (upstream behavior, no orbit changes)
- [ ] 11.6 End-to-end: run `/opsx:review-system <name>` and confirm Pass 0 + Passes 1–6 execute
- [ ] 11.7 End-to-end: run `/opsx:review-external <name> --as system`; ingest external findings
- [ ] 11.8 End-to-end: run `/opsx:archive <name>` and confirm pre-archive audit-drift sweep runs; verify archive run summary written
- [ ] 11.9 End-to-end: run `/opsx:audit-drift` standalone; verify findings across all four categories

## 12. Documentation and release (operational)

- [ ] 12.1 Final pass on `README.md` to ensure command reference matches shipped SKILL.md content
- [ ] 12.2 Document install pattern (after `openspec init`, copy `.claude/commands/opsx/` and `.claude/skills/openspec-*/` into target project)
- [ ] 12.3 Update GitHub issues #1, #2, #3 with implementation status and links to landed PRs
- [ ] 12.4 Tag a v0.1.0 release on `las-sal/openspec-orbit` once integration tests pass
- [ ] 12.5 Make the repo public if the user decides it's ready for external adopters (currently private during v1)
