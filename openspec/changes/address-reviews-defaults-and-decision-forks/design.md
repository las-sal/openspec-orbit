## Context

`/opsx:address-reviews` ships in lean v1 with three conservative defaults that hurt orbit's dogfooding loop (see `proposal.md` Why section + `explore.md` Premise). All three live on the same Step 3 walk axis of the same skill (`openspec-address-reviews`). The empirical case for flipping each:

- **#14 (P0 — cascade)**: `bootstrap-orbit-status-cli` (cross-repo: the change lives in sibling repo `las-sal/orbit-status`, archived at `openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/`; cited here as empirical evidence per the same cross-repo pattern used in baseline `orbit-review/spec.md`'s 3-of-3 evidence cite) iter-N+1 system-mode review caught 5 WARNINGs already named in iter-N's `ripple_flagged_files_aggregate` and ignored. Ripple-flag-without-cascade silently loses work.
- **#11 (P1 — walk-mode)**: every iter of every recent change asked "walk them?" verbally; the v1 batch-by-default never fires in practice. UX is already walking; the spec is just out of sync.
- **#18 (P2 — decision forks)**: bootstrap-orbit-status-cli W1 (test coverage gap) had a two-option recommendation; the AI flattened it to a passive line until user pushback. Decisions hidden inside `recommendation` prose don't surface naturally.

The three flips are highly interconnected:

```
Step 3 walk (per finding):
  ┌──────────────────────────────────────────────┐
  │ 1. pushback (verify against current state)   │
  │ 2. classify (stale / trivial / decide / unr.)│
  │ 3. [NEW] decision-fork detection             │  ← #18 fires here
  │      ↳ only when classify == "decide req."   │
  │ 4. fix (apply user choice or trivial)        │
  │ 5. ripple-cascade [NEW DEFAULT]              │  ← #14 fires here
  │      ↳ IN-set files → applied[]              │
  │      ↳ OUT-set files → flagged_not_applied[] │
  │ 6. remove-marker (invariant; --keep override)│
  └──────────────────────────────────────────────┘
       ↑ walk-mode runs this whole inner cycle  ← #11 governs the loop
         per-finding (default) or batch (opt-in)
```

This design captures the three coordinated changes plus the supporting JSON-shape updates and producer-side affordances.

## Goals / Non-Goals

### Goals

- **Walk-mode by default**: per-finding granularity is the lifecycle default; batch-mode is opt-in via `--batch` flag or verbal trigger in the invocation message.
- **Cascade by default**: ripple-flagged IN-set files (any extension) auto-apply; `--no-cascade` opts out. Ripple-flagged OUT-set files (the four lifecycle-invariant categories: `.orbit-runs/`, baseline specs, cross-change dirs, safe-exclusions) record in `flagged_not_applied[]` for audit-trail.
- **Decision-fork detection**: disjunctive recommendations surface as `AskUserQuestion` forks rather than collapsing to passive prose. Hybrid detection (structured `recommendation_options[]` for orbit-emit pipelines; conservative heuristic for external markdown).
- **Audit-trail completeness**: resolution log JSON captures `walk_mode`, `walk_mode_source`, `recommendation_fork`, and the cascaded vs. flagged-not-applied split.
- **Producer-side structured affordance**: `orbit-review` and `orbit-audit-drift` emit optional `recommendation_options[]` when their finding's recommendation is disjunctive — supporting the structured-detection path for #18.

### Non-Goals

- **Cross-change cascade** — explicitly OUT per #14. Cascading into another active change's directory is out of scope (the other change's authoring is its own context).
- **Auto-baseline-spec-edits during address-reviews** — baseline mutations flow through the proposal cycle (`/opsx:propose` → delta in current change → `/opsx:archive` triggers `sync-specs`), NOT via cascade-time direct baseline edits. Cascade refuses baseline writes and logs `flagged_not_applied[]` with reason guiding the user to add a delta.
- **Baseline `openspec/specs/` cascade** — sync-specs territory; baseline propagation happens at archive time.
- **`.orbit-runs/` cascade** — audit-trail; never edited.
- **Mid-walk loose-phrase mode-switch** — only `--batch` flag, verbal `--batch` in the invocation message, OR a bare command-shaped mid-walk message ("go batch", "switch to batch") shifts mode. Loose conversational phrases ("yeah just fix them all") do NOT shift mode.
- **Global opt-out for #18 decision forks** — none. `[discuss]` inside each fork is the per-prompt escape; no flag suppresses fork detection. Designing for hypothetical callers wanting legacy "flatten" behavior would add flag surface for no real use case.
- **Loose-or-detection** — the heuristic detector does NOT trigger on prose like "X or Y could happen" or "fix it now or later." Only strict signals: numbered alternatives, "either…or" with clause-level branches, or `**Options:**` prefix.
- **Hard schema change to existing `recommendation` field** — the structured `recommendation_options[]` field is ADDED alongside, not replacing. Backward-compatible.
- **Cross-finding cascade aggregation in walk-mode** — each per-finding cycle applies ripples independently. Aggregating cascades across findings (e.g., to deduplicate edits) is a v2 refinement.

