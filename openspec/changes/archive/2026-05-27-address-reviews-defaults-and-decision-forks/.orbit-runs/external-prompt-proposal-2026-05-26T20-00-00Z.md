# External Review Request: `address-reviews-defaults-and-decision-forks` (proposal mode, iter-1 external)

You are running a second-opinion editorial review of an OpenSpec change in the **openspec-orbit** repo. This is a proposal-mode (pre-apply) review of artifacts. Your job is to surface findings that internal review missed.

## How to read this prompt

This file is self-contained — read it top to bottom, then pull the repo (`https://github.com/las-sal/openspec-orbit`) and read the files it references. Write your findings file at the path specified in the **Output** section below.

## Cycle context

Two internal proposal reviews + two address-reviews walks have happened:

- **iter-1 internal** (`.orbit-runs/review-proposal-2026-05-26T18-00-00Z.json`): 7 findings (1 CRITICAL, 2 WARN, 4 SUGG)
- **iter-1 walk** (`.orbit-runs/address-reviews-2026-05-26T18-30-00Z.json`): all 7 resolved. **CRITICAL drove a substantive scope reframe** — user proposed "Option D" superseding the originally-surfaced A/B/C options. Cascade scope changed from "markdown-only IN with sub-categories" to "lifecycle-invariant OUT list only; everything else IN regardless of extension." This was a real architectural shift, not a localized fix.
- **iter-2 fresh-subagent internal** (`.orbit-runs/review-proposal-2026-05-26T19-00-00Z.json`): 5 findings (0 CRITICAL, 2 WARN, 3 SUGG) — caught residue from the Option D reframe that the in-context iter-1 walk missed.
- **iter-2 walk** (`.orbit-runs/address-reviews-2026-05-26T19-30-00Z.json`): all 5 resolved as trivial fixes.

The change has gone through 2 full review-and-resolve cycles. `openspec validate --strict` passes. You're being asked to test whether the Option D reframe + supporting design holds up under a fresh cross-AI read.

**Do not assume prior findings are wrong.** Read the current state. But also: don't anchor on prior findings either — your fresh perspective is the point.

## What the change does

Bundles 3 GitHub issues:

- **#14 (P0)** — cascade by default: `/opsx:address-reviews` Step 3e auto-applies ripple-flagged edits (vs. v1 list-only). `--no-cascade` opts out.
- **#11 (P1)** — walk-mode default: each finding gets its own pushback → classify → fix → cascade → remove-marker cycle (vs. v1 batch-by-default). `--batch` opts INTO legacy.
- **#18 (P2)** — decision-fork detection: disjunctive `recommendation` fields surface as A/B/discuss forks via `AskUserQuestion` (vs. v1 passive-line flattening). Hybrid detection: structured `recommendation_options[]` from orbit producers + heuristic over external markdown.

Plus supporting JSON shape additions (`walk_mode`, `recommendation_fork`, `ripple_cascade.applied/flagged_not_applied`) and producer-side `recommendation_options[]` field on orbit-review + orbit-audit-drift.

## Repository context

- Repo URL: `https://github.com/las-sal/openspec-orbit`
- Repo root: `openspec-orbit/`
- Project conventions: `CLAUDE.md` at repo root (orbit is NOT a fork; pegs to `@fission-ai/openspec@1.3.1`; three execution disciplines: read-before-reference / change completeness / pushback)
- Change directory: `openspec/changes/address-reviews-defaults-and-decision-forks/`
- Baseline specs (the canonical state being mutated):
  - `openspec/specs/orbit-address-reviews/spec.md`
  - `openspec/specs/orbit-review/spec.md`
  - `openspec/specs/orbit-audit-drift/spec.md`
  - `openspec/specs/orbit-run-summary-emit/spec.md`
  - `openspec/specs/orbit-conventions/spec.md`

## Files to read in the change directory

