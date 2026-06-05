# Explore: fork-and-orbify

## Premise

Collapse orbit's **overlay model** into a single cohesive **fork of openspec@1.3.1**, integrating
orbit's content directly into the source instead of working around upstream. Motivation: the overlay
is structurally awkward — orbit's `.claude/skills/*/SKILL.md` and `.claude/commands/opsx/*.md` are
*generated output* of openspec's template→generation→adapter pipeline, hand-maintained in the output
position and divorced from the generator that's supposed to own them. Upstream `init`/`update` fights
the hand-edits (version-stamped `generatedBy` drift), and orbit's 4 net-new workflows can't
participate in adapters, `--tools`, profiles, validation, or drift detection.

Grounding doc: `openspec/architecture/openspec-1.3.1-architecture.md` (verified read of the v1.3.1
source). The exploration surfaced that "fork-and-orbify" is **two distinct workstreams with very
different risk profiles**, not one:

```
                        FORK & ORBIFY
                              │
            ┌─────────────────┴──────────────────┐
            ▼                                     ▼
   (1) ENGINE workstream              (2) GENERATOR + CONTENT workstream
   "grow the engine"                  "relocate content, modestly grow generator"
   genuinely new architecture          mostly relocation + 2 small generator gaps
            │                                     │
   promote review → graph gate         move prose overlay → source TS modules
   non-monotonic staleness model       multi-file skills (references/)
   generalize apply → N gates          arbitrary frontmatter metadata
                                       markdown-as-source vs inline-TS decision
```

