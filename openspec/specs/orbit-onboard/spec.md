# orbit-onboard

## Purpose

Orbit-authored onboarding walkthrough that loads orbit's identity, canonical 9-phase workflow, command surface, and getting-started nudge into an AI session (or human reader) in a single reference-leaning pass. Replaces upstream's guided-tour body with orbit-specific content tuned for AI cold-load, future-self refresh, and collaborator handoff.

## Requirements

### Requirement: Skill body 5-section structure

The orbit-authored `openspec-onboard` SKILL.md SHALL have exactly 5 sections in this order: Setup verification, Identity statement, Canonical-flow walkthrough, Quick-reference command table, Try-it nudge. The 5-section structure is the reference-leaning hybrid walkthrough style — single-pass read with no interactive demo, no auto-generated sample change, no simulation of cross-AI loops.

#### Scenario: Sections appear in order

- **WHEN** a reader (human or AI) reads `.claude/skills/openspec-onboard/SKILL.md`
- **THEN** the body presents Section 1 (Setup verification), then Section 2 (Identity statement), then Section 3 (Canonical-flow walkthrough), then Section 4 (Quick-reference command table), then Section 5 (Try-it nudge), in that order; sections do not interleave; section boundaries are clearly delimited by H2 headings

#### Scenario: Single-pass reference, not interactive

- **WHEN** a user invokes `/opsx:onboard`
- **THEN** the skill executes Section 1 (Setup verification) as gating logic, then renders Sections 2-5 as a sequential read; the skill does NOT create demo changes in the user's project, does NOT auto-invoke other `/opsx:*` commands, does NOT prompt the user for interactive choices outside Section 1's verification

#### Scenario: Audience-tuned for AI cold-load and human reference

- **WHEN** the SKILL body is authored or revised
- **THEN** the prose is tuned for the audience triple: (a) AI sessions loading orbit context cold, (b) future-self after a context break, (c) eventual collaborators / handoff readers. Content is parseable workflow documentation, NOT a tutorial requiring step-by-step execution

### Requirement: Setup verification section

Section 1 of the SKILL body SHALL perform setup verification before any walkthrough content renders. Verification has three outcomes (hard-stop, warn-continue, pass-with-layered-checks) keyed to specific failure modes.

#### Scenario: Hard-stop on overlay-incomplete

- **WHEN** the user's `.claude/` state is missing any required orbit-shipped skill or command (failure modes: overlay step skipped entirely, partial overlay, wrong `--tools` install profile, `openspec-sync-specs` missing) OR `openspec --version` does not match the pinned upstream version per `orbit-conventions` `Upstream version pinning` requirement
- **THEN** Section 1 emits a lumped "overlay incomplete; see README install section #2" error message pointing the user at the documented install steps, AND halts the onboard session — Sections 2-5 do NOT render

#### Scenario: Warn-and-continue on prune-step-skipped

- **WHEN** the user's `.claude/skills/feedback/` directory is present (indicating the documented `rm -rf .claude/skills/feedback` prune step from the README install section was not run after the overlay)
- **THEN** Section 1 emits an inline `⚠` warning with the remediation hint (`rm -rf .claude/skills/feedback`) and proceeds to Sections 2-5; the user is informed but not blocked

#### Scenario: Pass with layered ✓ checks

- **WHEN** the user's `.claude/` state matches the documented post-install surface (15 skills + 15 commands total — 9 upstream-modified + 5 orbit-authored + 1 upstream-required primitive `openspec-sync-specs` skills; 9 orbit-modified + 5 orbit-authored + 1 upstream-untouched (`ff.md`) commands — after this change archives. Command total drops from 16 to 15 because orbit's verbatim-duplicate `fast-forward.md` is removed from the overlay per `orbit-conventions` `Verbatim upstream files not in orbit's overlay`; upstream's `ff.md` is what users have for the fast-forward workflow)
- **THEN** Section 1 emits a layered ✓ output mirroring the four `orbit-conventions` `Overlay file disposition` categories: ✓ openspec CLI version match; ✓ N upstream-modified skills present (where N matches the disposition baseline); ✓ M orbit-authored skills present (where M matches baseline); ✓ openspec-sync-specs upstream-required primitive present; ✓ feedback/ absent (prune-step verified). Then proceeds to Sections 2-5.

#### Scenario: Lumped messaging for overlay-incomplete sub-modes

- **WHEN** any hard-stop sub-mode fires (overlay-incomplete, partial overlay, wrong `--tools` profile, `openspec-sync-specs` alone missing, OR wrong upstream version)
- **THEN** the emitted message does NOT distinguish between sub-modes; a single lumped message points to the README's `## Installation` section as a whole (both `### 1. Initialize upstream OpenSpec` for version-pin issues and `### 2. Overlay orbit` for overlay-incomplete issues). The user re-reads install steps from the top; this is the trade-off accepted by the lumping decision (per design D-onboard-4) — slightly broader remediation reading in exchange for one detection branch + one prose message vs five sub-variants

#### Scenario: Verification cannot detect stale AI client cache

