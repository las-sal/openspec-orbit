## Context

Orbit's command surface today splits into two emission patterns:

- **Editorial commands write run-summary JSONs.** `/opsx:review`, `/opsx:address-reviews`, `/opsx:audit-drift` (inline within archive), and `/opsx:archive` each write `openspec/changes/<name>/.orbit-runs/<command>-<TS>.json` with a `next_recommended` field carrying orbit's recommendation for the next step.
- **Workflow commands emit nothing.** `/opsx:explore`, `/opsx:propose` (and variants `/opsx:new`, `/opsx:continue`, `/opsx:ff`), `/opsx:apply`, `/opsx:verify` all produce artifacts but no run-summary JSON. State is recoverable only indirectly, via artifact presence on disk.

This asymmetry creates a downstream synthesis problem. orbit-status — the change-state observability CLI — needs a "what's next?" recommendation for every change in the project. For changes that have had an editorial command run, orbit-status reads `next_recommended` directly (tier-1). For pre-review changes that only have workflow-command artifacts, orbit-status falls back to a Tier-2 synthesis layer (4 rules at `orbit-status-recommendation/spec.md:23–28`) whose sole purpose is to fill orbit's silence. Every future consumer (dashboards, CI integrations, IDE plugins) would have to re-invent the same synthesis logic.

The orbit-status dogfood (bootstrap-orbit-status-cli, archived 2026-05-20) confirmed the tier-2 layer was load-bearing for the v0.1 release, but documented the underlying gap as orbit's responsibility to close. This change closes it.

**Prior art**: orbit's overall emit architecture was established by the bootstrap-openspec-orbit change (archived 2026-05-18). That change introduced the `orbit-conventions` capability (including the `Internal-run JSON summary format` requirement this change modifies), the 3 per-skill `run-summary-schema.md` reference files for review/audit-drift/address-reviews, the `.orbit-runs/` persistence layout convention, and the editorial-command emit pattern. This change extends the established pattern to workflow commands while preserving the architecture.

## Goals / Non-Goals

**Goals:**
- Every artifact-mutating orbit command writes a run-summary JSON to `.orbit-runs/` with a documented schema (shared spine + per-command extensions + `next_recommended`)
- orbit's own recommendation becomes the single source of truth for "what's next" across the full command surface; downstream consumers stop synthesizing
- Per-command recommendation rules are explicit and predictable, mirroring the orbit canonical flow (`explore → propose → review → apply → verify → review --as system → archive`)
- `/opsx:apply` chunk-end emission enables forensic timeline reconstruction (which chunk introduced a regression) and unlocks the intra-apply review cadence improvement tracked in [#22](https://github.com/las-sal/openspec-orbit/issues/22)
- orbit-status's tier-2 synthesis becomes deletable (tracked separately as [orbit-status#2](https://github.com/las-sal/orbit-status/issues/2))

**Non-Goals:**
- Modifying any upstream skill's behavior. The emit-layer wraps upstream commands; it does NOT add features (e.g., verify does NOT gain marker-dropping; that stays `/opsx:review --mark`'s job).
- Modifying the capability **specs** for existing emit-producing capabilities (`orbit-review`, `orbit-address-reviews`, `orbit-audit-drift`, `orbit-archive-modifications`, `orbit-review-external`). Those specs' requirements stay as authored — no MODIFIED deltas for them in this change. Only `orbit-conventions` gets a MODIFIED delta (the universal-spine work). (Note: SKILL.md-level updates to those skills' actual emit instructions — adding `kind` to align with the universal spine — ARE in scope and tracked in `tasks.md` group 1.4; that's a behavior-implementation change, not a capability-contract change.)
- Modifying orbit-status's reader. Its existing tier-1 reads the new JSONs via the same path; tier-2 cleanup is the orbit-status side's separate change.
- Anticipating future issues' behavior changes in emit text (no forward-references to [#20](https://github.com/las-sal/openspec-orbit/issues/20)'s reviewer-mode inversion, etc.).
- Schema versioning. The `command` field implicitly versions per-command schemas; explicit `schema_version` adds overhead without current consumer demand.

## Decisions

### D-arch-1: Emit-layer lives in `## Orbit additions` sections; upstream body unchanged

The new emit behavior is implemented as additions inside `# Orbit additions` sections within each command's SKILL.md. The upstream-authored body of each SKILL.md (everything above the `# Orbit additions` marker) remains unchanged; the wrapper boundary IS the marker itself. For upstream-derived skills without an existing additions section (`openspec-new-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-apply-change`, `openspec-verify-change`), this change introduces the first additions section — additive, not modifying the upstream body above it.

