# Next steps

## Tabled

### `workflow-inflection-point-fixes` bundle (#15 + #7 + #16)

The explore staging at `openspec/explore/workflow-inflection-point-fixes/` is preserved. Scope was locked at Option B' (umbrella convention + 2 worked examples: post-review + post-propose). 4-5 capability deltas; ~35 scenarios; ~6 SKILL+command files. Tabled pending the macro architecture pivot — the bundle might be re-scoped, dropped, or land within the new architecture.

The 3 issues remain open in GitHub.

When resuming:
- Decide whether the bundle still makes sense post-pivot (the menu pattern likely still does; just might live in different files)
- Either `rm -rf openspec/explore/workflow-inflection-point-fixes/` to drop, or pick it back up

## Macro architecture planning — the gap

orbit's existing primitives are tuned for **change-scale**:

| Primitive | Scale | Lifecycle |
|---|---|---|
| `/opsx:explore` named | 1 upcoming change | explore.md → consumed by propose |
| `/opsx:propose` → `/opsx:apply` → `/opsx:archive` | 1 change cycle | days/weeks |
| `openspec/lenses/` | judgment captures | grows over time |

What's **missing**: the macro-architectural layer that spans many changes — multi-month vision + phased plan + ADR-style decision log.

### Three options for filling the gap

**(A) Claude plan mode + commit to markdown**

- Gives: one-session focused brainstorming, structured output
- Misses: doesn't persist across sessions naturally; no audit trail; no orbit-style discoverability
- Good for: "let me think through phase 1 in one session" — but not the whole multi-month thing

**(B) Repurpose `/opsx:explore` (named) for living macro-docs**

- Gives: orbit's existing tooling; markdown lives in `openspec/explore/<name>/explore.md`; multi-session capture; familiar 5-section convention works at any scale
- Misses: explore.md is implicitly "feeds into one propose" — using it for multi-month vision bends the convention. Users opening it might expect to consume it
- Good for: lightest touch; works today; no new orbit tooling

**(C) New orbit convention — `openspec/architecture/<topic>.md`**

- Gives: explicit signal that this isn't a single change. Lives outside `openspec/changes/` so doesn't confuse `openspec list`. Different section convention (Vision / Phases / Decisions / Open questions / References / Status). Each "phase" becomes a normal `/opsx:explore` → `/opsx:propose` cycle that references the architecture doc
- Misses: new orbit primitive to spec — overhead
- Good for: when you know you'll have multiple of these long-lived macro plans

### Recommendation

**Combine (A) + (C)**:

- **Plan mode for sessions** when you want focused brainstorming on a piece of the macro plan ("today I want to think through phase 1 dependencies")
- **`openspec/architecture/orbit-v2.md`** (or similar) as the **living macro document** — persists across sessions, version-controlled, structured for north-star planning

The new convention is small enough you could spec it as a single change (`add-architecture-planning-convention`) that adds:
- A new `orbit-conventions` requirement: macro-planning artifact lives at `openspec/architecture/<topic>.md`
- 7-section template: Vision / Status / Phases / Decisions / Open questions / Considered & out / References
- Lifecycle rules: long-lived; not consumed by propose; phases become regular changes that reference the doc

### Seed for the orbit-v2 / orbit-spec pivot

```
openspec/architecture/orbit-v2.md
├─ Vision: orbit subsumes @fission-ai/openspec@1.3.1 (fork);
│          rename to orbit-spec; deterministic-helper layer
│          (orbit-lib); context-window-optimized SKILL.md
├─ Status: planning
├─ Phases:
│   1. Bootstrap orbit-lib (first extraction: emit-summary helper)
│   2. Extract 5 mechanical-logic blobs from SKILL.md
│   3. Fork @fission-ai/openspec@1.3.1 → orbit-spec
│   4. Rename + distribute
│   5. SKILL.md compression pass (post-extraction cleanup)
├─ Decisions: language pick, distribution, breaking-change posture
├─ Open questions
├─ Considered & out
└─ References
```

Each phase eventually becomes its own `/opsx:explore` → `/opsx:propose` cycle that **references** the architecture doc.

### Pragmatic next step

**Minimum effort**: just create `openspec/architecture/orbit-v2.md` as a regular markdown file (no orbit tooling needed) and start filling in Vision + initial phases. Then when you come back, you've got the seed.