- **WHEN** the user's `.claude/` state is correct on disk but the AI client has cached an older `.claude/` snapshot
- **THEN** Section 1's checks pass (it cannot read the client's cache state); the SKILL body's gotcha section MAY mention this limitation with the remediation "restart your AI client after install"; this is a documented limitation, not a verification responsibility

### Requirement: Identity statement section

Section 2 of the SKILL body SHALL state what orbit IS using post-pegging framing and enumerate orbit-distinctive layers.

#### Scenario: Post-pegging framing

- **WHEN** the Identity section is read
- **THEN** the prose positions orbit as a workflow tool that owns the `.claude/` surface and uses `@fission-ai/openspec` at the pinned version as a CLI engine; the prose does NOT use augmentation language (`augments cleanly`, `overlay on top of`, `layered on top of`, `extends upstream`) — orbit is a distinct workflow tool that shares concepts with upstream, not an additive layer

#### Scenario: Enumerates orbit-distinctive layers

- **WHEN** the Identity section is read
- **THEN** the prose names the orbit-distinctive layers concretely: editorial review (`/opsx:review`, `/opsx:review-external`, `/opsx:address-reviews`), drift audit (`/opsx:audit-drift`), capture (lenses — `openspec/lenses/perspectives.md`, `openspec/lenses/critical-paths.md`), JSON run-summary emission, three execution disciplines (read-before-reference / change completeness / pushback)

### Requirement: Canonical-flow walkthrough section

Section 3 of the SKILL body SHALL present orbit's 9-phase canonical workflow as an ASCII diagram followed by one paragraph per phase.

#### Scenario: 9-phase diagram

- **WHEN** the Canonical-flow section is read
- **THEN** an ASCII diagram (or equivalent markdown structure) shows the phases in order: explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive

#### Scenario: One paragraph per phase

- **WHEN** the Canonical-flow section is read
- **THEN** each of the 9 phases has one paragraph describing the phase's purpose, the primary `/opsx:*` command that drives it, and the typical output the user will see

#### Scenario: Lenses introduced contextually

