---
name: openspec-onboard
description: Orbit workflow reference walkthrough — setup verification, identity statement, 9-phase canonical-flow walkthrough, quick-reference table, try-it nudge. Reference-leaning hybrid for AI cold-load + future-self + collaborators.
license: MIT
compatibility: Requires openspec CLI at the pinned version per `orbit-conventions` `Upstream version pinning` (currently `@fission-ai/openspec@1.3.1`).
metadata:
  author: orbit
  version: "0.1.0"
  generatedBy: "1.3.1"
---

> **Note on `/opsx:onboard` emit semantics**: this command does NOT emit a run-summary JSON when invoked (composes with `orbit-run-summary-emit` `Emit scope`, which lists `/opsx:onboard` as a non-emit command). `/opsx:onboard` is a single-pass reference read — no workflow state advances, no artifacts produced in user project. Future orbit-additions or refactors MUST preserve this non-emission discipline.

This skill is the orbit-authored onboarding read for the `/opsx:onboard` slash command. Single-pass; renders Sections 1-5 sequentially. Section 1 (Setup verification) gates the rest — on hard-stop, Sections 2-5 do not render.

This SKILL.md body and `.claude/commands/opsx/onboard.md` are duplicates by current convention (only frontmatter differs); when revising one, update the other in the same edit. Cross-command body deduplication is tracked for systematic cleanup as [openspec-orbit#29](https://github.com/las-sal/openspec-orbit/issues/29).

---

## Section 1: Setup verification

Run this section first. It checks that the user's project has orbit's overlay applied against the pinned upstream version. The check has three outcomes:

- **Pass** → emit a layered ✓ output describing what's present, then proceed to Section 2.
- **Warn-continue** → the prune step (`rm -rf .claude/skills/feedback`) was not run; emit `⚠` with remediation and proceed to Section 2.
- **Hard-stop** → orbit's overlay is incomplete OR the upstream version doesn't match the pin; emit the lumped remediation message and STOP. Do not render Sections 2-5.

### Skill checks (in order)

a. **Upstream version pin match**: run `openspec --version`. Expected: `1.3.1` (per `orbit-conventions` `Upstream version pinning`). Mismatch → hard-stop.

b. **Orbit-authored skill presence**: each of these MUST exist:
   - `.claude/skills/openspec-review/SKILL.md`
   - `.claude/skills/openspec-audit-drift/SKILL.md`
   - `.claude/skills/openspec-address-reviews/SKILL.md`
   - `.claude/skills/openspec-review-external/SKILL.md`
   - `.claude/skills/openspec-onboard/SKILL.md` (this file)

   Any missing → hard-stop.

c. **Upstream-required primitive presence**: `.claude/skills/openspec-sync-specs/` MUST exist (orbit's archive flow uses it as a callable primitive). Missing → hard-stop.

d. **Orbit-modified skill count**: `grep -l "^# Orbit additions" .claude/skills/openspec-*/SKILL.md | wc -l` MUST equal `9` (post-this-change baseline — the 10 originally-modified skills minus `openspec-onboard`, which is now orbit-authored). Mismatch → hard-stop.

e. **Pruned-skill absence**: `.claude/skills/feedback/` MUST NOT exist. Present → warn-continue (NOT hard-stop) with remediation `rm -rf .claude/skills/feedback`.

### Command checks (in order)

f. **Orbit-authored command presence**: each of these MUST exist:
   - `.claude/commands/opsx/review.md`
   - `.claude/commands/opsx/review-external.md`
   - `.claude/commands/opsx/audit-drift.md`
   - `.claude/commands/opsx/address-reviews.md`
   - `.claude/commands/opsx/onboard.md`

   Any missing → hard-stop.

g. **Orbit-shipped sync command presence**: `.claude/commands/opsx/sync.md` MUST exist (orbit ships sync.md alongside the openspec-sync-specs primitive). Missing → hard-stop.

h. **Total command count**: `ls .claude/commands/opsx/ | wc -l` MUST equal `15` (post-this-change baseline — 14 from orbit's overlay + 1 upstream-untouched `ff.md`). Mismatch (≠ 15) → hard-stop.

### Layered ✓ output on pass

```
✓ openspec CLI @ 1.3.1 (pin match)
✓ 9 upstream-modified skills present (`# Orbit additions` count = 9)
✓ 5 orbit-authored skills present (openspec-review, openspec-review-external,
   openspec-audit-drift, openspec-address-reviews, openspec-onboard)
✓ 1 upstream-required primitive present (openspec-sync-specs)
✓ feedback/ absent (prune step verified)
✓ 15 commands present (14 orbit-shipped + 1 upstream-untouched ff.md)
→ Proceeding to canonical-flow walkthrough.
```

The 5 ✓ skill/feedback lines mirror the four `orbit-conventions` `Overlay file disposition` categories (orbit-authored / orbit-modified / upstream-required primitive / not-shipped). The version-pin and command-count lines are additional checks beyond the category framework.

### Hard-stop output (lumped per `Lumped messaging for overlay-incomplete sub-modes`)

When any hard-stop check (a, b, c, d, f, g, h) fails, emit ONE message regardless of which sub-mode triggered:

```
✗ Setup verification failed.

Your orbit installation is incomplete or running against the wrong upstream
version. See the README's `## Installation` section as a whole — both
`### 1. Initialize upstream OpenSpec` (for version-pin issues) and
`### 2. Overlay orbit` (for overlay-incomplete issues).

After fixing the install, re-run /opsx:onboard to verify.
```

This lumping is intentional: remediation requires re-reading install steps from the top regardless of which check failed; distinguishing sub-modes would multiply prose without changing the user's action.

### Warn-and-continue output

When check (e) fails but all hard-stop checks pass:

```
⚠ feedback/ skill present in .claude/skills/feedback/ — orbit no longer
ships this skill. Run `rm -rf .claude/skills/feedback` to align with the
current overlay disposition. (Documented in README install + update +
uninstall sections.)

Continuing to canonical-flow walkthrough.
```

### Gotcha not detected by verification

Setup verification reads files on disk; it cannot detect a stale AI client cache. If your AI client cached an older `.claude/` snapshot before you ran the install, verification passes but the client may still see the old surface. Remediation: restart your AI client after install.

---

## Section 2: Identity statement

Orbit is a workflow tool that owns the `.claude/` surface (skills, commands, supporting docs) and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine. The upstream CLI binary is unchanged and version-pegged; orbit's contribution lives in markdown content under `.claude/`. Orbit is its own thing — it shares concepts with upstream OpenSpec but ships an opinionated workflow tool with distinct disciplines, not a delta on top of upstream's defaults.

What makes orbit distinctive vs running upstream alone:

- **Editorial review** — `/opsx:review` runs a 9-pass editorial pass over the artifacts (proposal mode) or a 7-pass pass over the post-apply product state (system mode, which wraps upstream's `verify-change` as Pass 0). `/opsx:review-external` packages a self-contained prompt for a different AI (codex / fresh Claude / GPT) to do a second-opinion pass; the external AI commits findings back to the repo. `/opsx:address-reviews` walks each finding through a structured pushback → classify → fix → ripple-flag → remove lifecycle (works on both inline `@review:` markers and external-review findings files).

- **Drift audit** — `/opsx:audit-drift` scans for drift between captured knowledge (specs, lenses, governing docs) and reality across four categories: vocabulary residue (FROM names of renamed requirements still appearing), lens staleness (perspectives or critical-paths referencing nonexistent capabilities), cross-doc consistency (CLAUDE.md / project.md disagreement with current specs), archive coherence (ADDED requirements missing from baseline, RENAMED FROM names lingering). Standalone for "something feels off" checks; auto-invoked as system-mode review Pass 6 and as a pre-archive sweep.

- **Capture (lenses)** — `openspec/lenses/perspectives.md` records named callers worth simulating from during review; `openspec/lenses/critical-paths.md` records flows worth walking end-to-end. Lenses grow organically through `/opsx:explore` capture triggers; system-mode review Passes 4-5 consult them.

- **JSON run-summary emission** — every editorial and lifecycle command emits a JSON run-summary at command completion (or per chunk for `/opsx:apply`). Summaries persist to `openspec/changes/<name>/.orbit-runs/` (per-change) or `openspec/.orbit-runs/` (project-wide). The schema follows a universal spine with per-command extensions. (Note: `/opsx:onboard` is one of the few commands that does NOT emit — it's a reference read, not a workflow advancement.)

- **Three execution disciplines** codified in `orbit-conventions`:
  - **Read-before-reference** (authoring-time): when generating code, specs, or docs that name a specific construct, read the actual definition first. Don't infer the shape from common patterns or training-data conventions.
  - **Change completeness** (modification-time): substantive modifications to a change-in-flight must apply fully across ALL affected artifacts before being declared done. Known residue MUST NOT be left for a downstream review to catch.
  - **Pushback** (review-time): verify each flagged issue against current state before acting. Stale findings get reported with evidence and suppressed.

---

## Section 3: Canonical-flow walkthrough

Orbit's canonical flow has 9 phases. Each maps to a `/opsx:*` slash command. The flow is:

```
   explore ─→ propose ─→ review ─→ address-reviews ─→ apply ─→ verify ─→
   review --as system ─→ address-reviews ─→ archive
```

**Phase 1 — explore (`/opsx:explore [<name>]`)**: Thinking-mode entry. The user explores an idea, problem, or comparison; orbit acts as thinking partner. In named mode (`/opsx:explore <name>`), decisions auto-capture to `openspec/explore/<name>/explore.md` in a 5-section convention (Premise / Decisions / Open questions / Considered & out / References). In bare mode (no name), the explore stays in conversation until 2+ substantive decisions emerge, then prompts to crystallize a name. Capture triggers also offer to save **lenses** — `perspectives.md` records named callers worth validating from during review (e.g., "API client", "Claude Desktop using MCP"); `critical-paths.md` records flows worth walking end-to-end (e.g., "First-time user adds their first device"). Lenses grow organically; system-mode review Passes 4-5 consult them.

**Phase 2 — propose (`/opsx:propose <name>`)**: Generates the change artifacts (proposal, design, specs, tasks) from the explore.md (consume mode) or from a prompted description (standalone mode). Consume mode treats `explore.md` as authoritative seed: Premise → proposal motivation; Decisions → spec deltas + design + tasks; Considered & out → design's "Alternatives considered"; References → contextual reads. Open questions become per-question prompts during propose with three resolution paths (resolve now → Decision; defer → `@review:` marker in artifacts; abandon → moves to Considered & out). After artifact generation, the staging directory at `openspec/explore/<name>/` moves to `openspec/changes/<name>/` (the historical `explore.md` persists alongside the new artifacts). Emits a T0 run summary to `openspec/changes/<name>/.orbit-runs/propose-<TS>.json`.

**Phase 3 — review (proposal mode, `/opsx:review <name> --as proposal`)**: Editorial pre-apply pass over the artifacts. 9 passes: Structure & Delta Integrity (validate runs clean, deltas use ADDED/MODIFIED/REMOVED/RENAMED correctly) / Internal Coherence (proposal aligns with design aligns with specs aligns with tasks) / Cross-Doc Coherence (CLAUDE.md and project.md remain accurate after this change) / Archive Consistency (ADDED don't contradict baseline, RENAMED FROM names exist) / Codegen Readiness (no implicit requirements, no ambiguity) / Gap Hunt (could a fresh AI implement this from these specs alone?) / Drift Hunt (old vocabulary lingering) / Inline Review Marker Residue (`@review:` markers must be 0) / Pre-Handoff Sweep. Findings roll into a 3-dimension scorecard (Completeness / Correctness / Coherence) with CRITICAL / WARNING / SUGGESTION severities. The final-assessment line uses stock phrasings ("All checks passed. Ready to apply." vs "X critical issue(s) found. Fix before /opsx:apply."). Run summary persists to `openspec/changes/<name>/.orbit-runs/review-proposal-<TS>.json`.

**Phase 4 — address-reviews (`/opsx:address-reviews <name>` or `--from-file <path>`)**: Walks each finding (or each `@review:` marker) through a structured lifecycle: pushback → classify → fix → ripple-flag → remove-marker. Pushback verifies against current state (a finding cited at file:line might already be resolved; stale findings get suppressed with evidence). Classification routes to one of four outcomes: stale (already resolved; just remove marker), trivial fix (apply directly), decision required (surface 2-4 options via `AskUserQuestion`), unresolvable (file as task or convert to `@todo:` / `@review(escalated):`). Marker removal is invariant on resolution (unless `--keep-resolved-markers` is set). Emits a resolution-log run summary. **Cross-AI loop**: `/opsx:review-external <name>` packages a self-contained prompt at `openspec/changes/<name>/.orbit-runs/external-prompt-<as>-<TS>.md` — paste into a different AI session, the external AI commits findings back to a sibling file, then `/opsx:address-reviews <name> --from-file <findings-path>` ingests and walks each. See `.claude/skills/openspec-review-external/SKILL.md` for the full external-review loop (recommended-session logic, output format, iteration tracking).

**Phase 5 — apply (`/opsx:apply <name>`)**: Implements tasks from `tasks.md` chunk by chunk. Chunks are natural pause points — at each chunk's end, the SKILL emits a run summary to `.orbit-runs/apply-<TS>.json` describing what was changed, what's done, what remains, and the next recommended action. The user can interrupt between chunks. Apply pauses if a task surfaces a premise problem (e.g., implementation reveals a design issue) and prompts for direction — `@review(escalated):` markers, artifact updates, or scope cuts are all valid responses.

**Phase 6 — verify (`/opsx:verify <name>`)**: Runs upstream's `openspec-verify-change` skill — structural correctness check confirming tasks are complete, spec coverage holds, and design adherence is intact. orbit composes this as the structural gate before system-mode review.

**Phase 7 — review (system mode, `/opsx:review <name> --as system`)**: Post-apply review of the whole product state. Pass 0 wraps `openspec-verify-change` (delegated structural check). Passes 1-6: Baseline Compliance (does this change break archived `openspec/specs/` requirements?) / Cohesion (callers/dependents outside the tasks list — signature drift, ripple effects) / Surface Walk (every CLI/MCP/HTTP/public-function surface in `openspec/specs/` still coherent?) / Perspective Reviews (for each named perspective in `openspec/lenses/perspectives.md`, simulate the caller's POV) / Critical-Path Scan (for each flow in `openspec/lenses/critical-paths.md`, walk end-to-end) / Drift / Residue (invokes `/opsx:audit-drift` as a library function). Run summary persists to `.orbit-runs/review-system-<TS>.json`.

**Phase 8 — address-reviews (system-mode findings)**: Same `/opsx:address-reviews` lifecycle as Phase 4, applied to system-mode review's findings. The cross-AI external loop also works here (`/opsx:review-external --as system`).

**Phase 9 — archive (`/opsx:archive <name>`)**: Moves the change to `openspec/changes/archive/<YYYY-MM-DD>-<name>/` and runs sync-specs (invokes the `openspec-sync-specs` upstream-required primitive) to propagate spec deltas (ADDED/MODIFIED/REMOVED/RENAMED) into baseline at `openspec/specs/`. Pre-archive sweeps: unresolved `@review:` marker check (prompts to address or convert to `@todo:`); audit-drift sweep (runs `/opsx:audit-drift --context pre-archive`). The per-change `.orbit-runs/` directory travels with the change into the archive, preserving the full audit trail. Run summary emits to `.orbit-runs/archive-<TS>.json` inside the archived location.

---

## Section 4: Quick-reference command table

Every slash command in your post-install `.claude/commands/opsx/` (15 files: 14 from orbit's overlay + 1 from upstream init untouched). One-line descriptions; orbit-modified commands carry orbit-specific behavior on top of upstream's body, orbit-authored commands are 100% orbit.

| Command | One-line description |
|---------|---------------------|
| `/opsx:address-reviews [<scope>] [--from-file <path>]` | Walk inline `@review:` markers or `--from-file` findings (external-review markdown, internal review JSON, or audit-drift JSON — auto-detected by content sniff) through pushback → classify → fix → ripple → remove. Resolves rather than scans. Auto-discovers internal review/audit-drift JSON when invoked with a change name and no markers found (no `--from-file` needed). |
| `/opsx:apply <name>` | Implement tasks from `tasks.md` chunk-by-chunk with end-of-chunk run-summary emits. Pauses on premise problems. |
| `/opsx:archive <name>` | Move change to `openspec/changes/archive/`, run sync-specs to propagate deltas to baseline. Pre-archive `@review:` marker check + audit-drift sweep. |
| `/opsx:audit-drift [<change-name>]` | Scan for drift between captured knowledge and reality (4 categories: vocabulary, lens, cross-doc, archive). Standalone or library-call. |
| `/opsx:bulk-archive` | Archive multiple completed changes at once. |
| `/opsx:continue <name>` | Create the next artifact in the change's workflow. |
| `/opsx:explore [<name>]` | Thinking-mode entry. Named: auto-captures decisions to `explore.md`. Bare: stays in chat until crystallization. |
| `/opsx:ff` | Fast-forward through artifact creation in one go. |
| `/opsx:new <name>` | Start a new change using the experimental artifact workflow. |
| `/opsx:onboard` | Run this skill (the reference walkthrough you're reading now). |
| `/opsx:propose <name>` | Generate proposal + design + specs + tasks artifacts in one step. Consume mode reads `openspec/explore/<name>/explore.md` if present. |
| `/opsx:review <name> [--as proposal\|system] [--fresh]` | Editorial review: 9 passes proposal-mode (pre-apply) or 7 passes system-mode (post-apply, wraps `verify-change` as Pass 0). |
| `/opsx:review-external <name> [--as proposal\|system]` | Package a self-contained prompt for a different AI to do a second-opinion review. Commits prompt to repo; external AI commits findings back. |
| `/opsx:sync <name>` | Sync delta specs from a change to main specs without archiving. |
| `/opsx:verify <name>` | Run upstream's structural verify-change (tasks done, spec coverage, design adherence). Composed as Pass 0 of system-mode review. |

For categorization (which commands are orbit-authored vs orbit-modified vs upstream-untouched), see `orbit-conventions` baseline `Overlay file disposition` requirement. This table documents the user-invocable surface; the disposition framework documents how orbit ships each file.

---

## Section 5: Try-it nudge

Two paths from here, depending on what you have:

- **If you have a concrete project idea** — run `/opsx:explore <your-change-name>`. Replace `<your-change-name>` with a kebab-case name describing what you want to build or change (e.g., `add-user-auth`, `migrate-postgres-to-sqlite`, `redesign-archive-flow`). Named-mode explore starts a thinking partnership: orbit asks clarifying questions, surfaces threads, captures decisions to `openspec/explore/<name>/explore.md` as they emerge. When decisions stabilize, `/opsx:propose <your-change-name>` generates proposal + design + specs + tasks from the explore.md.

- **If you're orienting and don't have a concrete idea yet** — run `/opsx:explore` (no name). Bare-mode explore stays in thinking-mode without committing to a change. Orbit acts as thinking partner; if 2+ substantive decisions emerge during the conversation, orbit prompts you to crystallize a name. Useful for investigating a vague concern, comparing approaches, or just orienting yourself to the codebase.

When you've gathered findings worth acting on, see `/opsx:address-reviews` to walk them. When you want a project-wide drift check, see `/opsx:audit-drift`. When you want a cross-AI second opinion, see `/opsx:review-external`.

