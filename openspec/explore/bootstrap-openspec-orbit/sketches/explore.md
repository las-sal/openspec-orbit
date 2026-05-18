# Sketch: `/opsx:explore` modifications

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — preserves explore's "stance, not workflow" character). Modifications add capture affordances that activate during exploration; they do NOT change the base thinking-partner stance.

## What stays the same

Upstream's explore is the foundation:

- It's a stance, not a workflow. No fixed steps, no mandatory outputs.
- Thinking partner — follow the conversation wherever it goes.
- Read files and investigate freely; never implement.
- May create OpenSpec artifacts if asked.

All of this is preserved verbatim. Orbit's additions activate **on top of** the conversation as capture-worthy moments emerge.

## What changes

Three additions:

1. **Capture affordances** — when the conversation produces a capture-worthy moment, explore offers to write it to the right file. Offer, don't auto-capture.
2. **`explore.md` authoring** — explore proactively maintains a durable record of the exploration at `openspec/explore/<name>/explore.md` when the exploration is named.
3. **Three invocation modes** — handles bare invocation, named invocation, and the case where a bare invocation crystallizes into a named one midway.

## The three invocation modes

```
Mode A:  /opsx:explore
─────────────────────────────────────────────────
         Pure think mode. No file created.
         Same as upstream behavior.
         If a name emerges in conversation,
         transitions to Mode C.

Mode B:  /opsx:explore <name>
─────────────────────────────────────────────────
         Named exploration.
         If openspec/explore/<name>/ doesn't exist:
           Create it with stub explore.md (5 sections).
           Treat the conversation as authoring this file.
         If it exists:
           Read explore.md for context.
           Resume from prior state.

Mode C:  Bare invocation → crystallizes
─────────────────────────────────────────────────
         User starts in Mode A.
         After ~2-3 substantive decisions emerge,
         explore offers:
           "We have enough material to capture here —
           what should we call this exploration?"
         On accept + name: create explore.md, back-fill
         with what's been discussed, continue as Mode B.
         On decline: continue in Mode A.
```

## Capture affordances — five types

When explore detects capture-worthy content, it pauses (briefly) to offer a write. Each type has a trigger pattern and a target file. The offer reads like a single short sentence.

### 1. Conventions

**Trigger signals**: "we always do X", "X should be named Y", "X must follow Z format", "this is the rule for…", "let's standardize on…"

**Target file**: a topic-specific convention file at project root (e.g., `naming_convention.md`, `error_handling.md`, `testing_convention.md`). New file if no relevant one exists; append to existing if there's a match.

**Offer**: "This feels like a durable convention. Capture in `naming_convention.md`?" (file name inferred from topic).

**Why a separate file, not CLAUDE.md**: CLAUDE.md is for handoff orientation (terse, references topic files). Conventions can grow detailed; topic files keep them organized.

### 2. Perspectives

**Trigger signals**: "X calls our Y", "from X's point of view", "X is a client/consumer of…", "X interacts with us via…", "how does X see…"

**Target file**: `openspec/lenses/perspectives.md` (append a new entry).

**Offer**: "That sounds like a perspective we care about — '<X using Y>'. Capture in `openspec/lenses/perspectives.md`?"

**Entry shape** (matches the `openspec/lenses/` content conventions):

```markdown
## <Perspective name>

**Surfaces**: <surface IDs from openspec/specs/<capability>/>

<Description: what this caller does, what we want to validate from this lens>

**Typical call patterns**:
- <example 1>
- <example 2>
```

### 3. Critical paths

**Trigger signals**: "the typical user flow is…", "the most important journey is…", "users typically…", "the critical path is…", "X happens when…"

**Target file**: `openspec/lenses/critical-paths.md` (append a new entry).

**Offer**: "That sounds like a critical user flow. Add to `openspec/lenses/critical-paths.md`?"

**Entry shape**:

```markdown
## <Flow name>

<Description: what the user is doing, why it matters>

**Touchpoints**:
1. <step 1>
2. <step 2>
3. <step 3>

**Expected behavior**:
- <SLO / contract / invariant>
```

### 4. Decisions in this exploration

**Trigger signals**: explicit decisions in the conversation ("OK let's go with X", "yes, X over Y because…", "I agree on…", "locked").

**Target**: the `Decisions` section of the current exploration's `openspec/explore/<name>/explore.md` (only applies in Mode B/C — there's no file in pure Mode A).

**Offer**: Usually doesn't ask explicitly during fluid conversation; explore captures decisions proactively when they crystallize, with a brief "captured" acknowledgment in chat. User can also say "capture that" to force a write, or "don't capture that yet" to defer.

**Entry shape** (matches the existing `explore.md` decision convention):

```markdown
- **<Decision title>** (YYYY-MM-DD) — <decision + brief rationale + supersession note if applicable>
```

### 5. References

**Trigger signals**: the user mentions a file, transcript, repo, or external doc that's informing the thinking.

**Target**: `openspec/explore/<name>/explore.md` `References` section.

**Offer**: Usually inline ("Captured as reference") rather than an explicit prompt. User can say "this is a reference" to force.

## The `explore.md` file

Five sections, matches the convention we've been using throughout this exploration:

```markdown
# Exploration: <name>

> **Status**: exploring. Promoted to proposal/design/specs via `/opsx:propose` when decisions firm up.

## Premise

<What we're trying to do and why.>

## Decisions

- **<title>** (YYYY-MM-DD) — <decision + rationale>

## Open questions

- **<question>** — <context + what's needed to resolve>

## Considered & out

- **<rejected option>** (rejected YYYY-MM-DD) — <why>

## References

- <file path | url | transcript> — <relevance>
```

Lifecycle:

- **Mode B/C**: explore writes/updates the file as the conversation progresses. AI proactively moves resolved Open questions → Decisions, moves rejected ideas → Considered & out, accumulates References.
- **`/opsx:propose`** reads this file later: Premise → proposal.md, Decisions → spec deltas + design.md choices, Considered & out → design.md alternatives section. Then **moves** `openspec/explore/<name>/` to `openspec/changes/<name>/`; explore.md persists as historical record.

## When to offer (heuristics)

The offer cost is real — interruption mid-thinking. Three heuristics to keep offers calibrated:

1. **Err toward asking, especially early.** The first few exchanges in a new exploration are when conventions, perspectives, and paths first crystallize. Missing the capture there means re-deriving later.

2. **Don't double-offer.** If the user just declined a similar capture, don't ask again for the same shape within the conversation. Track recent declines.

3. **Group offers when natural.** If three convention statements come up in close succession, offer once: "Three conventions came up — should we capture them together in `naming_convention.md`?"

The crystallization heuristic for **Mode C** (bare → named): trigger the name prompt when:

- The conversation has produced **2+ substantive decisions** (not just exploratory thinking)
- Or the user references "earlier we decided…" (signals they want persistent context)
- Or the user explicitly says "let's start capturing" / "this should be a real exploration"

## What explore does NOT do

- **Doesn't auto-capture** without an offer. Even decisions are proactively captured *and acknowledged*, but the user can always say "don't capture that."
- **Doesn't implement** anything. Same guardrail as upstream.
- **Doesn't promote** to a change directory. That's `/opsx:propose`'s job.
- **Doesn't validate** captured content for correctness — captures user statements as-is. Validation happens later via review commands.
- **Doesn't gate the conversation** on captures. Explore stays a thinking partner; capture is an affordance, not a checkpoint.

## Composition with other commands

```
                 /opsx:explore [<name>]
                          │
                          │
       ┌──────────────────┼──────────────────┬─────────────────────┐
       │                  │                  │                     │
       ▼                  ▼                  ▼                     ▼
   <name>/             *_convention.md    openspec/             (no file —
   explore.md          (in project root)  lenses/*.md           pure think
                                                                  mode)
       │                  │                  │
       │                  │                  │
       └──────────────────┴──────────────────┘
                          │
                          ▼
                  /opsx:propose [<name>]
                          │
                          ▼
                  Reads <name>/explore.md.
                  Generates proposal.md, design.md, specs/, tasks.md.
                  Moves openspec/explore/<name>/ → openspec/changes/<name>/.
                          │
                          ▼
                  Standard openspec change flow continues.
```

The lenses files and convention files **persist across explorations** (they're project-level). The explore.md travels with its change when promoted.

## Open design questions

1. **Convention file location**. Project root (alongside `CLAUDE.md`) or in a subdir like `openspec/conventions/`? Lean: project root, since `naming_convention.md` is referenced from `CLAUDE.md` and lives alongside it (matches the user's home-control precedent).
2. **Convention file naming**. `<topic>_convention.md` is the user's existing pattern. Stable; don't change.
3. **Crystallization signals tuning**. The "2+ substantive decisions" heuristic for Mode C is a guess; will need to calibrate from real use. Worth tracking after v1 ships.
4. **Group-offer detection**. The "three conventions came up — capture together?" pattern requires the AI to recognize multiple capture-worthy moments before offering once. Worth implementing but may not nail it in v1.
5. **Decline-tracking scope**. "Don't double-offer within this conversation" — does decline persist across sessions? Probably not (each conversation is fresh); but a recently-declined capture might warrant a "still no?" prompt the next time. Likely overkill for v1.

## Parallels with other orbit commands

Explore's modifications are different in shape from the review / audit / address commands — explore is a stance, the others are operations. But the orbit-coherence principles apply:

| | review / audit / address | explore (modified) |
|---|---|---|
| Operation | scan + report / act + log | conversational think mode |
| Output | structured report | conversation + offered captures |
| User control | invoke when ready | flow naturally; user accepts/declines captures |
| Coherence with openspec | adopts upstream conventions | preserves upstream stance entirely; adds affordances |

The shared orbit-coherence properties: pushback discipline (never assume; offer and confirm), guiding principle 2 (capture early, save downstream cost), and structural file conventions (lenses/, explore.md, convention files).

## What this means for the SKILL.md modifications

The upstream `openspec-explore` SKILL.md describes the stance and the "may create artifacts if asked" capability. Orbit's modifications extend this with:

- A "capture affordances" section describing the five capture types
- Trigger pattern hints (paraphrased above)
- The `explore.md` five-section convention
- The three invocation modes (A/B/C) with the Mode C crystallization heuristic
- Explicit "don't gate the conversation" guardrails

Orbit ships these as additions appended to the upstream SKILL.md content, marked as orbit additions. The base stance text is left intact (per guiding principle 1).

Same shape for the slash-command body (`.claude/commands/opsx/explore.md`) — append orbit-specific behavior.