Concretely: `/opsx:verify` keeps its upstream job — run `verify-change`, report pass/fail with findings. The recommendation logic that classifies verify's output (`/opsx:apply` on tasks-incomplete, `/opsx:review --as system` on impl-vs-spec gap [without `--mark` — that flag is proposal-mode only], verbatim validator message on openspec-validate failure) lives in `openspec-verify-change/SKILL.md`'s `# Orbit additions` section, NOT in the upstream body.

**Why this architecture works:**
- Upstream skill bodies receive `openspec update`-driven improvements unchanged; orbit's emit work lives below the marker and doesn't conflict
- Emit-layer can evolve independently of upstream skill logic
- The orbit/upstream boundary stays clean — easy to audit (everything above `# Orbit additions` is upstream-or-untouched; everything below is orbit's)
- Aligns with the delta-only-overlay principle established in the [openspec-orbit#6](https://github.com/las-sal/openspec-orbit/issues/6) feedback

**Alternative considered (rejected):** A SEPARATE wrapper artifact (e.g., a sibling `openspec-verify-change-emit.md` skill that's invoked after verify completes). Rejected because: (1) two artifacts per command doubles maintenance surface; (2) emit logic is tightly coupled to the command's output shape, so co-locating in the same SKILL.md keeps related concerns together; (3) AI invocation patterns work cleanly with single-skill emits because the AI completing the command's primary work naturally proceeds to the orbit-additions emit instructions in the same file.

### D-spine-1: 6-field shared spine + per-command extensions

Every run-summary JSON includes:

```
command          string   identifies which command emitted
timestamp        ISO-8601 UTC, also embedded in filename
change           string   the change name (or null for project-scope cmds)
final_assessment string   narrative of what just happened
next_recommended string   verbatim recommendation
kind             enum     "workflow" | "editorial" | "lifecycle"
```

Per-command extensions add command-specific state (e.g., `tasks_completed`, `chunk_complete`, `verdict`, `decisions_captured`, `prompt_path`). Each command's spec defines its extensions.

**Why this spine** (vs minimal `command + next_recommended`): matches the de facto spine already used by `review-*.json`, `address-reviews-*.json`, `archive-*.json` — backward-compatible with orbit-status's existing tier-1 reader. The `kind` field is structurally new; `final_assessment` and `next_recommended` codify existing de facto fields (present in actual emits but previously unspecified in orbit-conventions) into formal spec. This change modifies `orbit-conventions/spec.md`'s `Internal-run JSON summary format` requirement to unify the spine across all kinds (workflow / editorial / lifecycle), with per-kind extensions. The 4 existing per-skill schema reference files (`.claude/skills/openspec-<skill>/references/run-summary-schema.md` for review/audit-drift/address-reviews, and `archive-summary-schema.md` for openspec-archive-change) inherit the universal spine from orbit-conventions and continue to document per-command extensions.

**Alternative considered (rejected):** structured `next_recommended` (object with `command`, `args`, `reason`). Rejected because orbit-status today best-effort-parses a verbatim string (`orbit-status-recommendation/spec.md:7`); producer-side structuring is redundant and loses prose nuance for ambiguous recommendations (e.g., review-external's multi-step instructions).

### D-spine-2: `kind` taxonomy of 3 values

- `kind: "workflow"` — forward-progressing commands: `explore`, `propose`, `new`, `continue`, `ff`, `apply`, `verify`
- `kind: "editorial"` — evaluative or resolution-focused: `review`, `address-reviews`, `audit-drift`, `review-external`
- `kind: "lifecycle"` — terminal transitions: `archive`

Maps to issue [#8](https://github.com/las-sal/openspec-orbit/issues/8)'s own "workflow commands vs editorial commands" framing, with `archive` broken into its own category because it transitions the change out of the active set rather than progressing or evaluating it.

### D-emit-1: Per-variant filenames preserve entry-point provenance

`/opsx:new`, `/opsx:continue`, `/opsx:ff` are propose-shaped (produce the canonical artifact set), but each emits with its own command-name prefix: `new-<TS>.json`, `continue-<TS>.json`, `ff-<TS>.json`. Same shape, distinct origins.

**Why this matters:** orbit-status sorts and groups `.orbit-runs/` entries by filename prefix; per-variant filenames preserve provenance for free without inspecting JSON bodies.

### D-emit-2: Bare-mode `/opsx:explore` does NOT emit; crystallization requires explicit user warning

Bare-mode explore (no name argument) is pre-commitment, ephemeral thinking. No JSON is written. When the user requests crystallization (asks for a name and explore.md creation), the AI MUST surface an explicit warning describing the persistence consequences (the new `openspec/explore/<name>/` directory, the start of the audit trail, visibility to orbit-status/`openspec list`) and confirm before proceeding.

**Why warn rather than just emit:** prevents accidentally seeding a partial change the user didn't intend to start. Aligns with [#7](https://github.com/las-sal/openspec-orbit/issues/7) (don't auto-invoke) and [#15](https://github.com/las-sal/openspec-orbit/issues/15) (inflection points surface consequences).

**Alternatives considered (both rejected):** (a) emit to a project-wide `openspec/.orbit-runs/explore-<TS>.json` for "user is exploring something unnamed" visibility — rejected as polluting project scope with pre-commitment thinking. (b) per-session UUID directory — rejected as adding directory ceremony without commensurate value.

### D-emit-3: `/opsx:apply` emits per chunk-end (and on mid-chunk pause), not per session

Apply emits at chunk boundaries when the apply was structured with explicit chunks (as is canonical orbit practice for changes with >3 task groups). Rules:

1. **Chunk completion → emit** with `chunk: "N of M"`, `chunk_complete: true`. `next_recommended` advances to next chunk or `/opsx:verify` on apply-complete.
2. **Mid-chunk session pause → emit** with `chunk_complete: false`. Preserves resumability across context switches.
3. **No-chunking apply → single emit at session end** with `chunk: null`.

**Forensic justification:** orbit-status's apply (76 tasks, 5 chunks, 42 minutes) had no intra-apply emits, so any post-hoc regression bisection would have had to span the full 76-task window. Per-chunk emit bounds the blast radius to a single chunk's task set — typically a 10-20× reduction in suspect surface.

### D-rec-1: Per-command recommendation rules

Each command's `next_recommended` follows documented rules. Codified in the spec; summarized here:

| Command | State | Recommendation |
|---|---|---|
| `explore` (named) | 0–1 decisions | `/opsx:explore <name>` — continue thinking |
| `explore` (named) | 2–3 decisions | `/opsx:explore <name>` — continue, or `/opsx:propose <name>` if ready |
| `explore` (named) | 4+ decisions, ≤1 open Q | `/opsx:propose <name>` — substantial thinking captured |
| `propose`/`ff` | always | `/opsx:review <name>` (per [#9](https://github.com/las-sal/openspec-orbit/issues/9)) |
| `new` (scaffold-only) | always after fresh scaffold | `/opsx:continue <name>` (artifact-completion-aware; isComplete=false) |
| `continue` | artifacts incomplete (isComplete=false) | `/opsx:continue <name>` — next missing artifact |
| `continue` / `new` | artifacts complete (isComplete=true) | `/opsx:review <name>` |
| `apply` | chunk N done, more chunks | `/opsx:apply <name>` — next chunk |
| `apply` | all chunks done | `/opsx:verify <name>` |
| `verify` | pass | `/opsx:review --as system <name>` (with `/opsx:archive` in reason) |
| `verify` | fail mode ① tasks incomplete | `/opsx:apply <name>` |
| `verify` | fail mode ② impl-vs-spec gap | `/opsx:review --as system <name>` (without `--mark` — that flag is proposal-mode only; user walks system review findings manually) |
| `verify` | fail mode ③ openspec-validate fail | verbatim validator message |
| `verify` | warn | `/opsx:review --as system <name>` |
| `review-external` | T0 (prompt packaged) | multi-step prose: paste, save, `/opsx:address-reviews --from-file` |
| `audit-drift` (standalone) | findings | `/opsx:address-reviews <name> --from-file <this>` |
| `audit-drift` (standalone) | clean | copy `next_recommended` from prior latest JSON (preserve workflow narrative) |

Verify's fail-mode ② recommends `/opsx:review --as system <name>` (without `--mark` — that flag is proposal-mode only per `orbit-review/spec.md`'s `Requirement: --mark flag is proposal-mode only`). The user walks the system review's findings manually to triage each as code-vs-spec. (Earlier draft recommended `--as system --mark` to drop markers for address-reviews; corrected during external review iter-1 EW3 fix once it was verified that `--mark` is silently ignored in system mode — system-mode marker writing is v2 work tracked at the relevant follow-up.) Verify itself stays upstream-unchanged per D-arch-1 in both cases.

### D-rec-2: Recommendations don't forward-look at other issues' future behavior changes

When a recommendation points at a command whose default behavior is expected to change (e.g., `/opsx:review --as system`, which [#20](https://github.com/las-sal/openspec-orbit/issues/20) will invert to default-external), the emit text does NOT preview or anticipate the future-issue behavior. It just recommends the canonical command; when the future issue lands, every prior recommendation automatically benefits without #8's emit needing to update.

**Why:** clean responsibility split (#8 = "right next command"; #20 = "smarter default in that command"). No forward-reference rot if a future design shifts before landing.

## Risks / Trade-offs

- **Chunk-end signal detection is heuristic.** The AI must recognize when a chunk has completed (last task in chunk N checked, plus any chunk-specific exit criteria). [Mitigation] Orbit's canonical practice already names chunks explicitly in `tasks.md` preambles; emit-layer reads that preamble for chunk identity and counts. Falls back to single-emit on apply-complete when no chunks are declared.
- **Filename timestamp ordering is load-bearing for orbit-status.** Filenames embed an ISO-8601 timestamp suffix that orbit-status parses for "most recent JSON" ordering. [Mitigation] Strict format enforcement in emit-layer (`YYYY-MM-DDTHH-MM-SSZ`); regression test in orbit-status's test suite.
- **Bare-mode crystallization warning depends on AI behavior.** The AI must surface the warning, not silently transition. [Mitigation] Warning text codified verbatim in `openspec-explore` SKILL.md additions; non-compliance becomes a review-time finding under [#7](https://github.com/las-sal/openspec-orbit/issues/7) (don't auto-invoke).
- **Per-skill schema reference docs need parallel updates to acknowledge the unified spine.** The 4 reference files (`run-summary-schema.md` for review/audit-drift/address-reviews + `archive-summary-schema.md` for openspec-archive-change) document per-skill emit shapes today; they need brief edits to point at orbit-conventions for the universal spine and retain per-command extension documentation. [Mitigation] Tracked as task 1.2; doc-only changes (no behavior change). Tier-1 in orbit-status works either way because it reads the spine fields opportunistically — out-of-sync reference docs are a code-comment-style hazard, not a runtime hazard.
- **Mid-apply chunk-pause heuristic could over-emit.** If "session pause" is detected aggressively, apply could emit on every conversation turn boundary. [Mitigation] Pause = AI returns control to user without a `chunk_complete: true` emit in the same turn; emit only fires on explicit "stopping here for now" signals or extended idle (orbit-status doesn't read these mid-chunk emits as "advance the workflow" — they're informational for resumability).

## Migration Plan

Single-cycle rollout — no phased migration needed. Steps:

1. Land this change (orbit's emit behavior for the 9 new commands)
   - Includes modifying `orbit-conventions/spec.md`'s `Internal-run JSON summary format` requirement to define the universal spine + per-kind extensions (the `orbit-conventions` MODIFIED delta in this change)
   - Includes updating the 4 existing per-skill schema reference files (`.claude/skills/openspec-{review,audit-drift,address-reviews}/references/run-summary-schema.md` + `.claude/skills/openspec-archive-change/references/archive-summary-schema.md`) to acknowledge inheritance from orbit-conventions's universal spine — doc-only updates, no behavior change (tracked in tasks.md group 1.2)
2. orbit-status's existing tier-1 reader picks up the new JSONs immediately, with no orbit-status change required
3. After landing, orbit-status's tier-2 cleanup ([orbit-status#2](https://github.com/las-sal/orbit-status/issues/2)) ships as a follow-up — deletes the now-dead synthesis layer
4. Forward-looking issues ([#15](https://github.com/las-sal/openspec-orbit/issues/15), [#16](https://github.com/las-sal/openspec-orbit/issues/16), [#22](https://github.com/las-sal/openspec-orbit/issues/22)) that depend on these JSONs become straightforwardly implementable

**Rollback strategy:** revert the SKILL.md changes; existing emitting commands continue unchanged. orbit-status's tier-1 reader gracefully degrades (no JSON found → tier-2 synthesis fires per its existing logic), so rollback is non-destructive.

## Open Questions

None. The one verification question raised during explore — "is orbit-status tier-2 purely synthesis-output, safe to delete?" — was resolved with a code-level scan of `/Users/sal/code/orbit-status/.claude/skills/openspec-status/bin/opsx-status` confirming `source_label` is set-and-emit only, never branched on (see `explore.md` D15).