- **WHEN** the Canonical-flow section is read
- **THEN** lenses (`perspectives.md` + `critical-paths.md`) are introduced in the explore-phase paragraph (where lenses are captured per `openspec-explore`'s capture triggers) and re-referenced in the review-phase paragraphs (where `/opsx:review --as system` Passes 4-5 consult them)

#### Scenario: External-review demoed abstractly

- **WHEN** the Canonical-flow section's review or address-reviews phase paragraphs are read
- **THEN** the external-review loop is named (`/opsx:review-external`) with a one-paragraph what-it-does (packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`) and a pointer to `.claude/skills/openspec-review-external/SKILL.md` for details; NO bundled sample prompt artifact is included in the SKILL body; NO simulated cross-AI exchange is rendered

### Requirement: Quick-reference command table section

Section 4 of the SKILL body SHALL list every `/opsx:*` command in orbit's overlay with a one-line description in a markdown table.

#### Scenario: Complete enumeration

- **WHEN** the Quick-reference section is read
- **THEN** the table lists every slash command in the user's post-install `.claude/commands/opsx/` (15 files: 14 from orbit's overlay + 1 from upstream init untouched). Enumeration: `address-reviews`, `apply`, `archive`, `audit-drift`, `bulk-archive`, `continue`, `explore`, `ff` (upstream-installed, untouched by overlay — orbit's previous `fast-forward.md` was removed per the verbatim-duplicates principle), `new`, `onboard`, `propose`, `review`, `review-external`, `sync`, `verify`; each row has the command name and a one-line description

#### Scenario: Table does NOT include a disposition column

- **WHEN** the Quick-reference table is rendered
- **THEN** the table does NOT include a column distinguishing orbit-authored / orbit-modified / upstream-required-primitive / upstream-untouched (that classification is canonical in `orbit-conventions` `Overlay file disposition`; duplicating it here creates drift risk). The table MAY include a brief pointer line to `Overlay file disposition` for readers who want categorization

### Requirement: Try-it nudge section

Section 5 of the SKILL body SHALL conclude with two closing recommendations covering both audiences: users with a concrete project idea AND users orienting without a concrete idea yet.

#### Scenario: Named-mode recommendation for concrete-idea users

- **WHEN** the Try-it nudge section is read
- **THEN** one recommendation is "If you have a concrete project idea, run `/opsx:explore <name>`" — pointing users with substantive work to start named-mode explore immediately. The recommendation does NOT use "for practice" or "try a demo" framing (no synthetic-change creation)

#### Scenario: Bare-mode recommendation for orientation-only users

- **WHEN** the Try-it nudge section is read
- **THEN** a second recommendation is "If you're just orienting and don't have a concrete idea yet, run `/opsx:explore` (no name)" — pointing orientation-only users to bare-mode explore as a no-decision-yet entry. This preserves the no-demo-change discipline (named-mode forces a name; bare-mode is thinking-mode without commitment)

#### Scenario: Try-it nudge is the section designated for future extension

- **WHEN** future orbit work explores an interactive guided-tour enhancement to onboard (currently rejected as over-built per design.md Non-Goals)
- **THEN** the Try-it nudge section is the natural extension point — it MAY grow to invite the user into an interactive prompt without rewriting Sections 1-4

### Requirement: Non-emission of run-summary JSON

The orbit-authored `openspec-onboard` SKILL SHALL NOT emit a run-summary JSON to `openspec/.orbit-runs/` (or any per-change `.orbit-runs/`) when invoked. This composes with the existing `orbit-run-summary-emit` baseline `Emit scope` requirement, which lists `/opsx:onboard` as a non-emit command.

#### Scenario: Brief metadata note in SKILL body asserts non-emission

- **WHEN** the SKILL body is authored or revised
- **THEN** a brief metadata note (one or two lines) placed at the SKILL body's discretion (footer, near-frontmatter, or other natural reading position — flexibility is intentional) explicitly states that `/opsx:onboard` does NOT emit run-summary JSON, citing `orbit-run-summary-emit` `Emit scope`; the note prevents future orbit-additions or refactors from silently introducing emit behavior

#### Scenario: No `.orbit-runs/` writes during invocation

- **WHEN** a user invokes `/opsx:onboard` (regardless of verification outcome)
- **THEN** no JSON file is written to `openspec/.orbit-runs/`, `openspec/changes/<name>/.orbit-runs/`, or any per-change subdirectory; the SKILL's effects are limited to producing the verification output and rendering Sections 2-5 (or halting at Section 1 on hard-stop)

### Requirement: Command-file body matches SKILL.md body (duplicate-pattern discipline)

The `.claude/commands/opsx/onboard.md` slash-command file SHALL carry the same body as `.claude/skills/openspec-onboard/SKILL.md`, differing only in frontmatter. This follows the existing convention for orbit-authored command/skill pairs and is tracked for cross-cutting cleanup as [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29).

#### Scenario: Bodies are duplicate prose, frontmatter differs

- **WHEN** a reader compares `.claude/commands/opsx/onboard.md` to `.claude/skills/openspec-onboard/SKILL.md` (excluding the frontmatter block at the top of each)
- **THEN** the prose body is identical: same Section 1 verification logic, same Section 2 identity, same Section 3 walkthrough paragraphs, same Section 4 table, same Section 5 nudge. Differences are limited to frontmatter (the command file has `category` + `tags`; the SKILL has `license` + `compatibility` + `metadata`)

#### Scenario: Edit-time discipline — update both files

- **WHEN** an orbit contributor revises either `.claude/commands/opsx/onboard.md` OR `.claude/skills/openspec-onboard/SKILL.md`
- **THEN** the contributor SHALL update the other file in the same edit (or in a same-change task) so the bodies stay in sync; review processes that surface drift between the two files treat the drift as a CRITICAL finding

#### Scenario: Discipline is transitional, tracked for refactor

- **WHEN** orbit considers refactoring orbit-authored skill/command pairs to a single-source pattern
- **THEN** the work is tracked at [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29) and applies to all 5 orbit-authored pairs (review, review-external, audit-drift, address-reviews, onboard); resolving #29 SHALL be its own change, not bundled with onboard-specific work

### Requirement: Slash-command surface `/opsx:onboard`

The orbit-authored skill SHALL preserve the `/opsx:onboard` slash-command surface; the new body replaces the upstream-bodied content without changing the user-facing command name or invocation pattern.

#### Scenario: Same directory, same name

- **WHEN** orbit ships the new orbit-authored body
- **THEN** the skill lives at `.claude/skills/openspec-onboard/` (matching sibling `openspec-*` naming convention per `orbit-conventions` `Overlay file disposition` `Orbit-authored — full ownership` scenario); the slash command is `/opsx:onboard` (matching pre-rewrite invocation); no transitional rename to `orbit-onboard` or `openspec-onboard-orbit` is performed

#### Scenario: Invocation unchanged for users

- **WHEN** a user types `/opsx:onboard` in their AI client (regardless of prior orbit version)
- **THEN** the command resolves to the orbit-authored skill body; users familiar with the pre-rewrite invocation pattern do not need to learn a new command surface

### Requirement: Identity section MUST NOT contain augmentation language

The Identity statement section (Section 2) SHALL avoid language that frames orbit as an additive layer on upstream (e.g., "augments cleanly", "overlay on top of", "layered on top of", "extends upstream OpenSpec"). This composes with the post-pegging framing in `orbit-conventions` `Distribution model — pegged engine + orbit-owned surface`.

#### Scenario: Negative test — augmentation language is absent

- **WHEN** the Identity section is reviewed (manually or via grep at apply time)
- **THEN** the prose does not contain the phrases `augments cleanly`, `overlay on top of`, `layered on top of`, or `extends upstream` (in the senses described above); a grep `grep -E 'augments cleanly|overlay on top of|layered on top of|extends upstream' .claude/skills/openspec-onboard/SKILL.md` returns no matches

#### Scenario: Positive framing — workflow tool with pinned engine

- **WHEN** the Identity section is read
- **THEN** the prose positions orbit as a workflow tool that owns the `.claude/` surface and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine — the same framing established in `orbit-conventions` `Distribution model — pegged engine + orbit-owned surface`