## Decisions

### D1 — Spec delta placement (deviation from explore.md D4)

**Decision**: All address-reviews resolution-log shape spec lives in `orbit-address-reviews` (as the new ADDED requirement `Resolution-log JSON shape extensions for walk-mode / decision-forks / cascade`), NOT `orbit-run-summary-emit`.

**Why** (deviation rationale): explore.md D4 placed the resolution-log shape MODIFIED operation in `orbit-run-summary-emit`. Re-reading the baseline (`openspec/specs/orbit-run-summary-emit/spec.md:25`): *"Existing emit-producing commands (/opsx:review, /opsx:address-reviews, /opsx:archive, inline /opsx:audit-drift during archive) continue emitting as today; this change does not modify their emit behavior."* The orbit-run-summary-emit capability explicitly excludes address-reviews emit from its scope. The address-reviews emit shape's natural home is the orbit-address-reviews capability itself (where the command's other emit semantics already live via `Resolution log output`).

**Alternatives considered**:

- **A** (explore.md D4 original): MODIFY a requirement in `orbit-run-summary-emit`. Rejected — placement violates the capability's stated scope.
- **B** (chosen): ADD a new requirement in `orbit-address-reviews` covering the JSON shape additions. Clean placement; doesn't widen any capability's purpose.
- **C**: split — put `walk_mode` top-level in orbit-run-summary-emit (universal-spine-adjacent) and `recommendation_fork` + `ripple_cascade` in orbit-address-reviews. Rejected — increases ops count and creates cross-capability coupling for fields that all serve the same command's audit-trail.

### D2 — `Ripple flag without auto-cascade` rewrite path

**Decision**: MODIFIED-with-rename via `## RENAMED FROM`. Title changes to `Ripple cascade by default`; body fully rewritten; scenarios refreshed.

**Why**: the v1 title literally contradicts new behavior. Keeping it (Path B in explore.md) leaves a permanent name-vs-behavior smell. Splitting into REMOVED + ADDED (Path C) is two operations where one will do — openspec's MODIFIED supports `## RENAMED FROM` directives natively.

**Alternatives considered**:

- **A** (chosen): MODIFIED-with-rename. One operation; clean title that matches behavior.
- **B**: MODIFIED keeping old title, body flipped. Rejected — title `Ripple flag without auto-cascade` would persist forever, misleading readers.
- **C**: REMOVED `Ripple flag without auto-cascade` + ADDED `Ripple cascade by default`. Rejected — two ops, more spec churn, no upside vs. (A).

### D3 — Walk-mode trigger semantics (strict-with-narrow-accommodation)

**Decision**: `--batch` flag is canonical. Verbal trigger phrases ("fix them all", "batch them", etc.) honored ONLY in the user's invocation message (the message that initiated `/opsx:address-reviews`). Mid-walk step responses do NOT shift mode through phrase-detection. Bare command-shaped mid-walk interruptions ("go batch", "switch to batch") DO shift mode (treated as explicit verbal `--batch` for the remainder).

**Why**: heuristic phrase-detection during the walk is exactly the kind of non-determinism that violates #11's "predictable defaults" goal. The invocation-line phrase is well-bounded (one message; the user clearly intends `--batch` semantics for the whole run). Bare command-shape interruptions are clear user intent (a unilateral mode-switch directive). Loose conversational phrases mid-walk ("just keep going", "yeah fix all of them" said in passing) are NOT shifts — too easy to misfire.

**Resolution-log capture**: the resolution log distinguishes `walk_mode_source: "flag" | "verbal" | "command-shape-interruption"` so future auditors can see how batch-mode was entered.

**Alternatives considered**:

- **A** (chosen): strict-with-narrow-accommodation. Predictable; matches user intent in clear cases; refuses to silently shift on ambiguous prose.
- **B**: strict flag-only. Rejected — verbal "/opsx:address-reviews foo, fix them all" in the invocation message is unambiguous user intent; refusing to honor it would be over-rigid.
- **C**: liberal phrase-detection throughout the walk. Rejected — non-determinism risk; mid-walk responses are ambiguous (could be acknowledgment, could be mode-switch); user wouldn't know which way the AI interpreted them.

### D4 — Decision-fork detection: hybrid (structured + heuristic)

**Decision**: Hybrid. Orbit-emit pipelines (`orbit-review`, `orbit-audit-drift`) gain an optional `recommendation_options: [{label, body}]` field on each finding (producer-side). External markdown findings (where orbit doesn't control the format) use a conservative heuristic regex over the `**Description**:` content. The consumer (address-reviews) tries structured first; falls back to heuristic.

**Why**: pure-structured would block on producers we don't control (external AIs writing markdown — they emit `**Description**:` prose). Pure-heuristic would lose the cleaner contract for orbit's own pipeline and risk false positives on prose like "fix it now or later." Hybrid combines: clean contract for orbit-emit, best-effort detection elsewhere.

**Heuristic strictness**:

- Triggers: numbered alternatives (`(A) … (B)`, `1. … 2.`, `[A] … [B]`), "either … or" with clause-level branches, "Options:" prefix followed by a list (both bold variants `**Options**:` and `**Options:**` are accepted after markdown normalization, per spec).
- NOT triggers: loose "or" in prose. False-negative bias (better to miss a fork than to fire one inappropriately).

**Source recording**: resolution log captures `recommendation_fork.source: "structured" | "heuristic"` for downstream auditing.

**Alternatives considered**:

- **A** (chosen): hybrid. Best of both worlds; clean contract where we control producer; conservative best-effort elsewhere.
- **B**: structured-only. Rejected — external markdown can't be structured; we'd lose #18 coverage for external findings entirely.
- **C**: heuristic-only. Rejected — wastes the producer-side opportunity; pure prose-parsing has false-positive risk we don't need.

### D5 — Fork-prompt firing point (after classify, decision-required only)

**Decision**: Decision-fork prompts fire within Step 3's walk, AFTER classify, BEFORE fix, and ONLY when classify == "decision required." Other classifications (`stale`, `trivial fix`, `unresolvable`) short-circuit BEFORE fork detection.

**Why**: a fork is a refinement of the existing "decision required" path. Stale findings shouldn't waste user attention on a decision; trivial fixes have an unambiguous answer; unresolvable findings get filed/escalated, not decided. Placing fork detection inside the "decision required" branch keeps the spec delta surface minimal (no new classification dimension) and matches the natural lifecycle order.

**Alternatives considered**:

- **A** (chosen): fork inside "decision required" branch, after classify.
- **B**: fork as a new classification dimension parallel to stale/trivial/decision/unresolvable. Rejected — more spec surface; classification is about findings, fork detection is about recommendations within findings.
- **C**: fork before classify. Rejected — would surface forks on stale findings that the user shouldn't have to think about.

### D6 — No global opt-out for #18

**Decision**: no flag (e.g., `--no-decision-forks`) suppresses fork prompts. The `[discuss]` option inside each prompt provides per-prompt escape.

**Why**: surfacing a disjunctive recommendation as a fork is information delivery; the `[discuss]` button already lets the user escape to a tradeoff conversation if the fork itself isn't quite right. Adding a flag would design for a hypothetical caller wanting the legacy passive-line behavior — orbit's principle is "don't add flags for scenarios that can't happen." No empirical use case for the opt-out exists.

**Alternatives considered**:

- **A** (chosen): no flag; `[discuss]` is the escape.
- **B**: `--no-decision-forks` opt-out flag. Rejected — flag surface for no use case; the `[discuss]` button already handles every edge case.

### D7 — Cascade scope: lifecycle-invariant OUT list only; everything else IN regardless of extension

**Decision**: Cascade scope is defined by exclusion. The OUT list captures four lifecycle-invariant categories. Any ripple-flagged file NOT in the OUT list is IN, regardless of extension (`.py`, `.swift`, `.c`, `.sh`, `.md`, dotfiles, configs — all eligible).

**OUT categories** (all four are structural workflow constraints, not safety heuristics):

1. **`.orbit-runs/*`** (both change-scoped and project-scoped) — audit-trail; editing corrupts the workflow's own record.
2. **Baseline `openspec/specs/<capability>/spec.md`** — baseline mutations flow through the proposal cycle (`/opsx:propose` → delta spec in current change → `/opsx:archive` triggers `sync-specs` to propagate). Cascade refuses direct baseline edits. When cascade sees a ripple flag pointing at baseline, it logs `flagged_not_applied[]` with reason guiding the user to add a delta in the current change's `specs/<capability>/spec.md`.
3. **Cross-change directories** (`openspec/changes/<other-name>/`) — change-isolation invariant; each change's authoring is its own context.
4. **Safe-exclusions** (`.git/`, `node_modules/`, `dist/`, `build/`) — universal "never edit" set.

**Everything else is IN**. Cascade applies the same pushback + classify + fix lifecycle to any ripple-flagged file in the IN set, regardless of extension. The safety mechanisms (pushback against current state, decision-fork prompts for "decision required" classifications, `--no-cascade` per-invocation opt-out, `[discuss]` per-prompt escape hatch) work uniformly across file types.

**Why this framing — file extension is not a discriminator**:

The earlier draft excluded "code files" on the rationale that "code files have unbounded blast-radius; markdown-only keeps the safety case preserved." That argument doesn't hold up:

- Address-reviews ALREADY auto-edits the primary-fix file regardless of extension. If `/opsx:review` produces a finding pointing at `src/handlers.py:142`, address-reviews edits `handlers.py:142` via the existing lifecycle. The safety mechanisms protecting that primary edit (pushback, decision-required prompts) apply equally to ripple edits in the same file or any other code file.
- Excluding code from cascade would make the cascade default trivial for consumer repos (Swift, Python, C, etc. projects using orbit as their workflow framework) — cascade would only ripple through markdown, which in those repos is mostly docs. The "5 WARNINGs silently dropped" empirical case from `bootstrap-orbit-status-cli` happened in markdown; the equivalent risk in a Swift project would be in `.swift` files.
- The principled OUT list is about **lifecycle plumbing** (audit-trail, baseline-spec mutation pattern, change isolation), not "code is dangerous."

**Framework vs. dev project — context-sensitive without special-casing**:

When orbit runs on a consumer repo (e.g., `homeENV`), the user's project files (`.bashrc`, `Brewfile`, dotfiles, scripts) are IN — they ARE the dev project. Orbit's own machinery (`.orbit-runs/`, baseline `openspec/specs/`) is OUT regardless of repo.

When orbit develops itself, `.claude/skills/`, `.claude/commands/`, README, etc., are IN — because here, those files ARE the project being developed. The same OUT-list rules produce the right behavior in both contexts; no `is-this-orbit-self-development?` flag needed.

The contextual scope judgment ("is THIS file relevant to my finding?") lives in the earlier **ripple-flag analysis** step — the AI deriving which files a fix actually affects. If ripple-flag analysis didn't surface a file as a ripple target, cascade never sees it. Cascade just trusts ripple-flag's output and applies the fix to anything IN.

**Alternatives considered**:

- **D** (chosen): lifecycle-invariant OUT list only; everything else IN regardless of extension. Pushback + decision-fork prompts + `--no-cascade` provide safety uniformly.
- **A** (earlier draft): markdown-only IN with sub-categories (change-dir markdown + project-level governing docs). Rejected — over-conservative; limits orbit's general-purpose use for non-markdown projects; the file-extension discriminator was not principled.
- **B**: include all markdown in repo (including `.claude/skills/`, `README.md`, etc.) but exclude code. Rejected — same file-extension-discrimination flaw as A.
- **C**: per-finding user-confirmation on every ripple regardless of category. Rejected — defeats the purpose of cascade-by-default (the whole point is to stop dropping ripple work; constant per-ripple prompts would re-introduce the friction).
- **D-future**: cascade-baseline-via-delta — when ripple points at baseline, cascade auto-generates the delta operation in the current change's `specs/<capability>/spec.md` instead of refusing. Out of scope for v1 (auto-generating spec-delta operations requires understanding ADDED vs MODIFIED vs RENAMED FROM semantics). Worth a follow-up issue.

### D8 — Producer-side `recommendation_options` is shared across orbit-review and orbit-audit-drift

**Decision**: The optional `recommendation_options: [{label, body}]` field shape is defined identically in both `orbit-review` and `orbit-audit-drift` specs. Consumer-side (address-reviews) reads them uniformly.

**Why**: the field is the producer-side affordance for the structured-detection path. Having two identical-but-separate definitions per producer is more redundant than ideal, but the alternatives (one shared spec home, or define-once-with-cross-reference) create more complexity than the duplication saves. Each producer's spec is self-contained for the field; the consumer's spec (orbit-address-reviews's `Disjunctive recommendation fields surface as decision forks`) cross-references both producers.

**Alternatives considered**:

- **A** (chosen): identical-field-shape in each producer's spec.
- **B**: define the field in `orbit-run-summary-emit` (universal-spine home), reference from both producers. Rejected — orbit-run-summary-emit doesn't currently spec finding-shape; widening it for this would expand the capability's purpose.
- **C**: define a new `orbit-finding-format` capability and reference from all consumers/producers. Rejected — over-engineering for a single optional field; capability proliferation has its own coherence cost.

## Risks / Trade-offs

| Risk | Mitigation |
|---|---|
| **Cascade silently edits files the user didn't expect** | Cascade scope is documented in the requirement body + scenarios; `flagged_not_applied[]` records every skip with reason; `--no-cascade` is a one-flag opt-out. |
| **Heuristic detector misfires** (e.g., loose "or" in prose triggers fork prompt) | Heuristic strictness rules are spec'd (numbered alternatives + clause-level "either…or" + `**Options:**` only); false-negative bias intentional. `[discuss]` escape gives user recovery if a misfire happens. |
| **Verbal `--batch` accidentally triggers in user's invocation message** | Only the invocation message is scanned (not subsequent messages); the recognized phrases are explicit batch-intent ("fix them all", "batch them", etc.); ambiguous phrases default to walk-mode. |
| **Mid-walk command-shape interruption is too liberal** | Requires bare unambiguous mode-switch messages ("go batch", "switch to batch"); conversational continuations ("yeah let's just keep going") do NOT shift mode. Spec'd scenario for the negative case. |
| **Producer-side `recommendation_options` field is forgotten by emitters** | Field is OPTIONAL; absence triggers heuristic fallback at the consumer. Worst case: the structured-path benefit is missed for that finding, but heuristic still catches obvious cases. Not a hard failure. |
| **Resolution-log v1 format readers break on new fields** | Forward-compat — readers SHOULD ignore unknown fields. Backward-compat — the v1 top-level `ripple_flagged_files_aggregate` field is REPLACED (not renamed) by per-resolution `ripple_cascade.applied / flagged_not_applied` (structurally different shape; NOT bijective — per-resolution attribution is lost when reading v1). Reader-side migration: consumers checking for which field is present route to the matching parser; downstream tooling (e.g., `orbit-status`, which is the sibling-repo consumer documented in `orbit-conventions` + `orbit-review` + `orbit-run-summary-emit` baselines) does best-effort parse and degrades gracefully. |
| **Cascade misidentifies a file as IN when it should be OUT** | The OUT list is structural-lifecycle-only (4 categories: `.orbit-runs/`, baseline `openspec/specs/`, cross-change dirs, safe-exclusions). The IN/OUT determination is path-prefix matching, not extension- or content-based — predictable and easily verified. Safety mechanisms (pushback against current state, decision-fork prompts, `--no-cascade` per-invocation opt-out) protect IN edits uniformly. If a path matches none of the 4 OUT prefixes, cascade treats it as IN regardless of file type — that's the Option D framing. |

## Migration Plan

No data migration required. Schema additions are additive on the input side; the output side has one structural change:

1. **Resolution-log JSON** (`address-reviews-<TS>.json`): structural shape change. v1 had a top-level `ripple_flagged_files_aggregate` flat array (aggregated across all resolutions). v2 has per-resolution `ripple_cascade.applied / flagged_not_applied` objects. The shapes are NOT bijective — per-resolution attribution is captured in v2 but absent from v1; readers handling both formats must detect which field is present and route accordingly. Old archived JSONs retain v1 format (no in-place migration).
2. **Finding emit JSON** (`review-<mode>-<TS>.json`, `audit-drift-<TS>.json`): the `recommendation_options` field is optional; absence triggers heuristic fallback at consumer-side. No backfill needed.
3. **No flag-set changes** beyond adding `--batch` (opt-in) and `--no-cascade` (opt-out). Existing flags (`--from-file`, `--keep-resolved-markers`, scope positional) unchanged.

**Rollback**: behavioral changes are flag-gated for opt-out (`--no-cascade` reverts to v1 cascade behavior); walk-mode opt-out is `--batch` (which gives v1 batch-mode behavior). Decision-fork detection has no rollback flag, but `[discuss]` in every prompt provides per-prompt escape. If catastrophic, a `--legacy-defaults` superflag could be added in a follow-up.

## Open Questions

(none — all resolved in explore.md Decisions D1–D7 and the design-time decisions D1–D8 above)
