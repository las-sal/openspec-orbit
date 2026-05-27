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

## Context for resuming

### Just-archived change

`address-reviews-defaults-and-decision-forks` (2026-05-27) — landed `#14 + #11 + #18`. Cascade-by-default + walk-mode default + decision-fork detection. The Option D reframe (lifecycle-invariant OUT list only; no file-extension discrimination) emerged mid-cycle and broadened the cascade default's general-purpose utility for non-orbit consumers (Swift/Python/etc). 21 findings resolved across 6 review cycles.

### Token-extraction analysis (this session)

7 mechanical-logic blobs identified across recent changes as candidates for deterministic-helper extraction (orbit-lib). Per-invocation savings ~2000 tokens; per change ~100-200k; per year ~3-5M (modest in $ terms, but **30-40% context window reclaim** is the real prize). Growth trajectory: openspec-address-reviews/SKILL.md grew +63% in 9 days; doubles every ~2-3 months at current rate. The longer extraction waits, the more prose has to be migrated.

The extractions found:
1. Auto-discovery + tie-break (from `address-reviews-auto-discovers-internal-json`)
2. JSON parser routing + content sniff (from `address-reviews-accepts-internal-json`)
3. 5-state convergence detection (from `harden-review-mode-recommendations`)
4. 14-row stock phrasings lookup (from `harden-review-mode-recommendations`)
5. Cascade IN/OUT classification (from `defaults-and-decision-forks` — just archived)
6. Decision-fork hybrid detection (from `defaults-and-decision-forks`)
7. JSON emit / universal-spine construction (universal across emit-producing SKILL.md files)
