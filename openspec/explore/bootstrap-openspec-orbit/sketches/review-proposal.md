# Sketch: `/opsx:review-proposal`

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (coherence with openspec form and spirit) — adopts `verify-change`'s reporting convention verbatim.

## Purpose

Editorial review of a change's pre-implementation artifacts (`proposal.md`, `design.md`, `specs/*/spec.md`, `tasks.md`, plus `explore.md` for context). Codifies the recurring review-shape the user has been running manually across ~80 sessions in home-control / home-env / home-control-test.

**Generates findings. Does not resolve them.** Resolution flows through `/opsx:address-reviews` (markers, paste, finding-files) or by user replying directly to the report.

Pre-apply gate. Sister command of `/opsx:review-system` (post-apply) and `/opsx:verify-change` upstream (post-apply, structural-only).

## Inputs

- `<change-name>` (optional) — if omitted, prompt via `AskUserQuestion` from `openspec list --json`
- **Depth modes (mutually exclusive, pick one):**
  - `--fast` — cheap subset only: Pass 1 (Structure & Delta), Pass 7 (Drift Hunt), Pass 8 (Inline Review Marker Residue). Skips LLM-heavy passes (2/3/4/5/6/9). For mid-cycle quick sanity checks.
  - `--full` (**default**) — all 9 passes, sequential. The workhorse mode per guiding principle 2.
  - `--thorough` — all passes + extras (deeper Gap Hunt probe, Codegen Readiness with adversarial framing, Pre-Handoff Sweep gets a second read). Specifics tracked in GitHub issue. For pre-`/opsx:apply` deep clean.
- **Execution mode (orthogonal, combinable with any depth):**
  - `--parallel` — spawn subagents for heavy passes (2/4/6 specifically — internal coherence, archive consistency, gap hunt). ~3-4× wall-clock; also partitions context for larger projects. Opt-in.
- **Secondary flags:**
  - `--fresh` — main work runs in a clean-context subagent (different from `--parallel`; about *avoiding conversation framing*, not about *parallelism*). Opt-in for v1; revisit as default in v2.
  - `--mark` — additionally drop `@review: ...` markers into the relevant files based on findings, enabling unified resolution via `/opsx:address-reviews`. Default off.
  - `--focus <lens>` — emphasize one of: `rename`, `flip`, `refactor`, `extension` (see "Special-case lenses" below). Additive — raises rigor on named passes, doesn't skip others.
  - `--strict` — fail-fast on first CRITICAL finding. For CI-style usage.

## What it reads

Use openspec CLI as source of truth where possible (`openspec list --json`, `openspec status --change <name> --json`, `openspec instructions apply --change <name> --json`); fall back to direct file reads for non-CLI-exposed surfaces.

