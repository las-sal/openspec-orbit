## ADDED Requirements

### Requirement: Skill replaces upstream openspec-onboard at same location

The system SHALL ship an orbit-authored `openspec-onboard` skill at `.claude/skills/openspec-onboard/` that replaces upstream's `openspec-onboard` skill body entirely. The skill name and directory match upstream's so that the bound slash command `/opsx:onboard` works without rebinding.

#### Scenario: Same directory, same name, orbit-authored body

- **WHEN** orbit's overlay is applied to a project's `.claude/` directory
- **THEN** `.claude/skills/openspec-onboard/SKILL.md` contains 100% orbit-authored content; no upstream-derived body remains; no `# Orbit additions` section pattern is used (the entire skill is orbit's)

#### Scenario: Slash command surface unchanged

- **WHEN** a user invokes `/opsx:onboard`
- **THEN** `.claude/commands/opsx/onboard.md` resolves to the orbit-authored `openspec-onboard` skill; no other slash command or surface change is required

#### Scenario: Upstream onboard not separately preserved

- **WHEN** a user wants to read upstream's original onboarding tour
- **THEN** they SHOULD consult `@fission-ai/openspec`'s own documentation directly; orbit does not preserve upstream's onboard body as a sibling file

### Requirement: Skill body 5-section structure

The orbit-authored `openspec-onboard` SKILL.md body SHALL contain exactly 5 sections in canonical order: (1) Setup verification, (2) Identity statement, (3) Canonical-flow walkthrough, (4) Quick-reference command table, (5) Try-it nudge.

#### Scenario: All 5 sections present

- **WHEN** the SKILL.md is read end-to-end
- **THEN** each of the 5 canonical sections is present with its standard heading; reorderings or section omissions are spec violations

#### Scenario: No demo-change orchestration

- **WHEN** the skill executes (a user invokes `/opsx:onboard`)
- **THEN** the skill does NOT create any OpenSpec change, scaffold any `openspec/changes/<name>/` directory, or invoke other `/opsx:*` commands; it presents the 5-section content and ends

### Requirement: Setup verification section

The Setup verification section SHALL check that orbit is correctly installed and the upstream CLI matches orbit's pinned version. Verification SHALL be based on stable post-install artifacts only — NOT on workflow-state directories that may not exist on a fresh install.

#### Scenario: Verify overlay applied via stable post-install artifacts

- **WHEN** the Setup verification section executes
- **THEN** it confirms presence of orbit-authored skills + orbit-authored commands that are present immediately after a successful overlay install: at minimum `.claude/skills/openspec-review/SKILL.md`, `.claude/skills/openspec-audit-drift/SKILL.md`, `.claude/commands/opsx/review.md`, `.claude/commands/opsx/address-reviews.md`; verification does NOT depend on `openspec/lenses/` or `openspec/explore/` (those are workflow-state, created later by first use, and a correct install may have neither)

#### Scenario: Verify pruned upstream files are absent

- **WHEN** the Setup verification section executes
- **THEN** it confirms absence of upstream files orbit explicitly prunes from the overlay: at minimum `.claude/skills/feedback/` and `.claude/commands/opsx/sync.md` (per orbit-conventions `Overlay file disposition` for the NOT-shipped category); a correct install has these absent after the user runs the documented post-overlay-copy prune step

#### Scenario: Verify upstream version matches pin

- **WHEN** the Setup verification section executes
- **THEN** it runs `openspec --version` (or equivalent), compares against orbit's documented pinned version (per orbit-conventions `Upstream version pinning`), and reports match/mismatch with clear next-step guidance on mismatch

#### Scenario: Verification failures are informational

- **WHEN** the user's installation has issues (overlay incomplete, version mismatch, prune step not run)
- **THEN** the skill reports findings clearly and recommends fixes (including pointing to README's prune steps if pruned files are still present); the skill does NOT auto-repair or auto-install

### Requirement: Identity statement section

The Identity statement section SHALL convey what orbit IS, using the post-pegging forward-looking framing (per orbit-conventions `Distribution model — overlay, not CLI fork`).

#### Scenario: Workflow-tool framing

- **WHEN** the Identity section is read
- **THEN** it describes orbit as a workflow tool that owns the `.claude/` surface and uses `@fission-ai/openspec` at its pinned version as a CLI engine; framing avoids "an overlay that augments upstream cleanly" language that implies upstream-update compatibility

#### Scenario: Layer enumeration

- **WHEN** the Identity section is read
- **THEN** it enumerates orbit's distinctive layers: editorial review (`/opsx:review`, `/opsx:review-external`, `/opsx:address-reviews`), drift audit (`/opsx:audit-drift`), capture (lenses: perspectives, critical-paths), JSON run-summary emission, and execution disciplines (read-before-reference / change completeness / pushback)

### Requirement: Canonical-flow walkthrough section

The Canonical-flow walkthrough section SHALL present orbit's standard change lifecycle as a diagram plus one paragraph per phase, with contextual introduction of capture-layer lenses and abstract demonstration of the external-review loop.

#### Scenario: Canonical phases enumerated

- **WHEN** the walkthrough is read
- **THEN** the canonical flow is shown as: explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive; each phase has one paragraph describing its purpose and primary command

#### Scenario: Lenses introduced in explore-phase paragraph

- **WHEN** the explore-phase paragraph is read
- **THEN** it mentions that `/opsx:explore` may capture perspectives or critical-paths into `openspec/lenses/*.md` (per `openspec-explore` capture triggers); the reader learns lenses exist at the point in the flow where they're created

#### Scenario: Lenses re-referenced in review-phase paragraph

- **WHEN** the review-phase paragraph for `/opsx:review --as system` is read
- **THEN** it briefly notes that system-mode review consults `openspec/lenses/perspectives.md` and `openspec/lenses/critical-paths.md` (in Passes 4–5); the reader learns lenses are consumed at this point

#### Scenario: External-review loop demoed abstractly

- **WHEN** the review-phase or address-reviews-phase paragraph mentions external review
- **THEN** the walkthrough names the `/opsx:review-external` command, gives a one-paragraph what-it-does (packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`), and points readers to `openspec-review-external/SKILL.md` for full detail; no sample prompt file is bundled and no simulation runs

### Requirement: Quick-reference command table section

The Quick-reference command table section SHALL list all current `/opsx:*` slash commands with one-line descriptions.

#### Scenario: All current /opsx:* commands listed

- **WHEN** the Quick-reference table is read
- **THEN** every command currently shipped in `.claude/commands/opsx/` appears in the table with a one-line description of its purpose

#### Scenario: Table format is markdown

- **WHEN** the section is rendered
- **THEN** the commands appear in a markdown table with at least two columns: command and description; format is parseable for both human reading and AI ingest

#### Scenario: Origin information lives in orbit-conventions, not duplicated here

- **WHEN** a reader wants to know whether a specific command is orbit-authored, orbit-modified, or an upstream-required primitive
- **THEN** they consult `orbit-conventions`'s `Overlay file disposition` requirement; the orbit-onboard quick-reference table does NOT duplicate this classification information (single source of truth); the table MAY include a brief link or pointer to `Overlay file disposition` for readers who want to follow up

### Requirement: Try-it nudge section

The Try-it nudge section SHALL close the skill body by recommending the reader invoke `/opsx:explore` on a real idea they have, not on a demo or sandbox idea.

#### Scenario: Closing recommendation

- **WHEN** the Try-it nudge section is read
- **THEN** it explicitly recommends running `/opsx:explore <name>` on a real project idea the reader wants to work on; the recommendation framing avoids "for practice" language to discourage demo-change creation

#### Scenario: Future-extension hook

- **WHEN** orbit's contributors consider adding interactive-tour capability later
- **THEN** the Try-it nudge section is the designated extension point; an interactive prompt could replace or augment the current text-only nudge without restructuring the rest of the skill body

### Requirement: Reference-leaning hybrid style constraint

The orbit-authored `openspec-onboard` skill SHALL follow a reference-leaning hybrid style: setup verification + reference content with a single closing try-it nudge. The skill SHALL NOT orchestrate multi-phase workflows, create demo changes, or invoke other `/opsx:*` commands on behalf of the user.

#### Scenario: No demo change creation

- **WHEN** the skill executes
- **THEN** it does NOT create any `openspec/changes/<name>/`, `openspec/explore/<name>/`, or other persistent artifact in the user's repo; the user's filesystem state after invocation matches state before invocation, except for any voluntary action the user takes after reading

#### Scenario: No automated multi-phase orchestration

- **WHEN** the skill encounters opportunities to "show, not tell" (e.g., demonstrating `/opsx:explore` by actually running it)
- **THEN** the skill describes the command and links to the relevant SKILL.md rather than invoking it; the reader chooses when to leave the skill and try commands themselves

#### Scenario: Audience-first quality bar

- **WHEN** the skill body is reviewed
- **THEN** content quality (clear examples, accurate command syntax, well-explained rationale) is the primary review criterion; this skill is the entry point for new collaborators reading orbit cold and must function well in that context
