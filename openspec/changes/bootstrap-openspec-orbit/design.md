# Design: bootstrap-openspec-orbit

## Context

The originator (Sal) has run OpenSpec across multiple projects (`home-control`, `home-env`, `home-control-test`) and developed a repeatable review-and-capture workflow during that work. The pattern has been validated through ~80 review prompts across project transcripts and codified into two specific lessons (captured in a local `OPENSPEC_LESSONS.md` reference file).

The workflow exists today as discipline applied by the user during conversations with AI. Each session, the marker convention, pushback rules, capture habits, and external-review handoff format must be re-explained. The behaviors that consistently catch real issues (vocabulary residue across non-delta'd docs, stale findings causing churn, cross-AI cycle for second-pair-of-eyes) are exactly the ones that ad-hoc re-explanation loses fidelity on.

orbit packages this workflow as a `.claude/` overlay on upstream `@fission-ai/openspec`. Upstream provides the change-driven scaffold (explore → propose → apply → archive); orbit adds editorial review and resolution layers around it. The user explicitly does not intend to upstream orbit — it stays as a downstream opinionated companion.

The full exploration record that produced this change lives at `openspec/changes/bootstrap-openspec-orbit/explore.md` and `openspec/changes/bootstrap-openspec-orbit/sketches/` (moved into the change directory by `/opsx:propose`'s consume mode; `openspec/explore/<name>/` was the pre-propose staging location). The exploration record is the authoritative design source; this design document summarizes the key technical decisions for implementers.

## Goals / Non-Goals

**Goals:**

- Ship a `.claude/` overlay that adds four new opsx commands and modifies three existing ones, without forking the upstream CLI.
- Adopt upstream conventions wherever possible (vocabulary, file layout, reporting shape, CLI usage as source of truth). The acid test: a merge of orbit into upstream should read like a contributor's PR.
- Make the review-and-capture workflow durable across AI sessions: marker convention (`@review:`), pushback discipline, marker-removal invariant, cross-AI handoff format, and exploration capture.
- Compose with upstream `verify-change` (system-mode review wraps it as Pass 0) and `sync-specs` (audit-drift complements its delta-only coverage).
- Make the cross-AI review cycle frictionless: `/opsx:review-external` packages the handoff, `/opsx:address-reviews --from-file` ingests the findings; no copy-paste-per-finding.
- Preserve graceful degradation: empty `openspec/lenses/` → system-mode Passes 4/5 skip; missing `explore.md` → propose runs in standalone mode; absent prior `.orbit-runs/` → first-run path.

**Non-Goals:**

- Forking or modifying the `@fission-ai/openspec` CLI binary. All orbit behavior lives in markdown prompts under `.claude/`.
- Building a Claude Code plugin in v1 (deferred to Phase 2 once orbit stabilizes).
- Implementing `/opsx:distill-specs` (canonical-spec hygiene) — deferred to v2 with scope notes in `explore.md`.
- Caching pass results — deferred to v2 (issue #1).
- Defining `--thorough` mode extras precisely — deferred to v2 (issue #2).
- Comprehensive `/opsx:address-reviews` features beyond lean v1 — deferred to v2 (issue #3); v1 ships only the four enforcement wins plus `--from-file`.
- Source-code marker writing via `--mark` on `/opsx:review --as system` — deferred to v2 (issue #3).
- Auto-cascade for ripple resolution in address-reviews — v1 only flags affected files (deferred to v2).
- Backwards-compatibility shims for users without upstream openspec installed — orbit requires `openspec init` to have run first.

## Decisions

### D1. Distribution model: `.claude/` overlay, not a CLI fork

orbit ships markdown prompts (slash command bodies and SKILL.md files) that consumers drop into their project's `.claude/` directory. The upstream CLI binary is unchanged.

**Rationale:** orbit's value is in the AI-facing prompts, not in the structural CLI work that openspec already does well. Forking the CLI would inherit upstream's build/test/release infra and force orbit to track upstream forever. An overlay is the right artifact for what orbit ships.

**Alternative rejected:** clone `Fission-AI/OpenSpec`, modify `src/core/templates/workflows/*.ts`, publish as a competing npm package. Heavyweight for markdown-only changes; rejected in `explore.md`.

### D2. Naming taxonomy: `verify-*` / `review-*` / `audit-*` / `distill-*`

Four verb prefixes, each with a distinct meaning so adopters can predict where new commands go.

| Verb | Meaning | Examples |
|---|---|---|
| `verify-*` (upstream) | structural correctness checks at a defined scope | `verify-change` |
| `review-*` (orbit) | opinionated editorial passes layered on top | `review` (with `--as proposal\|system`), `review-external` |
| `audit-*` (orbit) | scan for drift / residue / staleness | `audit-drift` |
| `distill-*` (orbit, v2) | reduce to essential | `distill-specs` |

**Rationale:** The `review-` prefix on orbit's editorial commands signals "add-on for specific purposes, not a replacement for `verify-change`." Users mentally distinguish the structural (upstream) layer from the editorial (orbit) layer.

**Alternative rejected:** symmetric `/opsx:verify-proposal` + `/opsx:verify-system` — would force-fit "verify" onto operations that are really editorial review. Verify implies a ground-truth target, which a proposal doesn't have.

### D3. `/opsx:review --as system` wraps `verify-change` rather than replacing it

Pass 0 of `/opsx:review --as system` delegates to upstream `verify-change`. Passes 1–6 add system-wide checks on top.

**Rationale:** verify-change is rigorous within its scope (the change's deltas). orbit inherits that rigor for free and layers what verify-change doesn't do (baseline compliance, cohesion, surfaces, perspectives, critical paths, drift). Future upstream improvements to verify-change flow through automatically.

**Alternative rejected:** override `verify-change` SKILL.md entirely. Would force orbit to re-implement structural checks already done well upstream, and to track every upstream change to that command.

### D4. Reporting convention shared across orbit + upstream

All `review-*`, `audit-*`, and `verify-*` commands use the same output shape:

- **3-dimension summary scorecard**: Completeness / Correctness / Coherence (matches `verify-change`)
- **Severity ladder**: CRITICAL → WARNING → SUGGESTION (matches `verify-change`)
- **Per-finding format**: file:line + specific recommendation (matches `verify-change`)
- **Stock final-assessment phrasings**: "X critical issue(s) found. Fix before \<gate\>." / "No critical issues. Y warning(s) to consider. Ready \<gate\> (with noted improvements)." / "All checks passed. Ready \<gate\>."

**Rationale:** Adopters learn one report shape across upstream's structural verifier and orbit's editorial reviewers. Lower cognitive overhead.

**`/opsx:address-reviews` is the exception:** its operation is action (resolve markers), not scan. Output is a **resolution log** (✓ resolved / ⚠ stale / ⏸ deferred / ✗ escalated), not a scorecard. Different operation, different report shape.

### D5. Marker convention: `@review: <content>` across all file types

Single inline-review marker. Markdown carries it bare; source code and configs carry it inside the file type's comment syntax.

```
spec.md:    Each device exposes a state. @review: forbid sensitive ones?
foo.ts:     // @review: handles expired tokens?
config.yaml:  token_ttl: 3600  # @review: configurable per env?
```

**Rationale:** Simpler than `<!-- REVIEW: -->`, distinctive in prose (low false-positive grep), uniform across file types so address-reviews uses one grep pattern. The `@` prefix is uncommon enough at the start of content to be reliably greppable.

**Alternative rejected:** `<!-- REVIEW: -->` (HTML comment) — invisible in rendered markdown but verbose to type, hard to remember, HTML-ish in non-HTML files.

### D6. Lean v1 for `/opsx:address-reviews`

v1 ships marker scan + walk + pushback + remove + `--from-file` ingest. Deferred to v2: `--from-paste`, automatic ripple cascade, severity filtering in the resolution log, `--strict`, `--parallel`, categorized markers (`@review(blocker):`), source-code `--mark` writing, auto re-run.

**Rationale:** Four enforcement wins (convention durability, pushback discipline, marker removal invariant, multi-file-type uniformity) plus `--from-file` (needed for cross-AI loop) are the v1 must-haves. Comprehensive features iterate on a proven base.

### D7. `openspec/lenses/` as the judgment layer

`openspec/lenses/perspectives.md` and `openspec/lenses/critical-paths.md` are the durable home for subjective judgments code can't make. Captured during `/opsx:explore` via offer-don't-auto triggers. Consumed by system-mode Passes 4 (perspectives) and 5 (critical paths).

**Rationale:** Multiple commands need to know "which callers matter" and "which user flows are critical." Code can't tell you that; it's human judgment. Without a durable home, this knowledge stays in the user's head and is re-derived per-review. The `lenses/` directory makes it persistent and team-visible.

**Surfaces are NOT in lenses/** — capabilities in `openspec/specs/<capability>/` ARE the surfaces. Re-declaring them would duplicate and drift.

**Alternative rejected:** YAML format — easier to parse but harder to evolve as prose grows. Openspec's pattern: YAML for config, markdown for content-the-AI-reads. lenses content is content.

**Alternative rejected:** baking judgment-layer content into `CLAUDE.md` — different audience (handoff orientation), different lifecycle (rare updates).

### D8. `openspec/explore/<name>/` as staging; `/opsx:propose` promotes to changes

Pre-propose, exploration material lives at `openspec/explore/<name>/`. When `/opsx:propose <name>` runs, it consumes `explore.md`, generates artifacts, and *moves* the staging directory to `openspec/changes/<name>/`. The `explore.md` and any sibling captures persist alongside the generated artifacts as historical record.

**Rationale:** Pre-propose, the change directory doesn't exist (creating it is `propose`'s job in the existing flow). Staging in `openspec/explore/<name>/` respects upstream's flow while keeping exploration durable. Moving (not copying) avoids two-source-of-truth confusion.

**Alternative rejected:** put `explore.md` in `openspec/changes/<name>/` from the start — preempts propose's responsibility, doesn't fit upstream's flow.

### D9. `.orbit-runs/` for per-change iteration persistence

`openspec/changes/<name>/.orbit-runs/` holds JSON internal-run summaries (one per review/audit/archive invocation) and markdown external-review findings (one per `/opsx:review-external` invocation). Committed, dot-prefixed.

**Rationale:** Enables convergence tracking (iteration note in review reports), preserves cross-AI cycle history, supports team handoffs. Dot-prefix signals "orbit metadata, not part of canonical openspec change." Committed because the history is real evidence of the review cycle.

The directory moves with the change into `openspec/changes/archive/<name>/.orbit-runs/` during archive.

### D10. `--parallel` opt-in in v1; subagents for context partitioning, not just speed

Heavy system-mode review passes (cohesion, perspectives, critical paths) can spawn subagents for concurrent execution. Opt-in via `--parallel` flag. Default sequential.

**Rationale:** On real-sized codebases, single-context review hits ceiling. Parallel subagents partition context, not just speed up wall-clock. Per guiding principle 2 (cost up front trumps downstream cost), the higher per-run token cost is worth the better signal and ability to scale to larger projects.

**Why opt-in:** sequential default is easier to debug, deterministic, predictable. Adopters can graduate to `--parallel` when needed.

### D11. `/opsx:archive` auto-invokes `/opsx:audit-drift`, but as a prompt-not-gate

Pre-archive sweep catches the OPENSPEC_LESSONS doc-residue failure mode. Critical drift findings trigger a three-way prompt (address now / proceed / abort) — not a hard gate. Opt-out via `--skip-audit`.

**Rationale:** Users may legitimately archive with known drift (e.g., follow-up commit planned). orbit captures the decision in the archive run summary (`.orbit-runs/archive-<TS>.json`) for traceability, but doesn't override the user.

### D12. Cross-AI external review via `/opsx:review-external` + `/opsx:address-reviews --from-file`

External review packaging is its own command (`/opsx:review-external <change> [--as proposal|system]`). It writes the self-contained prompt to a committed file at `.orbit-runs/external-prompt-<as>-<TS>.md` and emits only a short invocation snippet to chat (file path + paste-ready instruction for the external AI + the eventual findings path). External AI pulls the repo, reads the prompt file, writes findings to `.orbit-runs/external-<as>-<TS>.md`, and (if it has git access) commits and pushes. Ingest via `/opsx:address-reviews --from-file <path>`.

**Rationale:** The cross-AI cycle was already working ad-hoc; the only friction was copy-paste-per-finding. Defining the prompt + findings file format closes the loop. `--from-file` is in v1 (revising earlier v2 deferral) because the user's workflow requires it.

**`--as` mode inference:** if not specified, infer from `tasks.md` state — unchecked → `proposal`, all checked + code → `system`. User can override with explicit flag.

### D13. Three cross-cutting execution disciplines bracket the authoring lifecycle

orbit codifies three execution disciplines as cross-cutting requirements in `orbit-conventions`, one per phase of authoring work:

- **Read-before-reference (authoring-time)**: read the actual definition of any specific named construct (function signature, type/interface, object shape, file path, spec requirement name) before generating code/tests/specs that reference it. Inference from training-data patterns is NOT a substitute for reading.
- **Change completeness (modification-time)**: substantive modifications to a change-in-flight must be applied fully across all affected artifacts before declared done. Residue cleanup is not optional and is not deferred to review.
- **Pushback (review-time)**: verify findings against current state before fixing; stale findings get suppressed with evidence; don't re-edit already-fixed state.

**Rationale:** each prevents a specific AI failure mode the spec-driven workflow alone doesn't catch (assumption-based authoring; sed-residue creep; stale-finding churn). Together they bracket the authoring lifecycle and compose with the per-command behavior the SKILL.md files implement.

**Implementer note:** all three disciplines MUST be embedded in each command's SKILL.md content. The `orbit-conventions` requirements are the normative contract; the SKILL.md content is the implementation. Intentional text duplication across SKILL.md files is acceptable for self-contained reliability (same trade-off as the original pushback-discipline decision).

**Adopter note:** orbit's README ships a recommended `CLAUDE.md` snippet bundling all three disciplines for project-level reinforcement. Behavior of orbit's commands does not depend on adopters using the snippet (the disciplines are self-contained in each SKILL.md), but project-level reinforcement helps non-orbit-command AI work in the project too.

**Alternative rejected:** Embed disciplines only in `CLAUDE.md` template, not in each SKILL.md. Rejected because CLAUDE.md isn't always loaded into context, and the disciplines must be reliable when commands run. Intentional duplication is the price; reliability is the payoff.

## Risks / Trade-offs

- **Text duplication across SKILL.md files for pushback discipline** → mitigation: documented as an intentional choice (per guiding principle 2 and explicit decision); CLAUDE.md snippet optional but not required for behavior.
- **No automatic ripple cascade in lean v1 of `/opsx:address-reviews`** → mitigation: ripple "flag" lists affected files; user resolves manually or re-runs the command after fixing. Auto-cascade is v2 (issue #3).
- **Lenses content can go stale relative to actual code** → mitigation: `/opsx:audit-drift` Category 2 (Lens Staleness) detects this; system-mode Pass 6 invokes audit-drift; pre-archive auto-invocation catches it before each archive.
- **External AI must understand the prompt and write the file** → mitigation: prompt is self-contained; format is rigid; if external AI has no file-write capability, prompt instructs it to output markdown for user to save manually.
- **`.orbit-runs/` clutter as iterations accumulate** → mitigation: each file is small (JSON summaries or short markdown); no automatic cleanup in v1; users can manually prune if needed. Files persist with the archive for full traceability.
- **8 capability specs is more files than the typical change** (down from 9 after the `orbit-review-proposal` + `orbit-review-system` merge in iter 4) → mitigation: each spec is small and focused, enabling clean per-command deltas in future changes. Larger up-front cost; smaller per-delta cost over orbit's lifecycle.
- **Adopters who don't run `openspec init` first see broken state** → mitigation: README explicitly documents the prerequisite; install instructions assume upstream is set up.
- **`/opsx:review-external --as` inference can mismatch** → mitigation: inferred mode is always shown in output; user can override with explicit flag.
- **Pre-archive audit prompt friction** → mitigation: `--skip-audit` flag available; prompt only fires on CRITICAL findings, not on every archive.

## Migration Plan

Not applicable for v1. This is a new overlay; there is no pre-existing orbit installation to migrate from. Future versions will add a `Migration` section if any breaking changes ship.

## Open Questions

All v1-blocking open questions are resolved during exploration. Remaining items are tracked outside this change:

- **`--thorough` extras** for review commands — defined in GitHub issue #2.
- **Pass-result caching** — GitHub issue #1.
- **Comprehensive `/opsx:address-reviews` features** (paste, cascade, severity, etc.) — GitHub issue #3.
- **`/opsx:distill-specs` design** — v2 scope notes captured in `explore.md`; not in this change.
- **Abandoned exploration cleanup** — v1 leaves user-managed; auto-archive deferred until clutter is a real problem.
- **`/opsx:explore` capture heuristics tuning** — to be calibrated from real-use observation after v1 ships.