- **Change artifacts**: `openspec/changes/<name>/{proposal,design,tasks}.md` + `specs/*/spec.md` + `explore.md` (if present)
- **Project context**: `CLAUDE.md`, `openspec/project.md`, `*_convention.md` files
- **Baseline state**: `openspec/specs/*/spec.md` (current archived requirements) + `openspec/changes/archive/` (prior changes' context where relevant)
- **Project lenses** (if present): `openspec/lenses/perspectives.md`, `openspec/lenses/critical-paths.md` — used lightly by proposal-side cross-checks. Most relevant to `review-system`. Degrade gracefully if absent.

## Passes

Nine passes, each producing findings with CRITICAL / WARNING / SUGGESTION severity, file:line refs, and a specific recommendation. Bias toward lower severity when uncertain; never vague text like "consider reviewing."

### 1. Structure & Delta Integrity

- All required artifacts present? (`proposal`, `design`, spec deltas if change touches specs, `tasks`)
- Delta sections valid? (`## ADDED Requirements` / `## MODIFIED Requirements` / `## REMOVED Requirements` / `## RENAMED Requirements`)
- Tasks have requirement → task back-references where it makes sense?
- `openspec validate` passes?

### 2. Internal Coherence

- Proposal motivation aligns with design decisions?
- Design decisions align with spec requirements?
- Spec requirements covered by tasks?
- Tasks not introducing scope creep (work not in spec)?
- Numbers / counts / lists consistent across artifacts?
  *(Lesson from home-env codex feedback: "14 new types" in proposal vs "16 total" in design)*

### 3. Cross-Doc Coherence

- `CLAUDE.md` still accurate after this change lands?
- `project.md` vocabulary aligned with proposed names?
- `naming_convention.md` and other `*_convention.md` honored?
- If any need updates, flag and suggest exact edits.

### 4. Archive Consistency

- New `ADDED` requirements don't contradict existing archived requirements?
- `RENAMED` requirements' FROM symbols actually exist in the current baseline?
- `REMOVED` requirements aren't still referenced by other active specs (or by archived specs that should be updated together)?
- Vocabulary aligned with existing archived specs (catch terminology drift early).

### 5. Codegen Readiness

- Implicit requirements? (Things implementer must "figure out" — should be explicit)
- Decisions left to codegen? ("or similar", "TBD", "either X or Y" without a pick)
- Underspecified surfaces? (E.g., "HTTP server accepts commands on port 8765" with no endpoint table)
- Ambiguous units / types / ranges?
  *(Lesson from home-env codex feedback: thermostat `temperatureUnit` case)*

### 6. Gap Hunt (generative completeness probe)

The differentiating pass vs. plain `openspec validate`.

- Read artifacts AS IF a fresh AI was about to implement this — what questions would arise that these specs don't answer?
- Cover-set check: all devices, all interactions, all modes, all edge cases mentioned?
- Error paths specified? (Not just happy paths)
- State transitions explicit? (Including invalid transitions — what should happen)
- **Success criterion**: a fresh AI handed these artifacts could implement the system without inventing requirements.

### 7. Drift Hunt (vocabulary & shape)

- Old vocabulary still present in any of the artifacts?
  *(Grep against known-deprecated terms from archive + `*_convention.md`)*
- New artifacts using terms that exist with different meaning elsewhere?
- If `--focus rename`: extra emphasis on naming consistency across all artifacts.

### 8. Inline Review Marker Residue

- Any `@review: ...` markers still present in the artifacts?
- Each unresolved marker = CRITICAL.
  *(Lesson: "There are still resolved REVIEW comments in the active specs/design — they won't break anything, but remove them before applying so they don't land in canonical specs.")*

### 9. Pre-Handoff Sweep

Final-pass discipline: "I'm about to ship this to another agent / hit apply — anything else?"

- Surfaces small things the other passes deferred to SUGGESTION but matter on final read.
- Resolved-marker residue (lingering `@review: ...` comments even if their issue was already addressed in the artifacts).

## Special-case lenses (`--focus`)

Lenses are additive — raise emphasis on named passes, don't skip others.

| lens | shifts emphasis to | because |
|---|---|---|
| `rename` | Pass 7 (drift), Pass 3 (cross-doc), Pass 4 (archive) | renames leak old names everywhere (lesson from `rename-store-and-split-stubbed-home`) |
| `flip` | Pass 2 (internal coherence on directionality), Pass 5 (surfaces specified for BOTH directions) | the WebSocket-direction-reversal pattern; comms ownership must be unambiguous |
| `refactor` | Pass 4 (archive), Pass 7 (drift), Pass 3 (cross-doc) | old shape may linger in non-delta'd specs |
| `extension` | Pass 6 (gap hunt) | adding capability — coverage must match existing depth |

## Reporting

Adopts `verify-change`'s convention verbatim. The 9 passes roll up into 3 dimensions for the summary scorecard; individual findings list by severity.

### Pass → dimension roll-up

| Dimension | Passes |
|---|---|
| **Completeness** | 1 (Structure & Delta), 5 (Codegen Readiness), 6 (Gap Hunt) |
| **Correctness** | 2 (Internal Coherence), 4 (Archive Consistency) |
| **Coherence** | 3 (Cross-Doc), 7 (Drift Hunt), 8 (Inline Review Residue), 9 (Pre-Handoff Sweep) |

### Output template

```markdown
## Review Report: <change-name>

### Summary
| Dimension    | Status                                            |
|--------------|---------------------------------------------------|
| Completeness | 4/4 artifacts, 8 reqs scanned, 2 gaps             |
| Correctness  | 1 internal inconsistency, 1 archive contradiction |
| Coherence    | 2 doc-drift findings, 0 markers residue           |

### CRITICAL (1)
1. specs/host-lifecycle/spec.md:42 — RENAMED requirement
   refers to FROM symbol `BridgeServer` not found in
   current baseline; baseline has `HostLifecycle`.
   → Update RENAMED block, or use ADDED if this is new.

### WARNING (2)
…

### SUGGESTION (5)
…

### Final Assessment
1 critical issue. Fix before `/opsx:apply`.
Suggested next: address the RENAMED inconsistency,
then re-run /opsx:review-proposal --focus refactor.
```

### Final-assessment phrasings

| State | Phrasing |
|---|---|
| ≥1 CRITICAL | `X critical issue(s) found. Fix before \`/opsx:apply\`.` |
| Only WARNING / SUGGESTION | `No critical issues. Y warning(s) to consider. Ready to apply (with noted improvements).` |
| All clear | `All checks passed. Ready to apply.` |

## Heuristics & graceful degradation

- **Lower severity when uncertain** — same as verify-change.
- **Every finding actionable** — file:line + specific recommendation. Never "consider reviewing X."
- **Degrade gracefully**:
  - If `openspec/specs/` is empty (new project), skip Pass 4 with a note.
  - If no `*_convention.md`, skip those checks in Pass 3 / Pass 7.
  - If `openspec/lenses/` is empty or absent, skip lens-driven cross-checks with a note.
- **Pushback hook** — if a finding contradicts current state (e.g., flag is "stale ref to X" but X is already removed), self-correct and note "stale finding suppressed."

## Open design questions

1. **`--fresh` default?** Subagent gives clean read but loses speed. The cross-AI cycle exists *because* in-session review inherits framing. Lean: opt-in for v1; default in v2 if it proves better.
2. **External-feedback paste flow.** Cleanest model: review-proposal **generates** findings; `/opsx:address-reviews` **consumes** findings from any source (markers, paste, finding-file). Pastes route through address-reviews with `--from-paste`, not through review-proposal. Keeps the generative/consumptive split clean.
3. **`--mark` ordering vs Pass 8.** Pass 8 only flags *pre-existing* markers; `--mark` writes new ones AFTER the report is generated, based on report findings. Must not double-count.
4. **Cost of Pass 6 (Gap Hunt).** The most expensive pass — essentially "simulate implementation, report gaps." Could become its own command (`/opsx:probe-completeness`), or a flag (`--no-probe`). Lean: always run; it's the differentiator vs. just running `openspec validate`.
5. **Project lenses shape locked** (2026-05-17): `openspec/lenses/{perspectives,critical-paths}.md`, markdown with structural conventions, grown via `/opsx:explore` capture triggers. See `explore.md` decisions.

## Composition with related commands

```
explore ─── /opsx:explore [<name>]
              │
              ├── captures decisions to openspec/explore/<name>/explore.md
              │   (and convention captures to <name>/conventions/* as needed)
              ▼
propose ── /opsx:propose [<name>]
              │
              ├── reads explore.md; promotes openspec/explore/<name>/ →
              │   openspec/changes/<name>/
              │
              └── generates proposal, design, spec deltas, tasks
                      │
                      ▼
review ─── /opsx:review-proposal [<name>] [--focus …] [--mark] [--fresh]
              │
              ├── runs 9 passes → 3-dimension scorecard
              ├── (optional) drops @review: markers via --mark
              └── reports CRITICAL/WARNING/SUGGESTION findings
                      │
                      ▼
resolve ── /opsx:address-reviews
              │
              ├── consumes @review: markers
              ├── consumes external-feedback pastes (--from-paste)
              ├── applies pushback discipline (verify against current state)
              └── removes markers on resolution
                      │
                      └─── cycle (review → address → review …) until clean
                              │
                              ▼
apply ──── /opsx:apply
```