- `proposal.md` — what/why/scope
- `design.md` — 8 design decisions (D1–D8); D7 is the cascade scope decision (the C1-iter1 reframe); Risks/Trade-offs table; Migration Plan; Alternatives Considered
- `explore.md` — historical pre-propose staging record (5 sections: Premise / Decisions D1–D7 / Open questions / Considered & out / References). Frozen; do not edit. Useful as context for how the bundle was scoped.
- `specs/orbit-address-reviews/spec.md` — 5 ops: MODIFIED `Discover → triage → walk → ripple flag → report lifecycle`, MODIFIED-with-rename `Ripple flag without auto-cascade` → `Ripple cascade by default`, ADDED `Walk-mode by default with --batch opt-in`, ADDED `Disjunctive recommendation fields surface as decision forks`, ADDED `Resolution-log JSON shape extensions for walk-mode / decision-forks / cascade`
- `specs/orbit-review/spec.md` — ADDED `Optional recommendation_options field on finding entries` (5 scenarios)
- `specs/orbit-audit-drift/spec.md` — ADDED `Optional recommendation_options field on audit-drift finding entries` (6 scenarios)
- `tasks.md` — 8 task groups in 5 chunks (Chunk 4 = docs, Chunk 5 = user-validation handoff)

## Files to read in `.orbit-runs/` (for cycle context only)

You may read the prior internal review + walk JSONs to understand what's already been examined. Do NOT just re-surface prior findings; bring fresh perspective.

## Pass instructions (proposal mode, 9 passes)

Run all 9 passes. Each pass produces findings (possibly zero) tagged CRITICAL / WARNING / SUGGESTION. Bias toward lower severity when uncertain; bias toward false-negative (miss > misfire).

1. **Structure & Delta** — artifact-completeness check. proposal/design/specs/tasks all present? Delta files use proper `## ADDED/MODIFIED/REMOVED/RENAMED FROM` operations? Scenarios use 4 hashtags (`####`)? Every requirement has ≥1 scenario? Spec deltas land in the right capability directories?
2. **Internal Coherence** — within each artifact, does it agree with itself? Within design.md, do D1–D8 contradict each other? Within spec deltas, do scenarios match the requirement body's normative claims? Within tasks.md, does the chunk preamble match group structure?
3. **Cross-Doc Consistency** — proposal.md ↔ design.md ↔ specs/ ↔ tasks.md ↔ explore.md. Do scope claims in proposal match what specs deliver? Does design.md's Goals section match proposal.md's What Changes? Do task file paths reference real files? Does explore.md's Decisions section align with design.md's final decisions?
4. **Archive Consistency** — does the change make claims about prior archived state that don't hold? (e.g., "renames X → Y" — verify X exists at the old name; verify no other in-flight changes carry references to the old name).
5. **Codegen Readiness** — for each task, can an implementing AI run it without ambiguity? Are file paths real? Are flag names consistent across SKILL.md, command-mirror, and spec? Does the chunk preamble parse cleanly per `orbit-run-summary-emit`'s `Apply per-chunk-end emission` format constraints?
6. **Gap Hunt** — for each requirement in the spec deltas, ask: (a) unstated assumptions an implementer would have to invent (file paths? defaults? edge cases?)? (b) error/edge-case paths specified or only happy paths? (c) state transitions explicit? (d) is "X SHALL do Y" precise enough that two implementers produce the same behavior?
7. **Drift Hunt** — sweep for residue from prior framings. The iter-1 walk applied "Option D" which changed cascade scope from "markdown-only with sub-categories" to "lifecycle-invariant OUT list only." Watch for stale terms: `code file; cascade off`, `in-scope markdown`, `project-level governing docs`, `project-level-skill umbrella`, `markdown-only`. Note: design.md "Alternatives Considered" section legitimately uses these terms when describing the REJECTED earlier draft — that's history, not residue.
8. **Inline `@review:` Marker Residue** — grep the change directory for `@review:` markers. Distinguish actual unresolved markers (CRITICAL) from documentation appearances of the marker syntax (e.g., `` `@review:` `` inside a backtick-quoted code reference, or inside a scenario describing the marker convention itself — those are NOT findings).
9. **Pre-Handoff Sweep** — overall readiness: does the proposal converge? Are there any unresolved decisions? Does the change introduce risk that's not surfaced in the Risks/Trade-offs table?

## Specific things worth scrutinizing for this change

The Option D reframe is novel and the hybrid decision-fork detection is subtle. Especially probe:

- **Cascade scope correctness**: the OUT list claims 4 categories cover all lifecycle-invariant exclusions. Are there cases NOT in the OUT list that SHOULD be (e.g., `package-lock.json`, lock files, `.openspec.yaml` itself, generated files)? Or cases IN the OUT list that have edge cases not covered (e.g., `.orbit-runs/` paths inside archived changes)?
- **`--no-cascade` semantics**: the spec says `--no-cascade` records ALL ripples in `flagged_not_applied[]` regardless of IN/OUT classification. Does this lose information vs. the IN/OUT determination?
- **Cascade trusts ripple-flag analysis**: the spec says cascade does not make independent scope judgments beyond the OUT check. But cascade DOES make path-prefix-matching judgments. Is "trusts" too strong? What if the ripple-flag analysis itself is buggy?
- **Walk-mode UX trigger detection**: 3 trigger modes (flag, verbal-in-invocation, mid-walk command-shape). Is "command-shape" detection well-defined enough that two implementers agree on what's a command vs. conversational continuation?
- **Decision-fork heuristic strictness**: the heuristic only triggers on numbered alternatives, clause-level "either…or", and `**Options:**` prefix. Does this miss common disjunctive patterns? Does it over-trigger on edge cases?
- **Hybrid detection routing**: when does the consumer try structured vs. heuristic? The spec says "try structured first; fall back to heuristic on malformed input." Does this routing handle every edge case? What about partial structure (e.g., 1 entry instead of 2)?
- **Producer-side enforcement**: orbit-review + orbit-audit-drift specs require ≥2 entries. Is this enforced at emit time, or only documented as "producer SHALL"? What's the failure mode if a producer emits a 1-entry array?
- **Resolution-log shape change (v1→v2)**: the structural change loses per-resolution attribution when reading v1. Is the migration story complete? Does `orbit-status` (cross-repo consumer) handle the v1↔v2 distinction?
- **Cross-repo references**: the change cites `bootstrap-orbit-status-cli` (lives in sibling `las-sal/orbit-status` repo). Are the citations accurate? Defensible without in-repo evidence?

These are PROBES, not guaranteed findings. If you don't find anything substantive, "None" is the right answer.

## Pushback discipline

**Do NOT apply pushback at your end.** That's address-reviews' job at ingest time. Flag everything you observe. If you think a prior iter-1 or iter-2 finding still applies (despite being marked resolved), say so — address-reviews will re-verify against current state.

## Output format

Write your findings to: `openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-proposal-<YOUR-TS>.md` where `<YOUR-TS>` is the current UTC time in `YYYY-MM-DDTHH-MM-SSZ` format (ISO-8601 with colons replaced).

Format MUST match this exact structure (orbit external-review markdown format; address-reviews `--from-file` parses it):

```markdown
# External Review: address-reviews-defaults-and-decision-forks (iteration 1)

**Reviewer**: <your model name, e.g., "GPT-5 Codex" or "Claude Opus 4.X" or "Gemini">
**Date**: 2026-05-26

## CRITICAL

### <Finding title>
**File**: <repo-relative-path>:<line>
**Description**: <what's wrong + specific recommendation>

(If no CRITICAL findings, write the single body line `None.`)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(If no WARNING findings, write `None.`)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(If no SUGGESTION findings, write `None.`)

## Notes

<Optional: overall impression, broader concerns, or recommendation for next step>
```

**Format constraints**:
- Severity headers exactly `## CRITICAL`, `## WARNING`, `## SUGGESTION` (case-sensitive)
- Each finding's title under `### ` (3 hashtags + space)
- `**File**:` and `**Description**:` field labels exact
- `None.` (with trailing period) for empty severity sections
- The `## Notes` section is optional

Multi-line descriptions are fine within the `**Description**:` field — just continue on subsequent lines.

## When you're done

After writing your findings file, **commit and push it back to the remote** so the orbit user can pull it for ingest. Suggested commit pattern (matches prior orbit external-review commits):

```
git add openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-proposal-<YOUR-TS>.md
git commit -m "External review (proposal, iter 1): address-reviews-defaults-and-decision-forks"
git push
```

The orbit user will then `git pull` and run `/opsx:address-reviews address-reviews-defaults-and-decision-forks --from-file openspec/changes/address-reviews-defaults-and-decision-forks/.orbit-runs/external-proposal-<YOUR-TS>.md` (or rely on auto-discovery, which will resolve to your file if it's the most recent in `.orbit-runs/`).

Thank you for the second opinion.