The deeper thesis tying both together: **move orbit's sequencing/gating intelligence out of fragile
prose (where the model must *remember* to review-before-archive) and into the deterministic engine
(where it's enforced).** Today orbit enforces quality gates only by telling the model to, in prose,
across SKILL.md bodies. The engine already has deterministic sequencing (`graph.getNextArtifacts` /
`getBlocked` / `isComplete`) — orbit's phases just aren't in it.

---

## Decisions

> Leaning/working decisions from the exploration — not yet ratified into a proposal. Several remain
> coupled to the open questions below.

### D1 — Two workstreams, sequence engine-light first

Treat ENGINE (1) and GENERATOR+CONTENT (2) as separable. They share no code: (1) lives in
`src/core/artifact-graph/` + schema YAML; (2) lives in `src/core/templates/` +
`command-generation/` + `shared/skill-generation.ts`. Lowest-risk first move is a **content
relocation spike on ONE workflow** (e.g. `explore`): fork, rip telemetry, green build, migrate that
single workflow overlay→template, confirm `openspec init --tools claude` regenerates orbit's exact
content. Proves the whole loop before touching the engine or the other 14 workflows.

### D2 — The engine win is enforced gating, not "review as artifact"

The point of putting review in the graph is **not** that review becomes a document. It's that
`openspec status` / `instructions <next>` route the agent into review and **block archive until
review passes**, with zero prose reminders. Prompts go back to being guidance, not the enforcement
mechanism.

### D3 — The core new concept is non-monotonic completion (staleness/freshness)

Upstream completion (`state.ts detectCompleted`) is **file existence = monotonic**: once
`proposal.md` exists it's done forever. Fine for authored documents. But review/verify/audit are
**non-monotonic**: review at state A, edit specs at state B → review is now *stale* (un-passes).
File-existence can't express that. A `.orbit-runs/review-summary-*.json` existing proves a review
*happened*, not that it's *current* or *passed*.

→ The engine needs a **gate** concept: "passes relative to a snapshot of its inputs, invalidated when
they change." This is the one thing that **cannot** be built as an overlay — it's the heart of the
engine case. Leaning toward the **principled** model (first-class gate node) over the **pragmatic**
one (model review as a file-artifact + bolt staleness onto validation), precisely because staleness
is the whole prize and a bolt-on makes `status` lie.

### D4 — Generalize the singular `apply` phase into N gates

Don't invent the "phase that isn't a document" concept — upstream already has exactly one: the
`apply` block (`ApplyPhaseSchema` in `artifact-graph/types.ts`), a top-level non-artifact with
`requires` / `tracks: tasks.md` / `instruction` and a real state machine (`blocked|ready|all_done`)
computed in `instructions.ts`. The engine move = **generalize that singular block into a list of
gates**, each with `requires`, a pass-condition, an `instruction`, and the new **freshness rule**.
`apply`, `review`, `verify`, and an archive gate all become instances of one concept.

### D5 — Reclassify the 4 orbit workflows; only `review` is a new gate

```
review          → a re-entrant GATE (review → fix → review), pass/fail + staleness   ← the one new node
address-reviews → review's REMEDIATION LOOP (its "not-clean" state, like apply→tasks)  ← not its own node
review-external → an ALTERNATE EXECUTOR of the review gate (hand prompt to another AI)  ← a delivery flag
audit-drift     → a PROJECT-WIDE check, not change-scoped                               ← global gate or stays orthogonal
```

This collapses the engine scope a lot: of the four, only `review` is a genuine new graph phase.

### D6 — markdown-as-source over inline-TS-literals (content workstream)

Upstream inlines all prose as TS template literals (`explore.ts` ≈ 470 lines, mostly one backtick
string with escaped ASCII diagrams). That fights orbit's grain: orbit's entire editorial-review
discipline (`/opsx:review`, `@review:` markers, prose diffing) operates **on markdown**. Trapping
prose in `.ts` literals regresses authoring/diff/review and forces backtick-escaping in content full
of code fences.

→ Lean: TS workflow module becomes a **thin shell** that reads `content/<wf>/SKILL.md` +
`content/<wf>/references/*.md` from the *source* tree and returns the template. Generator stays
upstream-compatible; orbit authors keep editing markdown; and the references gap (D7) dissolves
because references are just sibling files in the content dir.

### D7 — Grow the generator: references + metadata pass-through (two verified gaps)

Moving content to source is *mostly* relocation, but the generator must grow to host orbit's richer
skill shape. Two gaps verified against source:

- **Multi-file skills.** 5 orbit skills bundle `references/*.md` (address-reviews ×3, review-external
  ×2, review/audit-drift/archive-change ×1). Upstream `SkillTemplate` is `{ …, instructions: string }`
  — one blob, no sibling-file concept — and `init`/`update` write exactly one `SKILL.md` per dir. →
  add a `references` field to `SkillTemplate` + teach the skill-writer in `init.ts`/`update.ts` to
  emit them. (Plausibly something upstream wants back.)
- **Frontmatter metadata.** `generateSkillContent()` hardcodes exactly three emitted metadata keys
  (`author`, `version`, `generatedBy`) and passes nothing else through. Orbit uses
  `metadata.capability: orbit-review` → currently **silently dropped**. → generalize metadata emission.

### D8 — Single-source kills a drift class; conventions become DRY-at-source

Today orbit hand-maintains a `SKILL.md` *and* an `opsx/*.md` per workflow — they can diverge
(`audit-drift` literally polices this). From a single template module both derive from one source →
that whole drift class disappears. And CLAUDE.md's "intentional duplication for reliability" of
cross-cutting conventions flips from *hand-duplicated in every file* to **DRY-at-source /
duplicated-at-output** via injection (the existing `transformInstructions` hook or a shared snippet
at generation time) — same reliability, no maintenance tax.

### D9 — Phasing: 6 changes across 3 arcs, divided by a parity milestone (RATIFIED)

The effort decomposes into **6 changes** organized as **1 foundation + 3 cohesion + 2 capability**.
The structural driver is a **parity milestone**: a point where the fork has *full behavioral parity
with today's overlay* but is now cohesive (content at source, generated by the pipeline). Everything
before it is *relocation* (mechanical, testable risk); everything after is *net-new capability*
(novel, design-heavy risk). Parity falls **after B3** — the natural "safe to pause / ship
incrementally" line.

```
   ARC A: FOUNDATION        ARC B: COHESION (→ parity)         ARC C: NEW CAPABILITY
   ┌──────────────┐    ┌──────────────────────────────┐   ┌────────────────────────┐
   │ A1 fork      │───▶│ B1 relocate │ B2 grow │ B3   │──▶│ C1 engine gates +      │
   │   bootstrap  │    │   content   │ generator│ net- │   │   freshness (review)   │
   │ tool-breadth │    │  (11 wf +   │ (refs +  │ new  │   │ C2 archive merge fix   │
   │ + telemetry  │    │   md-source)│ metadata)│ wf×4 │   │   (independent)        │
   └──────────────┘    └──────────────────────────────┘   └────────────────────────┘
                          ════════ PARITY (after B3) ════════
```

| # | Arc | Change | Depends on |
|---|---|---|---|
| **A1** | Foundation | Fork @1.3.1, rip/gate telemetry (Q6), **decide tool breadth (Q1)**, green build+test | — |
| **B1** | Cohesion | Relocate the 11 modified-upstream workflows overlay→source; establish **markdown-as-source** pattern (D6); conventions delivery (D8/Q9) | A1 |
| **B2** | Cohesion | Grow generator: `references[]` + metadata pass-through (D7) | A1 |
| **B3** | Cohesion | Add 4 net-new workflows first-class (review, review-external, address-reviews, audit-drift) → **parity** | B1, B2 |
| **C1** | Capability | Engine: generalize `apply`→N gates + freshness (Q3) + wire `review` as a gate (D3/D4/D5) | B3 |
| **C2** | Capability | Fix archive parallel-merge bug (Q8) — independent, can land any time after A1 | A1 |

**Granularity is ratified at 6.** Decided NOT to go coarser (B-arc collapse blows past orbit's
empirical ~3-capability/13-file max bundle). **C1 is the change most likely to fragment** — its three
sub-risks (schema/types generalization · freshness mechanism · wire review+status/instructions) are
genuinely different — but we **do not pre-split it**: let the C1 proposal split itself once Q3 is
pinned. B2+B3 *could* merge to 5 (grow-and-immediately-use) but kept separate for a cleaner test/
review surface.

### D10 — New repo, 1.3.1-as-commit-1, hybrid history model (RATIFIED; resolves Q10)

Orbify lands in a **fresh repository** ("it's a new thing"), not an in-place restructure of
openspec-orbit. This dissolves the A1 chicken-and-egg (Q10): the new repo is **born as the fork** —
day one it's pristine upstream openspec, and orbit arrives as ordinary commits on top.

Mechanics:
- **Commit 1 = pristine upstream `v1.3.1` tree** (verbatim; source at `/tmp/openspec-src`), **tagged
  `upstream/v1.3.1`**. Permanent verifiable baseline → `git diff upstream/v1.3.1` always yields
  orbit's *total* delta from upstream.
- **Hybrid history model** (also resolves Q7): **detached/fresh primary history** (clean ownership, no
  upstream-history baggage) **+ upstream registered as a read-only remote** so specific upstream
  commits can be **cherry-picked on deliberate decision** (e.g. an eventual archive-merge fix, Q8).
  Matches orbit's pegged / no-auto-ingest / surgical-upgrade stance. NOT a GitHub fork-with-history
  (that optimizes for frequent syncing orbit won't do).

Consequences/sequencing:
- **"Bringing orbit in" = arcs B, not a copy.** A1 is only *container + 1.3.1 baseline + telemetry
  (Q6) + tool-breadth (Q1) decisions + green build*. Orbit content lands in the *source* position
  (D5/D6) across B1→B3; openspec-orbit (current repo) is the migration source-of-truth and **retires
  after parity (B3)**.
- **CLAUDE.md identity flips** — today it states orbit "is NOT a fork"; the new thing *is* a hard
  fork. Rewrite as part of A1.
- **Strip upstream's own `openspec/` dogfooding dir** — the 1.3.1 tree ships its own
  `openspec/changes/...`; orbit's `openspec/` takes that slot.
- Parkable A1 sub-decisions: repo name, npm package name, bin name (`openspec` vs `orbit` vs `opsx`).

### D11 — Tool breadth: Claude (primary) + Codex, shared mechanism-neutral bodies (RATIFIED; resolves Q1)

Target **Claude + Codex, Claude primary** — not all 26 adapters, not Claude-only. Grounded in how the
pipeline actually works:

- **The Codex adapter already exists** in the inherited 1.3.1 baseline (`adapters/codex.ts`): writes
  to `~/.codex/prompts/opsx-<id>.md` (global/absolute, NOT per-repo; frontmatter =
  `description` + `argument-hint`). "Adding Codex" is **not** adapter engineering.
- **The prose body is shared across tools by design** — adapters only change *path + frontmatter*,
  never the `instructions`/`content.body`. So Claude and Codex consume the *same* body.

→ The whole Claude-vs-Codex cost collapses to **one discipline: keep the shared body
mechanism-neutral.** Don't bake Claude-only mechanisms into bodies ("the Skill tool", hardcoded
`.claude/...` paths, colon `/opsx:` syntax — the last is already handled per-tool by
`transformToHyphenCommands`). These are Claude *flavor*, not Claude *requirements*; written neutrally
both tools work.

**Approach: both at once on content, Claude-first on polish.** B1/B3 write Claude-primary,
mechanism-neutral bodies + keep the codex adapter registered (free). This avoids the retrofit
prose-rewrite that "standalone Claude, then add Codex" would incur. Codex *runtime polish* is a small
**post-parity** change, not a phase — so D9's 6-phase plan is unchanged; D11 adds only a phrasing
discipline to B1/B3 (pairs naturally with D5/D6 markdown-as-source).

**Residual ✅ RESOLVED (verified 2026-06-05):** Codex **consumes Agent Skills via the SKILL.md
standard** (Claude-compatible), NOT primarily the `~/.codex/prompts/` commands the inherited
`codexAdapter` targets. Evidence: (a) local `~/.codex/skills/` is populated with real **multi-file**
skills (`skill-creator`, `skill-installer`, `openai-docs`, `imagegen` — each `SKILL.md` + `references/`
+ `scripts/` + `assets/`), while `~/.codex/prompts/` does not exist; (b) OpenAI docs — skills live in
`$CODEX_HOME/skills` (default `~/.codex/skills`, personal) + `.codex/skills/` (project), SKILL.md+
references shape, *"the same skills that work on Claude Code also work on Codex."*

Implications (refine D11):
- **Codex delivery = skills**, via the skill generator (`skillsDir:'.codex'` → `.codex/skills/`) — NOT
  the prompts-based `codexAdapter`, which targets a channel current Codex doesn't use. That inherited
  adapter is likely **irrelevant to orbit**. So Claude = skills + commands; **Codex = skills.**
- **Orbit's multi-file skills port to Codex natively** — identical `SKILL.md + references/` format, so
  D7's `references[]` generator growth serves Codex too, not just Claude.
- D11's **mechanism-neutral body discipline still holds** (Codex auto-loads skills vs Claude's "Skill
  tool" invocation), but the genericization tax is even smaller than assumed.
- Path nuance to confirm at B-time: project skills `.codex/skills/` vs the emerging cross-tool
  `.agents/skills` convention (docs mention both).

Sources: developers.openai.com/codex/skills · github.com/openai/codex/blob/main/docs/skills.md

---

## Open questions

### Q1 — Tool breadth: all 26 adapters or Claude-only? ✅ RESOLVED by D11

~~A template feeds all 26 registered adapters; orbit's prose is Claude-specific.~~ **Resolved:**
target Claude (primary) + Codex with shared **mechanism-neutral** bodies (D11). Both-at-once on
content (one phrasing discipline in B1/B3, no retrofit), Codex polish deferred post-parity. Residual:
verify whether Codex consumes skills or only prompts (tracked in D11).

### Q2 — Pragmatic vs principled gate model

Bolt staleness onto a file-artifact (cheap, ships fast, `status` lies a little) vs. a first-class
gate node (more work in `graph.ts`/`state.ts`/`types.ts`, refolds `apply`, makes the engine the true
source of "what's next + what's blocking"). Leaning principled (D3) but unratified.

### Q3 — How is freshness actually computed? (the crux mechanism)

A gate "passes relative to a snapshot of its inputs." What's the snapshot? Candidates: content hash
of the input artifacts at pass time (stored in the run-summary), git tree/commit, file mtimes. Hash
is most robust (survives non-semantic churn if normalized; deterministic). This mechanism is the
single most load-bearing unknown of the engine workstream.

### Q4 — What does the review gate gate? apply, archive, or both? Does audit-drift gate archive?

The skill bodies already imply: proposal-mode review gates `/opsx:apply`; system-mode review +
audit-drift gate `/opsx:archive` (audit-drift is documented as a pre-archive sweep). Needs to be made
explicit as graph edges.

### Q5 — Where does a project-wide check live in a per-change engine?

`audit-drift` is **not change-scoped** — it scans the whole repo (vocabulary residue, lens staleness,
cross-doc consistency, archive coherence). The artifact-graph is per-change. Does audit-drift stay
orthogonal (a global command), or become a special "global gate" the archive phase consults? The
engine has no concept of project-scoped phases today.

### Q6 — Telemetry: rip out posthog or make opt-in?

`telemetry/` + `posthog-node` + the Commander pre/postAction hooks. Isolated, easy to remove. A fork
almost certainly wants it gone or opt-in. (Likely a D-level decision once confirmed.)

### Q7 — Upstream rebase strategy

Once templates are edited in place, pulling future upstream fixes becomes a manual merge. Consistent
with orbit's existing "no auto-ingest, deliberate upgrades only" stance — the fork hardens it into
repo structure. But need a concrete policy (cherry-pick? periodic review? abandon upstream?).

### Q8 — The archive merge bug: fix in the fork or keep detecting?

`archive.ts` does replace-only requirement-block substitution with no base fingerprint / scenario
granularity → parallel changes touching the same requirement silently drop scenarios (documented in
upstream's own `openspec-parallel-merge-plan.md`). The fork is the natural place to fix the root
cause; orbit's audit-drift/review currently only *detect* the symptom. Fix now vs. later vs. never?

### Q9 — Cross-cutting conventions delivery shape

`orbit-conventions` (read-before-reference / change-completeness / pushback): injected snippet into
every workflow template (DRY source, matches D8) vs. a standalone always-installed skill. Interacts
with the markdown-as-source decision (D6).

### Q10 — A1 bootstrap chicken-and-egg ✅ RESOLVED by D10

~~Orbit dogfoods — changes to orbit are authored as openspec changes living in `openspec/changes/`.
But A1 restructures the very repo that holds those changes.~~ **Resolved:** D10 chooses a fresh repo
born as the fork (1.3.1 as commit 1), so there is no in-place restructure and no paradox. A1 happens
in the new repo before any orbit `openspec/changes/` exist there; the change workflow resumes for
B1 onward.

---

## Considered & out

- **Continue the pure overlay model (status quo).** Out — it's the exact thing being solved; the
  generator-divorce, drift-fighting, and second-class net-new workflows are inherent to it.
- **Make all 4 orbit workflows graph artifacts/gates.** Out — over-reach. Reclassification (D5) shows
  only `review` is a true gate; address-reviews/review-external/audit-drift are a remediation loop, an
  executor, and a global check respectively.
- **Inline prose as TS template literals (upstream style).** Considered, rejected for orbit (D6) —
  regresses the markdown-native editorial discipline that is orbit's core value.
- **Treat review as a file-existence artifact (no staleness).** Considered as the "pragmatic" path
  (Q2); disfavored because it discards the staleness prize and makes `status` untrustworthy.

---

## References

- `openspec/architecture/openspec-1.3.1-architecture.md` — the deep-dive walkthrough (this
  exploration's parent grounding doc; §3 = engine, §5 = generator/content, §7 = archive bug, §11 =
  fork decisions).
- v1.3.1 source clone (scratch): `/tmp/openspec-src` (cloned from `github.com/Fission-AI/OpenSpec`,
  tag `v1.3.1`).
- Engine source: `src/core/artifact-graph/{types.ts,graph.ts,state.ts,resolver.ts,instruction-loader.ts}`;
  `src/commands/workflow/instructions.ts` (the `apply` state machine); `schemas/spec-driven/schema.yaml`.
- Generator source: `src/core/templates/{types.ts,skill-templates.ts,workflows/*.ts}`;
  `src/core/shared/skill-generation.ts` (`generateSkillContent`, the two registration arrays);
  `src/core/command-generation/{types.ts,generator.ts,registry.ts,adapters/claude.ts}`;
  `src/core/{init.ts,update.ts}`.
- `openspec-parallel-merge-plan.md` (in the upstream repo) — the archive merge data-loss analysis (Q8).
- Memories: `openspec-generation-seam`, `orbit-single-repo-fork-goal`.
