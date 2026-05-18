# orbit-explore-modifications

## Purpose

Orbit modifications to `/opsx:explore` — preserves upstream's "stance, not workflow" character (think-mode, no fixed steps, no mandatory outputs, no implementation) and adds: three invocation modes (bare / named / crystallized), five capture types with offer-don't-auto triggers (decisions, conventions, perspectives, critical paths, references), `explore.md` five-section convention (Premise / Decisions / Open questions / Considered & out / References), sibling captures (sketches/ + draft conventions), decline-tracking to avoid double-offering.

## Requirements

### Requirement: Upstream explore stance preserved

The system SHALL preserve upstream `/opsx:explore`'s "stance, not workflow" character — thinking-partner mode, no fixed steps, no mandatory outputs, no implementation.

#### Scenario: Bare invocation

- **WHEN** the user invokes `/opsx:explore` with no argument and no orbit-specific capture trigger fires
- **THEN** the command behaves exactly as upstream: conversational think-mode, no file created

#### Scenario: Implementation guardrail preserved

- **WHEN** the user asks the command to implement code or apply features
- **THEN** the command refuses and reminds the user to exit explore mode (same as upstream)

### Requirement: Three invocation modes supported

The system SHALL support three invocation modes: (A) bare — no file; (B) named — `/opsx:explore <name>`; (C) crystallized — bare invocation that becomes named midway.

#### Scenario: Mode A — bare

- **WHEN** `/opsx:explore` is invoked with no argument
- **THEN** the command enters think-mode with no `explore.md` file; no staging directory is created

#### Scenario: Mode B — new named exploration

- **WHEN** `/opsx:explore <name>` is invoked and `openspec/explore/<name>/` does not exist
- **THEN** the command creates the staging directory and writes a stub `explore.md` with the five sections (Premise / Decisions / Open questions / Considered & out / References); ongoing conversation populates the file

#### Scenario: Mode B — resume

- **WHEN** `/opsx:explore <name>` is invoked and `openspec/explore/<name>/explore.md` already exists
- **THEN** the command reads the existing file for context and resumes the exploration

#### Scenario: Mode C — crystallization prompt

- **WHEN** the user has been exploring in Mode A and the conversation has produced 2 or more substantive decisions
- **THEN** the command prompts the user: "We have enough material here to capture — what should we call this exploration?"

#### Scenario: Definition of "substantive decision"

- **WHEN** the AI is counting decisions toward the crystallization threshold
- **THEN** a decision is counted as substantive if it: (a) resolves between two or more named alternatives (e.g., "go with X instead of Y"), (b) locks a name, structure, or format ("we'll call it Z", "the file will have these sections"), or (c) supersedes an earlier choice ("change our mind on X, go with Y instead"); decisions that don't count: exploratory thinking, speculation, "let me think about X", or restating something already established

#### Scenario: Mode C — user accepts name

- **WHEN** the user provides a name in response to the crystallization prompt
- **THEN** the command creates `openspec/explore/<name>/explore.md`, back-fills the Premise and Decisions sections with what's been discussed, and continues as Mode B

#### Scenario: Mode C — user declines

- **WHEN** the user declines the crystallization prompt
- **THEN** the command continues in Mode A and does not re-prompt for this shape within the conversation

### Requirement: Five capture types with capture triggers

The system SHALL recognize capture-worthy content in the conversation and offer to write it to the appropriate file. Five capture types: conventions, perspectives, critical paths, decisions, references.

#### Scenario: Convention capture

- **WHEN** the user says something that reads as a durable convention (e.g., "we always do X", "X should be named Y", "let's standardize on…")
- **THEN** the command offers to capture it in a topic-specific convention file at project root (e.g., `naming_convention.md`); on accept, the file is created or appended; on decline, the command continues

#### Scenario: Convention capture uses the structured format