**Formalize first**: file an issue (`Add macro-architecture-planning convention`) so the next dev session (yours or AI's) treats it as a proper orbit primitive.

## "Mastermind" mode — uber-explore for multi-proposal efforts

Idea: a riff on a previously-suggested "mastermind" mode. An **uber-explore** that produces an architecture / large effort which then **chunks out into a set of core efforts**, each of which becomes its own proposal explored and pushed separately.

Early shape:
- `architecture.md` as the parent artifact (parent of `design.md`s) — describes the problem and decomposes it into a set of core efforts
- Each core effort = one candidate proposal that can be explored + pushed independently
- Architecture doc persists; child proposals reference it; phases land asynchronously

Open shape questions to revisit:
- Relation to the **macro-architecture-planning convention** sketched above (`openspec/architecture/<topic>.md`) — are these the same thing under two names, or is mastermind the *interactive mode* and architecture.md the *artifact*?
- Does mastermind get its own slash command (`/opsx:mastermind`?) or is it a flag on `/opsx:explore` (e.g. `--scope=architecture`)?
- Decomposition primitive: explicit "phase" entries in `architecture.md`, or a separate `phases/` directory, or just a section list that humans + AI both treat as the source of truth?
- How does a child proposal *reference* its parent architecture — frontmatter link, prose link, registry entry?
- Lifecycle: does the architecture doc get archived when all child phases land, or stay long-lived as a record?

Likely overlaps with the macro-architecture section above — worth merging the framings when this gets picked up.

**Worked exemplar (2026-06-05) — use as training/input when building this.** The fork-and-orbify planning session is a near-perfect specimen of the gap this mode fills: the work wasn't a *change*, it was establishing the strategic frame, decomposing a multi-workstream effort into a phased program, and identifying decision-forcing-functions *before any proposal existed*. `/opsx:explore` was the closest tool but it thinks about *one* problem — nothing owned the cross-change arc (the 6-change DAG, the "parity milestone", which decisions are forcing-functions). That cross-change ownership is exactly the mastermind/architect layer.
- Parent artifact produced: `openspec/architecture/openspec-1.3.1-architecture.md` (the grounding deep-dive) + `openspec/explore/fork-and-orbify/explore.md` (the exploration).
- The decomposition that session produced (1 foundation + 3 cohesion + 2 capability changes, organized by a "parity milestone") is itself the kind of output `/opsx:architect` should generate — chunking an architecture into a dependency-ordered set of child proposals. Mine this session's chat transcript for the interaction shape when picking this up.

## Deterministic-helper extraction (orbit-lib)

This is **Phase 1-2 of the orbit-v2 pivot** (per the seed above), but worth capturing standalone — it's a substantive architectural question independent of the rest of the pivot.

### The tension

orbit's current model: SKILL.md is everything. The AI is the runtime — reading prose + executing via tool calls. No deterministic code separate from upstream's `openspec` CLI.

Empirically observable costs:

| Cost | Evidence |
|---|---|
| **Token compounding** | openspec-address-reviews SKILL.md grew from ~270 → ~441 lines in 9 days; openspec-review is ~465 lines. Loaded on every invocation. |
| **AI variance** | Complex deterministic logic interpreted differently across model versions / sessions. The 5-state convergence model is ~50 lines of prose for ~30 lines of equivalent code. |
| **Spec maintenance** | Schema-shape rules duplicated across many SKILL.md files (universal-spine guidance, marker discovery patterns, JSON shapes). Changes require fan-out edits. |
| **Validation gaps** | `openspec validate` covers spec-side; nothing validates orbit-emit JSON shape. AI can omit fields or get them wrong. |

### Proposed architectural model

```
                              ┌────────────────────────┐
                              │  SKILL.md (prose)      │
                              │  — orchestration       │
                              │  — judgment calls      │
                              │  — narrative UX        │
                              │  — decision framework  │
                              └─────────┬──────────────┘
                                        │ calls
                                        ↓
       ┌──────────────────────────────────────────────────────┐
       │   .claude/orbit-lib/  (deterministic helpers)        │
       │                                                       │
       │   • emit-summary     — JSON schema enforcement       │
       │   • auto-discover    — recency rules + tie-break     │
       │   • convergence      — 5-state detection             │
       │   • compute-menu     — inflection-point recommendation│
       │   • discover-markers — grep patterns + exclusions    │
       │   • spec-diff        — delta vs baseline diff        │
       └──────────────────────────────────────────────────────┘
                              ↑ calls
                              │
                  ┌───────────┴────────────┐
                  │  upstream openspec CLI │
                  │  — already deterministic│
                  └────────────────────────┘
```

**Prose-best work** stays in SKILL.md: judgment calls, narrative explanations, decision frameworks, user-facing UX choices, push-back discipline, the "why."

**Code-best work** extracts to helpers: schema construction/validation, sorting/recency, hash-compare, JSON shape enforcement, pattern matching, file ops, anything mechanical with zero judgment.

### Triggers for extraction

When does logic move from prose to helper?

1. **Same algorithm duplicated across 3+ SKILL.md files** (e.g., universal-spine guidance is in every emit-producing SKILL.md)
2. **>50 lines of prose with zero judgment calls** (pure computation that a deterministic function does in <30 lines of code)
3. **AI-variance bug surfaced in review** (one cycle's AI got the sort wrong; another got it right — that's a signal)
4. **Frequent invocation** (loaded on every command call → tokens compound)
5. **Schema enforcement** (anything where AI omission/wrong-field shape is a real risk)

### Candidate themes (7 identified across the last 4 archived changes)

| Blob | Source change | Current prose | Post-extract | Tokens saved/load |
|---|---|---|---|---|
| Auto-discovery + tie-break | `address-reviews-auto-discovers-internal-json` | ~30 lines | ~3 lines | ~540 |
| JSON parser routing + content sniff | `address-reviews-accepts-internal-json` | ~35 lines | ~5 lines | ~600 |
| 5-state convergence detection | `harden-review-mode-recommendations` | ~60 lines | ~5 lines | ~1100 |
| 14-row stock phrasings lookup | `harden-review-mode-recommendations` | ~25 lines | helper-resolved | ~500 |
| Cascade IN/OUT classification | `defaults-and-decision-forks` (just archived) | ~40 lines | ~5 lines | ~700 |
| Decision-fork hybrid detection | `defaults-and-decision-forks` | ~50 lines | ~5 lines | ~900 |
| JSON emit / universal-spine construction | every emit-producing change (cumulative) | ~30 lines × 5 SKILL.md | helper-resolved | ~2700 |

Per-invocation savings: **~2000 tokens** reclaimed on average.

### Growth trajectory

| SKILL.md | Bootstrap (2026-05-18) | Today | Growth | Δ over ~9 days |
|---|---|---|---|---|
| openspec-address-reviews | 270 lines | 441 | +63% | +171 lines |
| openspec-review | 342 lines | 465 | +36% | +123 lines |
| openspec-audit-drift | ~250 (est) | 328 | +31% | +78 lines |

At current pace, the largest SKILL.md files **double every ~2-3 months**. Extraction NOW saves more than extracting later — the costs compound.

### Cycle-level projection (post-extraction)

| Metric | Current | Post-extract | Δ |
|---|---|---|---|
| Per cycle (~5-10 invocations) | ~30-60k tokens of SKILL.md load | ~20-40k | ~10-20k |
| Per change (~10 cycles for substantial changes) | ~300-600k | ~200-400k | ~100-200k |
| Per year (~50 archived changes) | ~10-15M | ~7-10M | ~3-5M |

### Cost vs. benefit

**Dollar cost**: ~$10-15/year per active orbit user reclaimed at Anthropic input pricing. **Small in absolute terms.**

**Real wins** (qualitative, harder to quantify but bigger):

1. **Context window reclaim**: ~30-40% reduction in SKILL.md tokens per invocation = roughly equivalent percentage of working memory freed for user-context, code-reads, conversation history. For long sessions (the just-archived change ran 6 cycles across one session — context pressure was real), this is the actual prize.

2. **AI variance elimination on mechanical logic**: subagent reviews currently catch some variance (Step 3d→3e cross-refs); but a different subagent might miss/catch differently. Deterministic helpers eliminate that variance for sort/classification/parsing logic.

3. **Authoring cost**: change a sort rule in code once vs update ~3 SKILL.md files + risk one slip-through (exactly what happened with the lifecycle reorder — Codex caught 3 sites missed in apply).

4. **Schema enforcement at emit time**: the resolution-log JSON shape went through 4 review cycles + had schema-doc drift surface as a pre-existing issue. A `orbit-lib emit-summary` helper that validates shape at write time would catch field omissions / type mismatches before they reach the JSON.

### Realistic extraction cost

- Bootstrap orbit-lib (language pick, distribution, first helper): **~1-2 change cycles**
- Each subsequent extraction (one of the 7 blobs above): **~0.5-1 change cycle**
- Top 5 extractions: **~5 cycles total** one-time investment

### When to file the issue

Natural trigger: **after the `workflow-inflection-point-fixes` bundle lands** (would add another ~50-100 lines of menu-heuristic prose — making the variance + duplication case sharper). Or earlier if appetite for the orbit-v2 pivot crystallizes first.

The dollar math alone doesn't justify it. The context-window + variance + authoring-cost math does — and grows sharper every change cycle.

## Context for resuming

### Just-archived change

`address-reviews-defaults-and-decision-forks` (2026-05-27) — landed `#14 + #11 + #18`. Cascade-by-default + walk-mode default + decision-fork detection. The Option D reframe (lifecycle-invariant OUT list only; no file-extension discrimination) emerged mid-cycle and broadened the cascade default's general-purpose utility for non-orbit consumers (Swift/Python/etc). 21 findings resolved across 6 review cycles.
