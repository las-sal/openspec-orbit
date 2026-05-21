## ADDED Requirements

### Requirement: Emit scope

The following orbit commands SHALL emit `openspec/changes/<name>/.orbit-runs/<command>-<TS>.json` (or `openspec/explore/<name>/.orbit-runs/<command>-<TS>.json` for explore) when invoked. This requirement covers two categories:

- **New emit behavior** (commands that don't emit any run-summary JSON today):
  - Workflow commands: `/opsx:explore` (named mode only — bare mode does NOT emit), `/opsx:propose`, `/opsx:new`, `/opsx:continue`, `/opsx:ff`, `/opsx:apply`, `/opsx:verify`
  - `/opsx:review-external` at T0 (when the prompt is packaged, before findings return — today it writes only the `.md` prompt file)
- **Refined existing emit** (already emits today; this change refines the recommendation logic and aligns with the universal spine from orbit-conventions):
  - Standalone `/opsx:audit-drift` (already documented at `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`)

The following commands SHALL NOT emit additional JSONs as part of this change:

- `/opsx:bulk-archive` — wrapper; each inner `/opsx:archive` invocation emits separately
- `/opsx:onboard` — meta walkthrough; no change-state transition
- `/opsx:sync-specs` — deprecated upstream; slated for removal by openspec-orbit#6

Existing emit-producing commands (`/opsx:review`, `/opsx:address-reviews`, `/opsx:archive`, inline `/opsx:audit-drift` during archive) continue emitting as today; this change does not modify their emit behavior.

#### Scenario: Named-mode /opsx:explore writes explore JSON
- **WHEN** the user invokes `/opsx:explore foo` (named mode) and the session ends
- **THEN** the emit-layer writes `openspec/explore/foo/.orbit-runs/explore-<TS>.json`

#### Scenario: Bare-mode /opsx:explore writes no JSON
- **WHEN** the user invokes `/opsx:explore` without a name argument
- **THEN** no `.orbit-runs/` JSON is written for that conversation

#### Scenario: /opsx:onboard does not emit
- **WHEN** the user invokes `/opsx:onboard` and completes the walkthrough
- **THEN** no `.orbit-runs/onboard-<TS>.json` is written anywhere in the project

#### Scenario: /opsx:bulk-archive relies on inner /opsx:archive emits
- **WHEN** the user invokes `/opsx:bulk-archive` to archive 3 changes
- **THEN** 3 `archive-<TS>.json` files are written (one per inner archive), and no `bulk-archive-<TS>.json` file is written

### Requirement: Workflow-kind emit shape (extends the universal spine)

Every workflow-command run-summary JSON (per `Requirement: Emit scope`) SHALL include the universal spine defined in the `orbit-conventions` capability's `Internal-run JSON summary format` requirement — 6 required top-level fields:

```
command          string       identifies which command emitted (matches filename prefix)
timestamp        string       ISO-8601 UTC; JSON field uses standard colon format `YYYY-MM-DDTHH:MM:SSZ`. Filename embeds a colon-replaced `<TS>` token `YYYY-MM-DDTHH-MM-SSZ` (colons aren't filesystem-safe across all platforms).
change           string|null  the change name (or null for project-scope commands)
final_assessment string       narrative of what just happened (human-readable)
next_recommended string       verbatim recommendation, suitable for orbit-status best-effort parse
kind             enum         "workflow" | "editorial" | "lifecycle"  (always "workflow" for commands in this capability's scope)
```

For workflow-kind emits specifically, `kind` SHALL equal `"workflow"`. Per-command extensions (defined in subsequent requirements in this capability) add command-specific state beyond the spine.

Filenames SHALL use the per-command prefix (preserving entry-point provenance): `<command>-<TS>.json`. Propose-shaped variants (`new`, `continue`, `ff`) each use their own command-name prefix, NOT `propose-<TS>.json`.

#### Scenario: explore emit includes all 6 spine fields
- **WHEN** the emit-layer writes `explore-<TS>.json` for a named-mode explore
- **THEN** the JSON contains `command`, `timestamp`, `change`, `final_assessment`, `next_recommended`, and `kind` fields with valid values

#### Scenario: apply emits with kind=workflow
- **WHEN** the emit-layer writes `apply-<TS>.json`
- **THEN** the `kind` field equals `"workflow"`

#### Scenario: /opsx:new uses its own filename prefix
- **WHEN** the user invokes `/opsx:new foo` and the emit-layer writes a JSON
- **THEN** the filename is `new-<TS>.json`, NOT `propose-<TS>.json`

#### Scenario: orbit-status tier-1 reader parses next_recommended verbatim
- **WHEN** orbit-status reads any run-summary JSON and best-effort parses `next_recommended` per `orbit-status-recommendation/spec.md:7`
- **THEN** the leading `/opsx:<verb> [args]` token (if present) is extracted into `command` and `args`; on parse failure (e.g., prose recommendation), the full string is preserved in `reason`

### Requirement: Emit timing semantics

The emit-layer SHALL fire at well-defined moments per command type. Three categories cover the full command surface:

**One-shot commands** — emit ONCE on natural command completion (when the AI completes the command's intended work and is about to return control to the user):
- `/opsx:propose`, `/opsx:new`, `/opsx:ff`, `/opsx:continue`
- `/opsx:verify`
- `/opsx:audit-drift` standalone (excluding the inline-during-archive case captured in `archive-<TS>.json`)
- `/opsx:review-external` at T0 (when the prompt is packaged, before findings return)

**Multi-turn commands with explicit phases** — emit at phase boundaries per the command-specific requirement:
- `/opsx:apply` — emits per `Requirement: Apply per-chunk-end emission` (chunk completion is the primary trigger; mid-chunk pause is the secondary trigger per the conversation-boundary rules below)

**Multi-turn conversational commands** — emit at conversation boundaries:
- `/opsx:explore` (named mode) — emits per `Requirement: Named-mode explore recommendation by maturity` with the timing defined here

A **conversation boundary** for multi-turn conversational commands is defined as EITHER of (both rules are operational signals the emit-layer can detect at JSON-write time without requiring future-message inspection):

1. **User explicitly signals stop or pause** — "stop", "pause", "that's it for now", "we'll come back to this later", or equivalent intent
2. **AI-initiated wrap** — AI says "I've captured everything; type `/opsx:propose <name>` when ready" or equivalent intent (an explicit suggestion of the next workflow step, signalling the conversation is at a natural pause point)

**Canonical orbit UX convention**: to make most natural handoffs become explicit boundaries, when the AI hands control back to the user after a multi-turn conversational command session (such as `/opsx:explore` named-mode), the AI SHOULD include an explicit wrap phrase suggesting the next step (e.g., "type `/opsx:propose <name>` when ready" or "type `/opsx:explore <name>` to continue capturing"). This turns the handoff itself into a rule-2 signal and triggers the emit naturally without requiring the AI to predict future user behavior.

**Known limitation** (accepted tradeoff): if the user closes the conversation without either an explicit stop signal or an AI-initiated wrap (e.g., user simply walks away mid-session), no emit fires for that session. The change's state persists in on-disk artifacts (e.g., `explore.md`), but the `.orbit-runs/` JSON timeline misses the implicit boundary. Capturing the implicit case would require future-message inspection unavailable to the emit-layer at JSON-write time; explicit-signal-only is the operationally simpler tradeoff.

For `/opsx:apply` mid-chunk pause specifically, the same conversation-boundary rules apply — in addition to the explicit chunk-completion trigger.

**What is NOT a session boundary**: AI returning control mid-thought to ask a clarifying question; user asking a tangential question and returning to the same workflow; AI mid-turn pauses to read files or run commands; AI tool-use sequences within a single response. These are continuations, not boundaries — no emit fires.

#### Scenario: One-shot command emits exactly once
- **WHEN** `/opsx:propose foo` runs and completes generating all artifacts
- **THEN** exactly one `propose-<TS>.json` is written at completion; no additional emits

#### Scenario: Apply chunk completion is an explicit phase boundary
- **WHEN** `/opsx:apply foo` checks the last task in chunk N
- **THEN** the emit fires per `Requirement: Apply per-chunk-end emission` with `chunk_complete: true`

#### Scenario: Apply mid-chunk pause on explicit user signal
- **WHEN** during `/opsx:apply foo` the user says "stopping here for now"
- **THEN** the emit-layer treats this as a conversation boundary and emits `apply-<TS>.json` with `chunk_complete: false` per the per-chunk-end requirement's rule 2

#### Scenario: Named-mode explore emits when AI includes wrap phrase on handoff
- **WHEN** `/opsx:explore foo` has captured N decisions; AI returns control to user with an explicit wrap phrase such as `"type /opsx:propose foo when ready"`
- **THEN** the emit-layer treats this as a rule-2 (AI-initiated wrap) boundary and emits `explore-<TS>.json` with current decision and question counts and recommendation per maturity rules

#### Scenario: Continuation within a conversation is NOT a boundary
- **WHEN** during `/opsx:explore foo` the user asks a tangential question, the AI answers, and the user returns to the exploration topic in the same conversation
- **THEN** the emit-layer does NOT emit during the tangent or upon return; emit only fires when an actual conversation boundary (rule 1 explicit signal, or rule 2 AI-wrap) is reached

#### Scenario: User abandons conversation without explicit signal (known limitation)
- **WHEN** `/opsx:explore foo` has captured N decisions; AI returns control to user without an explicit wrap phrase; the user does not respond and the conversation effectively ends
- **THEN** no emit fires for that session. The `openspec/explore/foo/explore.md` artifact retains captured state. The next `/opsx:explore foo` invocation picks up from disk and will emit at its own boundary; the prior session's implicit boundary is not captured in the `.orbit-runs/` timeline

#### Scenario: AI-initiated wrap fires emit
- **WHEN** during `/opsx:explore foo` the AI says "I've captured everything; type `/opsx:propose foo` when ready"
- **THEN** the emit-layer treats this as a conversation boundary and emits `explore-<TS>.json` with current state

### Requirement: Emit-layer wraps upstream skills without modifying them

The emit-layer SHALL be implemented as a wrapper that runs after a command completes, inspects the command's output (artifact state, return codes, tasks state), and writes the JSON. Upstream skill bodies (the `# <skill-name>` markdown for upstream-authored skills like `openspec-verify-change`, `openspec-apply-change`, etc.) MUST NOT gain new behavior as part of this change.

Specifically:

- `/opsx:verify` SHALL NOT gain marker-dropping, spec-edit shortcuts, or other behaviors that change what verify itself does. The recommendation-classification logic for verify-fail modes (per `Requirement: Verify fail-mode recommendations`) lives in the emit-layer, not in verify.
- `/opsx:apply` SHALL NOT gain task-ordering changes, parallel-task execution, or other behaviors that change what apply itself does. The chunk-end emit logic (per `Requirement: Apply per-chunk-end emission`) lives in the emit-layer, not in apply.
- Where orbit modifies an upstream skill via the existing `## Orbit additions` pattern (currently used in `openspec-explore`, `openspec-propose`, `openspec-archive-change`), the emit-layer instructions MAY be added to those `## Orbit additions` sections; they MUST NOT modify the upstream-authored body above the additions marker.

#### Scenario: /opsx:verify upstream behavior unchanged
- **WHEN** an `openspec update` brings a new version of `openspec-verify-change` from upstream
- **THEN** the upstream skill body in `.claude/skills/openspec-verify-change/SKILL.md` is overwritten by the update, and orbit's emit-layer additions (if scoped to `## Orbit additions`) are preserved or re-added without conflict; the emit-layer never silently overrides upstream verify behavior

#### Scenario: emit-layer does not modify task-checking
- **WHEN** `/opsx:apply` runs and checks off tasks per upstream behavior
- **THEN** the task-checking algorithm is identical to upstream; only the side-effect of writing `apply-<TS>.json` is new

### Requirement: Bare-mode explore non-emission and crystallization warning

When `/opsx:explore` is invoked without a name argument (bare mode), the emit-layer SHALL NOT write a `.orbit-runs/` JSON. Exploration without a name is pre-commitment thinking; no persistent emit occurs until the user crystallizes the exploration to a named change.

When the user requests crystallization during a bare-mode explore (e.g., "give this exploration a name and save it"), the AI SHALL surface an explicit warning describing the persistence consequences before creating `openspec/explore/<name>/explore.md` and emitting the first `explore-<TS>.json`. The warning MUST mention:

1. A new directory `openspec/explore/<name>/` will be created with `explore.md` capturing decisions to date
2. A `.orbit-runs/explore-<TS>.json` will start the change's audit trail
3. The change will become visible to orbit-status (which scans `openspec/explore/` via its `enumerate_explorations()` function) and any other consumer that scans `openspec/explore/`. Note: upstream `openspec list` does NOT scan `openspec/explore/`; the change appears in `openspec list` only after `/opsx:propose` moves the staging directory to `openspec/changes/<name>/`.
4. Abandonment after this point requires formal archive/discard, not just deleting the directory

The AI MUST wait for explicit user confirmation before proceeding.

#### Scenario: Bare-mode explore produces no .orbit-runs JSON
- **WHEN** the user invokes `/opsx:explore` (no name) and converses for 30 minutes without crystallizing
- **THEN** no `.orbit-runs/` JSON is written; the conversation produces no on-disk persistence

#### Scenario: Crystallization request triggers warning
- **WHEN** during a bare-mode explore the user says "save this as `<name>`"
- **THEN** the AI surfaces a warning enumerating the 4 consequences above and waits for user confirmation before creating any file

#### Scenario: Confirmed crystallization writes first emit
- **WHEN** the user confirms the crystallization warning
- **THEN** the AI creates `openspec/explore/<name>/explore.md` and writes the first `explore-<TS>.json` capturing decisions to date

### Requirement: Named-mode explore recommendation by maturity

Named-mode `/opsx:explore` emits SHALL include `next_recommended` text that adapts to the count of decisions captured and the count of open questions remaining. Four mutually-exclusive maturity states cover the full grid:

- **Early (0–1 decisions captured)**: `"/opsx:explore <name> — continue capturing thinking"`
- **Mid (2–3 decisions captured)**: `"/opsx:explore <name> — continue thinking, or /opsx:propose <name> if ready to formalize"`
- **Mature (4+ decisions captured AND ≤1 open question)**: `"/opsx:propose <name> — substantial thinking captured; ready to formalize the design (or /opsx:explore <name> to keep refining)"`
- **Stalled (4+ decisions captured AND >1 open question)**: `"/opsx:explore <name> — N decisions captured but M open questions remain; resolve open questions before formalizing"` (N = `decisions_captured`, M = `open_questions_count`). NOTE: this state does NOT surface `/opsx:propose` as an alternative because high open-question count signals NOT-ready regardless of decision count; the user is directed back to exploration to close out the open questions first.

In each case the leading `/opsx:<verb> <name>` token is the canonical recommendation that orbit-status parses into `command`/`args`; the alternative path (when present, Mid and Mature only) lives in the reason text.

#### Scenario: 1-decision explore recommends continued exploration
- **WHEN** named-mode explore emits with `decisions_captured: 1` and `open_questions_count: 3`
- **THEN** `next_recommended` begins with `"/opsx:explore <name>"` and contains "continue capturing thinking"

#### Scenario: 5-decision explore with 0 open questions recommends propose
- **WHEN** named-mode explore emits with `decisions_captured: 5` and `open_questions_count: 0`
- **THEN** `next_recommended` begins with `"/opsx:propose <name>"` and surfaces `/opsx:explore` as alternative in reason

#### Scenario: 0-decision explore recommends continued exploration
- **WHEN** named-mode explore emits with `decisions_captured: 0`
- **THEN** `next_recommended` begins with `"/opsx:explore <name>"` and contains language acknowledging that thinking is just beginning (e.g., "continue capturing thinking — no decisions captured yet")

#### Scenario: Stalled — 5 decisions captured but 3 open questions remain (Mature criteria for decisions met but open-question check fails)
- **WHEN** named-mode explore emits with `decisions_captured: 5` and `open_questions_count: 3`
- **THEN** `next_recommended` begins with `"/opsx:explore <name>"` (NOT `/opsx:propose`); the text references both counts (e.g., "5 decisions captured but 3 open questions remain; resolve open questions before formalizing"); `/opsx:propose` is NOT surfaced as an alternative

### Requirement: Propose-shaped and artifact-completion-aware recommendation logic

The four propose-family commands SHALL emit their `next_recommended` per one of two recommendation classes based on what they actually produce:

**Propose-shaped** commands (`/opsx:propose`, `/opsx:ff`) produce the canonical artifact set (proposal.md + design.md + tasks.md + at least one specs/<capability>/spec.md) in a single invocation. They SHALL emit:

```
next_recommended: "/opsx:review <name> — proposal artifacts ready; review before apply"
```

(Per `openspec-orbit#9` — propose should recommend review, not apply.)

**Artifact-completion-aware** commands (`/opsx:new`, `/opsx:continue`) produce partial artifact state — `/opsx:new` scaffolds the change directory without generating artifacts; `/opsx:continue` generates the next missing artifact. Both SHALL emit a recommendation that depends on upstream `openspec status --change <name> --json`'s `isComplete` field (the same field upstream `openspec-continue-change` uses to gate completion). This keeps the emit-layer schema-agnostic — whatever upstream's authoritative completeness definition is, the emit recommendation follows it:

- **`isComplete: true`** (all required artifacts present): `next_recommended: "/opsx:review <name> — all proposal artifacts now present"`
- **`isComplete: false`** (more artifacts pending): `next_recommended: "/opsx:continue <name> — <next missing artifact> still pending"` (where `<next missing artifact>` is the first artifact in upstream's `applyRequires` array whose `status` is not `done`)

For `/opsx:new` specifically, `isComplete: false` is the typical case immediately after scaffolding (upstream `/opsx:new` doesn't auto-generate artifacts; the user is expected to run `/opsx:continue` next). For `/opsx:continue`, `isComplete: true` is the typical terminal state.

#### Scenario: /opsx:propose recommends /opsx:review (propose-shaped)
- **WHEN** `/opsx:propose foo` completes after generating proposal/design/tasks/specs
- **THEN** the `propose-<TS>.json`'s `next_recommended` begins with `"/opsx:review foo"`

#### Scenario: /opsx:new (scaffold-only) recommends /opsx:continue
- **WHEN** `/opsx:new foo` completes after upstream scaffolds the change directory (no artifacts generated); `openspec status --change foo --json` reports `isComplete: false`
- **THEN** the `new-<TS>.json`'s `next_recommended` begins with `"/opsx:continue foo"` (artifact-completion-aware class, NOT propose-shaped — even though `/opsx:new` is in the propose family, it doesn't produce the canonical artifact set in one go)

#### Scenario: /opsx:continue with missing tasks.md recommends /opsx:continue
- **WHEN** `/opsx:continue foo` completes after generating design.md but tasks.md is still missing; `openspec status --change foo --json` reports `isComplete: false` with `tasks` next in `applyRequires`
- **THEN** the `continue-<TS>.json`'s `next_recommended` begins with `"/opsx:continue foo"` and identifies `tasks.md` as the next missing artifact

#### Scenario: /opsx:continue with all artifacts present recommends /opsx:review
- **WHEN** `/opsx:continue foo` completes after the final missing artifact is generated; `openspec status --change foo --json` reports `isComplete: true`
- **THEN** the `continue-<TS>.json`'s `next_recommended` begins with `"/opsx:review foo"`

#### Scenario: Schema-agnostic — alternative schemas with different completeness criteria work via isComplete
- **WHEN** a hypothetical future change uses a non-spec-driven schema where completeness requires only proposal.md + tasks.md (no design.md, no specs/); upstream reports `isComplete: true` after both are generated
- **THEN** the emit-layer recommends `/opsx:review foo` based on `isComplete: true` alone, regardless of whether design.md or specs/ are present — the emit-layer does NOT hand-roll the completeness check

### Requirement: Apply per-chunk-end emission

`/opsx:apply` SHALL emit `apply-<TS>.json` at chunk boundaries when chunks are explicitly declared in the change's tasks.md preamble. The chunk preamble SHALL use this format: an HTML comment block at the top of tasks.md (before the first `## <group-name>` header) containing lines of the form `Chunk N (groups X[-Y]): <chunk-name>`. Example:

```
<!--
Implementation chunks:
  Chunk 1 (groups 1):    Foundation
  Chunk 2 (groups 2-3):  Workflow emits
  Chunk 3 (groups 4):    Apply behavior
-->
```

**Format constraints**:
- The `(groups X[-Y])` portion supports a single group (`X`) or a contiguous range (`X-Y`). Non-contiguous group sets (e.g., `(groups 6,8,9)`) are NOT supported in v1 — restructure tasks.md groups to be contiguous, or split into multiple chunks.
- **Malformed preamble handling** (graceful degradation): if the preamble comment block exists but cannot be parsed (missing parens, typo in `Chunk N`, malformed group range, etc.), the emit-layer SHALL log a single warning to stderr (`"warning: tasks.md preamble at <path> could not be parsed as chunk declarations; falling back to no-chunking mode"`) and proceed under no-chunking-apply rules (single emit at session end with `chunk: null`). The implementation is "tolerant on read, strict on write" — don't block the user's work; surface the parse issue so they can fix the preamble.

The emit follows these rules:

1. **Chunk completion**: when the last task in chunk N is checked, emit with `chunk: "N of M"`, `chunk_complete: true`, and `next_recommended` advancing to the next chunk (`/opsx:apply <name>`) or to `/opsx:verify <name>` on apply-complete (chunk N == M).
2. **Mid-chunk session pause**: when the user pauses or hands off mid-chunk-N (e.g., "stopping here for now"), emit with `chunk: "N of M"`, `chunk_complete: false`, and `next_recommended: "/opsx:apply <name>"` to resume.
3. **No-chunking apply**: when tasks.md has no explicit chunk preamble, emit once at session end with `chunk: null`, `chunk_complete: true`, and `next_recommended` advancing to `/opsx:verify <name>` on apply-complete or `/opsx:apply <name>` if tasks remain.

The apply emit per-command extensions SHALL include the following fields beyond the spine:

```
tasks_completed              int   running total of checked tasks across all chunks
tasks_remaining              int   running total of unchecked tasks (includes user-validation tasks if present)
user_validation_remaining    int   subset of tasks_remaining that are acknowledged user-validation tasks
                                   (acknowledged-convention work intentionally left unchecked at apply-time for runtime
                                   testing or user confirmation; surfaces as warnings at archive time, not blockers).
                                   Default: 0. Set to a positive count when the AI recognizes specific unchecked tasks
                                   as user-validation per their text (e.g., "Live-test ...", "(User-validation handoff)").
chunk                        string | null   e.g., "3 of 5" or null
chunk_name                   string | null   e.g., "phase+attention engine"
chunk_complete               bool             true when AI-doable work in this chunk is done (i.e.,
                                              tasks_remaining - user_validation_remaining == 0 for this chunk's tasks);
                                              user_validation_remaining > 0 does NOT block chunk_complete: true
tasks_completed_this_session int             delta since prior apply JSON
```

**Semantic rule**: `chunk_complete: true` means "AI-doable work in this chunk is done." `tasks_remaining > 0` AND `user_validation_remaining == tasks_remaining` is the expected state at the final chunk's emit when user-validation tasks exist — the AI is done; the remaining work is user-driven runtime testing. `tasks_remaining > 0` AND `user_validation_remaining < tasks_remaining` with `chunk_complete: true` is a contradiction the emit-layer MUST NOT produce.

#### Scenario: Chunk-1-of-5 completion emits with chunk_complete=true
- **WHEN** `/opsx:apply foo` completes the last task in chunk 1 of a 5-chunk apply
- **THEN** `apply-<TS>.json` is written with `chunk: "1 of 5"`, `chunk_complete: true`, and `next_recommended` advancing to `/opsx:apply foo` (next chunk)

#### Scenario: Mid-chunk pause emits with chunk_complete=false
- **WHEN** the user pauses mid-chunk-3 with 5 of 12 tasks in the chunk checked
- **THEN** `apply-<TS>.json` is written with `chunk: "3 of 5"`, `chunk_complete: false`, `tasks_completed_this_session: 5`, and `next_recommended: "/opsx:apply foo"`

#### Scenario: Apply-complete recommends verify
- **WHEN** `/opsx:apply foo` completes the last task in the final chunk
- **THEN** `apply-<TS>.json` is written with `chunk_complete: true` and `next_recommended` beginning with `"/opsx:verify foo"`

#### Scenario: No-chunking apply emits once at session end
- **WHEN** `/opsx:apply foo` runs on a change with no explicit chunk preamble in tasks.md
- **THEN** exactly one `apply-<TS>.json` is written at session end with `chunk: null` and `chunk_complete: true`

#### Scenario: Final chunk completes with user-validation tasks remaining
- **WHEN** `/opsx:apply foo` completes the last AI-doable task in the final chunk, but 6 user-validation tasks (e.g., "9.1 Live-test ...", "9.9 (User-validation handoff)") remain explicitly unchecked per the user-validation convention
- **THEN** `apply-<TS>.json` is written with `chunk_complete: true`, `tasks_remaining: 6`, `user_validation_remaining: 6` (matching `tasks_remaining`), and `next_recommended` begins with `"/opsx:verify foo"` — the AI is done; user-validation happens at verify time

### Requirement: Verify pass recommendation

When standalone `/opsx:verify <name>` passes (verify-change reports no failures), the emit-layer SHALL write `verify-<TS>.json` with `next_recommended`:

```
"/opsx:review --as system <name> — verification clean; formal pre-archive
 review recommended (or /opsx:archive <name> if you're skipping the
 editorial pass)"
```

The leading `/opsx:review --as system <name>` token is the canonical recommendation; orbit-status's tier-1 parse extracts this into `command`/`args`. The alternative `/opsx:archive <name>` path lives in the reason text and is surfaced to the user via orbit-status's `recommended_next.reason` field.

#### Scenario: Verify pass emit recommends review-as-system canonically
- **WHEN** `/opsx:verify foo` passes
- **THEN** `verify-<TS>.json`'s `next_recommended` begins with `"/opsx:review --as system foo"` and the reason text mentions `/opsx:archive foo` as alternative

#### Scenario: orbit-status parses verify-pass recommendation
- **WHEN** orbit-status reads the verify-pass JSON and best-effort parses
- **THEN** `command` is `"/opsx:review --as system"`, `args` is the change name, `reason` is the full verbatim string

### Requirement: Verify fail-mode recommendations

When standalone `/opsx:verify <name>` fails, the emit-layer SHALL classify the failure mode based on verify-change's output and write `next_recommended` accordingly:

- **Mode ① (tasks-incomplete)**: tasks.md has unchecked items → `"/opsx:apply <name> — N tasks remain unchecked; complete implementation before re-verifying"`
- **Mode ② (impl-vs-spec gap)**: tasks all checked but spec scenarios fail → `"/opsx:review --as system <name> — N spec scenarios fail against implementation; system review will surface findings in its scorecard for you to walk each as code fix or spec revision"`
- **Mode ③ (openspec-validate failure)**: `openspec validate` itself fails → `"Fix artifact validation errors: <validator message verbatim>"` (no leading orbit command; reason field carries the validator output)

When verify completes with warnings (passes but with non-blocking findings), the emit-layer SHALL write `next_recommended: "/opsx:review --as system <name> — verification passed with N warnings; system review recommended"`.

The classification SHALL happen in the emit-layer; verify itself does not gain marker-dropping or other behaviors (see `Requirement: Emit-layer wraps upstream skills without modifying them`). The mode-② recommendation does NOT use `/opsx:review --as system --mark` because that flag is proposal-mode only and silently ignored in system mode (per `orbit-review/spec.md`'s `Requirement: --mark flag is proposal-mode only`). The user manually walks each system-review finding to decide code-vs-spec.

**Precedence when multiple fail signals fire simultaneously** (highest precedence first):

1. **Mode ③ (openspec-validate failure)** — if `openspec validate` itself errors, verify cannot meaningfully evaluate impl-vs-spec until the spec is fixable. Mode ③'s recommendation wins over modes ① and ②.
2. **Mode ① (tasks-incomplete)** — if tasks.md has unchecked items, evaluation is premature regardless of any impl-vs-spec signal. Mode ①'s recommendation wins over mode ②.
3. **Mode ② (impl-vs-spec gap)** — applies when modes ③ and ① are clean.
4. **Warn state** — applies on pass with warnings only when none of the failure modes fire.

This precedence directs the user to the root cause first: fix the spec (mode ③) → complete the implementation (mode ①) → resolve impl-vs-spec gaps (mode ②).

**Out of scope**: partial or aborted verify runs (e.g., verify timed out before classifying findings, transient validator failure). These are upstream verify-change concerns; the emit-layer SHALL emit whatever verify-change reports. If verify-change reports incomplete output, the emit-layer SHALL set `next_recommended` to `"Re-run /opsx:verify <name> — prior verify run incomplete (see verify-change output for details)"`.

#### Scenario: Mode-① fail recommends /opsx:apply
- **WHEN** `/opsx:verify foo` fails because tasks.md has unchecked items
- **THEN** `verify-<TS>.json`'s `next_recommended` begins with `"/opsx:apply foo"` and mentions the count of remaining tasks

#### Scenario: Mode-② fail recommends /opsx:review --as system (without --mark)
- **WHEN** `/opsx:verify foo` fails because spec scenarios don't pass against the implementation
- **THEN** `verify-<TS>.json`'s `next_recommended` begins with `"/opsx:review --as system foo"` (without `--mark` — that flag is proposal-mode only); reason text explains the user must walk each system-review finding as either a code fix or spec revision

#### Scenario: Mode-③ fail preserves verbatim validator message
- **WHEN** `/opsx:verify foo` fails because `openspec validate` itself errors (e.g., malformed spec frontmatter)
- **THEN** `verify-<TS>.json`'s `next_recommended` text begins with `"Fix artifact validation errors:"` and includes the validator's verbatim error message; orbit-status's tier-1 parse finds no leading slash command and preserves the full text in `reason`

#### Scenario: Multiple fail signals — precedence ③ > ① > ②
- **WHEN** `/opsx:verify foo` reports BOTH unchecked tasks (mode ① signal) AND `openspec validate` errors (mode ③ signal) simultaneously
- **THEN** `verify-<TS>.json`'s `next_recommended` follows mode ③ (verbatim validator message), NOT mode ①; the user is directed to fix the spec validation error first, after which a re-run will surface the tasks-incomplete signal

### Requirement: Review-external T0 multi-step recommendation

`/opsx:review-external` SHALL emit `review-external-<TS>.json` at T0 (when the prompt is packaged, before findings return from the external AI). The emit includes per-command extensions:

```
mode                  "proposal" | "system"
prompt_path           string   path to the just-written external-prompt-<mode>-<TS>.md
target                string|null   the named external AI (e.g., "codex", "gpt-5"), or null
awaiting_findings     bool     always true at T0
```

`next_recommended` SHALL be multi-step prose describing the user action plus the follow-up command:

```
"Paste openspec/changes/<name>/.orbit-runs/external-prompt-<mode>-<TS>.md
 into the target AI, save the response as
 openspec/changes/<name>/.orbit-runs/external-<mode>-<TS>.md,
 then /opsx:address-reviews <name> --from-file <path>"
```

orbit-status's tier-1 best-effort parse SHALL find no leading `/opsx:<verb>` (because the string leads with "Paste"); `command` and `args` are left null, and the full verbatim string is surfaced in `reason`. This is the expected behavior — multi-step user actions are accurately represented as prose, not collapsed into a single command.

T1 (findings returned, `/opsx:address-reviews --from-file` invoked) is the existing `/opsx:address-reviews` emit and is unaffected by this requirement.

#### Scenario: Review-external T0 emit has awaiting_findings=true
- **WHEN** `/opsx:review-external foo --as proposal` runs and packages the prompt
- **THEN** `review-external-<TS>.json` is written with `mode: "proposal"`, `awaiting_findings: true`, `prompt_path` pointing at the just-written `external-prompt-proposal-<TS>.md`

#### Scenario: Review-external T0 recommendation is prose-only
- **WHEN** orbit-status reads the review-external T0 JSON and best-effort parses `next_recommended`
- **THEN** no leading slash command is found; `command` and `args` are null; the full multi-step instruction is preserved verbatim in `reason`

### Requirement: Audit-drift standalone recommendations

Standalone `/opsx:audit-drift` already emits `audit-drift-<TS>.json` today per `.claude/skills/openspec-audit-drift/references/run-summary-schema.md`. The emit SHALL include the universal spine from `orbit-conventions`'s `Internal-run JSON summary format` (with `kind: "editorial"`). The emit-layer recognizes TWO standalone invocation modes — **change-scoped** (`/opsx:audit-drift <name>`) and **project-wide** (`/opsx:audit-drift` with no argument) — each with distinct emit path, `change` value, and recommendation logic:

**Change-scoped standalone** (`/opsx:audit-drift <name>`):
- Emit path: `openspec/changes/<name>/.orbit-runs/audit-drift-<TS>.json`
- `change` field: the change name
- `next_recommended` on findings: `"/opsx:address-reviews <name> --from-file <this-json> — N drift(s) detected; resolve before next workflow step"`
- `next_recommended` on clean (zero findings): copy from the most recent prior `.orbit-runs/*.json` for the same change, excluding the just-written audit-drift JSON itself. `final_assessment` SHALL note "drift check clean; deferring to prior workflow state."

**Project-wide standalone** (`/opsx:audit-drift` with no argument):
- Emit path: `openspec/.orbit-runs/audit-drift-<TS>.json` (create the directory if it doesn't exist)
- `change` field: `null` (no change scope)
- `next_recommended` on findings: `"/opsx:address-reviews --from-file <this-json> — N drift(s) detected project-wide; resolve before next workflow step"` (no `<change-name>` arg; address-reviews handles project-wide markers from --from-file)
- `next_recommended` on clean (zero findings): `"No project-wide drifts detected. Run /opsx:audit-drift periodically to catch new drift."` (no prior-workflow defer because project-wide audit-drift has no per-change workflow narrative to defer to). `final_assessment` SHALL note "project-wide drift check clean."

(The `--from-file` flag in both modes becomes optional once openspec-orbit#10 lands and `/opsx:address-reviews` auto-discovers internal JSONs; the JSON-format support is also tracked at openspec-orbit#4.)

Per-command extensions for audit-drift SHALL include:

```
categories_run        object   { vocab_residue, lens_staleness, cross_doc_consistency, archive_coherence } — bools
findings_by_category  object   counts per category
findings_total        int
```

This requirement applies only to **standalone** `/opsx:audit-drift` invocations (both change-scoped and project-wide). Inline audit-drift during `/opsx:archive` is captured in `archive-<TS>.json` per the existing archive emit convention (unchanged).

#### Scenario: Change-scoped standalone audit-drift with findings recommends address-reviews
- **WHEN** `/opsx:audit-drift foo` runs and detects 3 drifts
- **THEN** `openspec/changes/foo/.orbit-runs/audit-drift-<TS>.json` is written with `change: "foo"`, `findings_total: 3`, and `next_recommended` beginning with `"/opsx:address-reviews foo --from-file"`

#### Scenario: Change-scoped clean audit-drift defers to prior workflow recommendation
- **WHEN** `/opsx:audit-drift foo` runs and detects 0 drifts, and the prior latest `.orbit-runs/*.json` for `foo` is `apply-2026-05-21T10-00-00Z.json` with `next_recommended: "/opsx:apply foo — chunk 3 of 5 done"`
- **THEN** the new `audit-drift-<TS>.json` has `findings_total: 0` and `next_recommended` equal to the verbatim string `"/opsx:apply foo — chunk 3 of 5 done"`, and `final_assessment` notes drift-check-clean + deferral

#### Scenario: Project-wide standalone audit-drift with findings
- **WHEN** `/opsx:audit-drift` runs with no change argument and detects 2 project-wide drifts
- **THEN** `openspec/.orbit-runs/audit-drift-<TS>.json` is written with `change: null`, `findings_total: 2`, and `next_recommended` beginning with `"/opsx:address-reviews --from-file"` (no `<change-name>` arg)

#### Scenario: Project-wide clean audit-drift (no prior workflow to defer to)
- **WHEN** `/opsx:audit-drift` runs with no change argument and detects 0 drifts
- **THEN** `openspec/.orbit-runs/audit-drift-<TS>.json` is written with `change: null`, `findings_total: 0`, and `next_recommended` reads `"No project-wide drifts detected. Run /opsx:audit-drift periodically to catch new drift."`

#### Scenario: Inline audit-drift during archive unchanged
- **WHEN** `/opsx:archive foo` runs and the inline audit-drift step produces findings
- **THEN** the findings are captured in `archive-<TS>.json` per existing archive emit behavior; no separate `audit-drift-<TS>.json` is written for the inline pass

### Requirement: No forward-look at other-issue future behavior changes

Emit text SHALL recommend canonical command names without anticipating behavior changes scheduled in other open issues. Specifically:

- When a future issue inverts a command's default behavior (e.g., openspec-orbit#20 will invert `/opsx:review --as system`'s default to external/fresh-context reviewer mode), the emit text SHALL recommend `/opsx:review --as system` without qualifiers like "external recommended" or "fresh-context preferred"
- When a future issue adds a flag or sub-behavior, the emit text SHALL NOT preview the new flag or behavior

When the future issue lands, every prior emit that recommended its command automatically benefits because users invoking the recommended command experience the new default behavior at that moment; no emit-time forward-references are needed.

#### Scenario: Verify-pass recommendation does not reference openspec-orbit#20
- **WHEN** the emit-layer writes a verify-pass JSON's `next_recommended`
- **THEN** the text contains `"/opsx:review --as system <name>"` and does NOT contain qualifiers about external review, fresh-context, or reviewer-mode defaults

#### Scenario: Emit text contains no other-issue references
- **WHEN** any run-summary JSON is inspected
- **THEN** `next_recommended` and `final_assessment` text contain no issue numbers (e.g., `#20`, `#15`) or references to future-issue behavior changes
