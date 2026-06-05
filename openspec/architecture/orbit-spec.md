# orbit-spec — macro-architecture plan

> Living macro-architecture document (the "option C" convention sketched in `next_steps.md`).
> Long-lived; **not** consumed by `/opsx:propose`. Each phase below eventually becomes its own
> `/opsx:explore` → `/opsx:propose` cycle that *references* this doc.

## Vision

Collapse orbit from a two-layer "upstream + overlay" arrangement into a single cohesive,
self-contained tool — **orbit-spec** — that:

1. **Vendors a `@fission-ai/openspec@1.3.1` snapshot** rather than depending on it being
   installed separately. The pegged CLI engine (`validate` / `archive` move / `list`) ships
   *inside* the repo as a committed snapshot.
2. **Absorbs the generated markdown surface.** Stop relying on upstream `openspec init` to lay
   down a base set of skills/commands that orbit then overlays. The repo *is* the complete,
   cohesive surface (upstream's base + orbit's editorial/judgment layer).
3. **Extracts mechanical logic out of SKILL.md into a deterministic helper layer (`orbit-lib`)**
   — surgically, not maximally. The goal is **context-window reclaim + AI-variance elimination**,
   not token-shaving for its own sake. The judgment/editorial prose stays in SKILL.md; that layer
   is orbit's actual product.
4. **Renames `openspec-orbit` → `orbit-spec`** to reflect the new identity.

## Status

**Planning.** No phases started. This doc is the seed; `next_steps.md` holds the detailed
extraction analysis that phases 2–3 will draw from.

## Phases

Decoupled deliberately: the **value** (context reclaim) lives in orbit-lib; the **churn**
(vendor + rename) lives elsewhere. orbit-lib does not depend on the fork. Suggested order leads
with cheap validation, defers one-way doors.

| # | Phase | Depends on | Why this order |
|---|---|---|---|
| 1 | **orbit-lib proof-of-concept** — one helper (`emit-summary`, schema enforcement, used everywhere) | — | Validate the helper architecture cheaply before committing to the pivot |
| 2 | **Extract the mechanical blobs** — the 7 candidates identified in `next_steps.md` (auto-discovery sort, JSON routing, 5-state convergence, stock-phrasings lookup, cascade IN/OUT classification, decision-fork detection, universal-spine emit) | 1 | The bulk of the context-reclaim win |
| 3 | **Vendor `@fission-ai/openspec@1.3.1` + absorb the markdown surface** → cohesive repo | — (orthogonal to 1–2) | Requires the fork-positioning reversal (see Decisions) |
| 4 | **SKILL.md compression pass** — post-extraction cleanup of now-redundant prose | 2 | Only worthwhile once helpers exist to point at |
| 5 | **Rename `openspec-orbit` → `orbit-spec`** | 3 | One-way door; do last so references aren't redone |

## Decisions

| ID | Decision | State | Notes |
|---|---|---|---|
| D1 | **Vendoring does NOT break pegging.** A committed 1.3.1 snapshot makes the peg concrete; no-auto-ingest is preserved. | Settled | Pegging discipline becomes "we own the snapshot" rather than "we pin the dep." |
| D2 | **The "NOT a fork" positioning must be consciously reversed.** CLAUDE.md + the `orbit_pegged_to_openspec_1_3_1` memory both say orbit is not a fork. Vendoring + merging *is* a fork in practice. | Open — needs explicit rewrite | Do not let this drift silently; rewrite identity docs as part of phase 3. |
| D3 | **Extraction is surgical, not maximal.** Prose-best work (judgment, decision frameworks, pushback discipline, narrative UX, the "why") stays in SKILL.md. Only zero-judgment mechanical logic extracts. | Settled | "As much as possible into binary" would gut orbit's differentiating editorial layer. |
| D4 | **Helper form: default to script, not compiled binary.** Engine is already Node; a node/python helper keeps install friction near zero. Compiled binary adds platform builds + trust + install step against orbit's "it's just markdown" superpower. | Leaning script — revisit | Only go compiled if a concrete reason emerges. |
| D5 | **Goal is context reclaim + variance elimination, not dollar savings.** The dollar math (~$10–15/yr) does not justify the work; the context-window + authoring-cost + schema-enforcement math does. | Settled | Keep this framing to avoid over-extracting. |

## Open questions

- **orbit-lib language**: Node (matches the engine) vs Python vs something else? (D4 leans script; language still open.)
- **Distribution of orbit-lib**: bundled in the repo and invoked by relative path? Installed to a known location? How do helpers get on PATH for the AI's Bash calls?
- **Schema-enforcement boundary**: does `emit-summary` *validate* (reject bad shape) or *construct* (own the shape entirely)? Construct is stronger but moves more out of prose.
- **Vendoring mechanics**: copy the built CLI? the source? How do future 1.3.x → upgrade decisions get made against a vendored snapshot (the upgrade-is-a-deliberate-change-proposal principle)?
- **Relationship to "mastermind" / macro-architecture-planning convention** (`next_steps.md`): is *this very doc* the proof that the `openspec/architecture/<topic>.md` convention works, and should that convention now be formalized as an orbit primitive?
- **Workflow-inflection-point-fixes bundle** (tabled in `next_steps.md`): does it survive the pivot, get re-scoped into the new architecture, or get dropped?

## Considered & out

- **Maximal binarization** ("bring as much of the skills into binary as possible") — rejected per D3. Inverts orbit's value proposition.
- **Compiled binary as the default helper form** — deferred per D4 in favor of scripts; not ruled out permanently.
- **Lead with fork + rename** — rejected; that's churn with no functional gain. Lead with the orbit-lib proof (phase 1), which is where the value is.

## References

- `next_steps.md` — detailed deterministic-helper extraction analysis (7 candidate blobs, growth
  trajectory, cost/benefit), the orbit-v2 seed, the macro-architecture-planning gap, and the
  tabled `workflow-inflection-point-fixes` bundle.
- `CLAUDE.md` — current identity statement ("NOT a fork") that D2 will revise.
- Memory: `orbit_pegged_to_openspec_1_3_1`, `orbit_supports_full_openspec_1_3_1`,
  `openspec_1_3_1_actual_install_surface`.
