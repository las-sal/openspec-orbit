# orbit-propose-modifications

## Purpose

Orbit modifications to `/opsx:propose` — preserves upstream's standalone behavior (description → artifacts) and adds **consume mode** that activates when `openspec/explore/<name>/explore.md` exists. Consume mode reads explore.md as the authoritative seed, applies section-to-artifact mapping (Premise → motivation, Decisions → spec deltas + design + tasks, Considered & out → design alternatives, References → contextual reads), prompts user per Open question with three resolution paths (resolve / defer as `@review:` marker / abandon), generates artifacts, and then *moves* the staging directory to `openspec/changes/<name>/` — preserving `explore.md` as historical record.

## Requirements

### Requirement: Upstream propose behavior preserved in standalone mode

The system SHALL preserve upstream `/opsx:propose`'s behavior when no `openspec/explore/<name>/` staging directory exists.

#### Scenario: Standalone mode

- **WHEN** the user invokes `/opsx:propose <name>` and `openspec/explore/<name>/` does not exist
- **THEN** the command behaves exactly as upstream: takes a description, generates `proposal.md` / `design.md` / `specs/<capability>/spec.md` / `tasks.md` into `openspec/changes/<name>/`

### Requirement: Consume mode when staging directory exists

The system SHALL detect the presence of `openspec/explore/<name>/explore.md` and switch to consume mode.

#### Scenario: Consume mode activated

- **WHEN** the user invokes `/opsx:propose <name>` and `openspec/explore/<name>/explore.md` exists
- **THEN** the command reads `explore.md` as the authoritative seed for artifact generation rather than asking for a description

#### Scenario: Conflict — both dirs exist

- **WHEN** both `openspec/explore/<name>/` and `openspec/changes/<name>/` exist at invocation
- **THEN** the command halts with a conflict report and asks the user to choose: regenerate from explore (overwrite change), continue from change (discard explore staging), or abort

### Requirement: Section-to-artifact mapping

The system SHALL map `explore.md` sections to the generated artifacts as follows.

#### Scenario: Premise → proposal motivation

- **WHEN** `explore.md` has a Premise section
- **THEN** the content is used to seed `proposal.md`'s "Why" / motivation section

#### Scenario: Decisions → spec deltas + design + tasks

- **WHEN** `explore.md` has Decisions
- **THEN** the decisions inform spec requirements, design.md decisions, and tasks; specific wording from the Decisions is preserved where artifact format permits

#### Scenario: Considered & out → design alternatives

- **WHEN** `explore.md` has Considered & out entries
- **THEN** they are surfaced in `design.md`'s "Alternatives considered" or equivalent section to prevent rediscovery

#### Scenario: References → contextual reads

- **WHEN** `explore.md` lists References
- **THEN** those files/URLs are read during artifact generation as context (not copied verbatim)

### Requirement: Open question handling

The system SHALL prompt the user per Open question in `explore.md` with three resolution paths: resolve now, defer as `@review:` marker, or abandon.

#### Scenario: Resolve now

- **WHEN** the user resolves an Open question during the prompt
- **THEN** the question's resolution becomes a Decision and informs the generated artifacts; the Open question is moved to Decisions in the persisted `explore.md`

#### Scenario: Defer

- **WHEN** the user defers an Open question
- **THEN** a `@review: <question text>` marker is inserted at the most relevant location in the generated artifacts (typically design.md or the relevant spec); the Open question is moved to a "Deferred to markers" note or remains in Open questions with a "deferred" tag

#### Scenario: Abandon

- **WHEN** the user abandons an Open question (decides it doesn't apply to this change)
- **THEN** the question is moved to Considered & out with a brief rationale; no marker is created

#### Scenario: Default for unprompted bulk

- **WHEN** many Open questions exist and the user chooses "defer all"
- **THEN** all open questions become `@review:` markers in the generated artifacts

### Requirement: Move staging directory to changes

The system SHALL move `openspec/explore/<name>/` to `openspec/changes/<name>/` after generating the artifacts.

#### Scenario: Standard move

- **WHEN** consume mode completes artifact generation successfully
- **THEN** the contents of `openspec/explore/<name>/` (including `explore.md` and any sibling files like `sketches/`) are moved into `openspec/changes/<name>/`; the source staging directory no longer exists

#### Scenario: Generated artifacts coexist with moved content

- **WHEN** the move completes
- **THEN** `openspec/changes/<name>/` contains both the moved exploration files (`explore.md`, `sketches/`, etc.) AND the generated artifacts (`proposal.md`, `design.md`, `specs/`, `tasks.md`)

### Requirement: explore.md persists as historical record

The system SHALL preserve `explore.md` and its sibling files unchanged in the change directory after the move.

#### Scenario: No paraphrasing during move

- **WHEN** `explore.md` is moved into `openspec/changes/<name>/`
- **THEN** its content is unchanged; specific decision wording is preserved exactly

#### Scenario: Cite explore source

- **WHEN** the generated `design.md` references background or decisions originally captured in `explore.md`
- **THEN** the generated text cites `explore.md` rather than paraphrasing without attribution

### Requirement: Validate explore.md structure before consuming

The system SHALL check the basic structure of `explore.md` and warn (but not block) on issues.

#### Scenario: All five sections present

- **WHEN** `explore.md` has all five expected sections (Premise / Decisions / Open questions / Considered & out / References)
- **THEN** the command proceeds normally

#### Scenario: Missing section warning

- **WHEN** `explore.md` is missing one or more expected sections
- **THEN** the command warns about the missing sections and prompts the user to confirm proceed-anyway (default proceed)

#### Scenario: Malformed file

- **WHEN** `explore.md` cannot be read or parsed
- **THEN** the command halts with a clear error and exit; user fixes and re-invokes

### Requirement: Naming inference fallback

The system SHALL attempt to infer the change name when invoked without one.

#### Scenario: Recent explore activity

- **WHEN** `/opsx:propose` is invoked with no name and exactly one `openspec/explore/<name>/` directory exists
- **THEN** the command proposes that name to the user; on accept, proceeds in consume mode

#### Scenario: Multiple or no candidates

- **WHEN** zero or multiple staging directories exist
- **THEN** the command prompts via `AskUserQuestion` to specify a name (zero candidates) or pick among them (multiple)