- **WHEN** a convention capture is accepted and the AI writes to `<topic>_convention.md`
- **THEN** the file follows the four-section structured format defined in orbit-conventions (Purpose / Rules / Examples / Exceptions); when creating a new file, all sections are scaffolded; when appending to an existing file, the new rule is added to the appropriate section (typically Rules)

#### Scenario: Convention update offered on contradiction

- **WHEN** the user's statement contradicts an existing rule in a `*_convention.md` file
- **THEN** the command surfaces the contradiction and offers to update the convention rather than silently overwriting; user decides whether to update, supersede, or leave the existing rule in place

#### Scenario: Heuristic for new file vs. append

- **WHEN** a convention capture matches the topic of an existing file (e.g., user mentions naming and `naming_convention.md` exists)
- **THEN** the offer targets the existing file; only when no matching topic file exists is a new file proposed

#### Scenario: Perspective capture

- **WHEN** the user describes a caller or client of the system (e.g., "Claude Desktop calls our MCP server", "from the Swift host's POV")
- **THEN** the command offers to capture it in `openspec/lenses/perspectives.md` with the structured entry shape (Surfaces, Description, Typical call patterns)

#### Scenario: Critical-path capture

- **WHEN** the user describes a critical user flow (e.g., "the typical user flow is...", "users typically...")
- **THEN** the command offers to capture it in `openspec/lenses/critical-paths.md` with the structured entry shape (Description, Touchpoints, Expected behavior)

#### Scenario: Decision capture (Mode B/C only)

- **WHEN** the conversation produces an explicit decision in an active named exploration
- **THEN** the command proactively captures the decision in the `Decisions` section of `openspec/explore/<name>/explore.md` and acknowledges in chat ("captured")

#### Scenario: User vetoes decision capture

- **WHEN** the user says "don't capture that" or similar
- **THEN** the proposed capture is not written; the conversation continues unaffected

### Requirement: Offer-don't-auto for non-decision captures

The system SHALL offer rather than auto-write for convention, perspective, critical-path, and reference captures. Decisions are proactively captured with acknowledgment.

#### Scenario: Offer phrased clearly

- **WHEN** a capture-worthy convention/perspective/critical-path is detected
- **THEN** the command pauses briefly and emits a one-sentence offer naming the target file

#### Scenario: Group offer when natural

- **WHEN** multiple capture-worthy items of the same type emerge in close succession (e.g., three conventions)
- **THEN** the command groups them into a single offer rather than asking three times

### Requirement: explore.md five-section convention

The system SHALL maintain `openspec/explore/<name>/explore.md` with five sections in fixed order: Premise, Decisions, Open questions, Considered & out, References.

#### Scenario: Initial stub structure

- **WHEN** a new `explore.md` is created
- **THEN** all five sections are present with empty content under headings; the file includes the status header `> **Status**: exploring. Promoted to proposal/design/specs via /opsx:propose when decisions firm up.`

#### Scenario: Resolving an Open question

- **WHEN** an Open question is resolved during the conversation
- **THEN** the command moves the entry from `Open questions` to `Decisions` with a dated rationale, and acknowledges in chat

#### Scenario: Rejecting a considered option

- **WHEN** an option discussed is explicitly rejected
- **THEN** the command moves it to `Considered & out` with a brief rationale and date

### Requirement: Sibling captures supported

The system SHALL accept additional files in `openspec/explore/<name>/` beyond `explore.md` (e.g., `sketches/*.md`, draft convention files).

#### Scenario: Sketch directory

- **WHEN** a design sketch is captured during exploration
- **THEN** it can be written to `openspec/explore/<name>/sketches/<sketch-name>.md` and persists there until promotion

### Requirement: Decline tracking within conversation

The system SHALL track recent capture declines within a single conversation to avoid double-offering.

#### Scenario: Declined convention not re-offered

- **WHEN** the user has just declined a convention capture for a specific topic
- **THEN** the command does not re-offer that convention later in the same conversation
