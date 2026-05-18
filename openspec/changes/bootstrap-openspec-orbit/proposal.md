# Bootstrap openspec-orbit

## Why

The recurring review-and-capture workflow that has been working ad-hoc across multiple OpenSpec projects (home-control, home-env, home-control-test) requires the user to re-negotiate disciplines with the AI in every session — `@review:` marker conventions, pushback-on-stale-findings, marker-removal-on-resolution, cross-AI handoff format, capture habits. The result is inconsistent application of patterns that materially improve spec quality.

openspec-orbit makes the pattern durable: a `.claude/` overlay shipped as a standalone repo that extends upstream `@fission-ai/openspec` with editorial review passes, a drift audit, a formal marker convention, frictionless external-AI handoff, and an exploration-capture layer.

## What Changes

- **New command** `/opsx:review --as proposal` — editorial review pass over change artifacts before apply (9 passes rolled into a 3-dimension scorecard with CRITICAL/WARNING/SUGGESTION severities).
- **New command** `/opsx:review --as system` — editorial review pass over the whole product after apply (wraps upstream `verify-change` as Pass 0; adds 6 system-wide passes covering baseline compliance, cohesion, surface walk, perspectives, critical paths, drift).
- **New command** `/opsx:review-external` — packages a review request for an external AI (codex, fresh Claude, etc.); writes the full prompt to a versioned file (`.orbit-runs/external-prompt-<as>-<TS>.md`, committed) and emits a short invocation snippet to chat; defines the file-based findings format that closes the cross-AI loop without copy-paste-per-finding.
- **New command** `/opsx:audit-drift` — project-wide scan for drift between captured knowledge and reality (vocabulary residue, lens staleness, cross-doc consistency, archive coherence). Composes as a library function called by `/opsx:review --as system` Pass 6 and auto-invoked by `/opsx:archive` as a pre-archive sweep.
- **New command** `/opsx:address-reviews` (lean v1) — resolution counterpart to the review commands. Walks `@review:` markers anywhere in the repo (or ingests external-review findings via `--from-file`) with pushback discipline; removes markers on resolution.
- **Modified `/opsx:explore`** — preserves the upstream "stance, not workflow" character; adds capture affordances (5 types: conventions, perspectives, critical paths, decisions, references) and `explore.md` authoring; supports three invocation modes (bare / named / crystallized).
- **Modified `/opsx:propose`** — preserves upstream's standalone behavior; adds consume mode that reads `openspec/explore/<name>/explore.md`, prompts for Open question handling, generates artifacts, then *moves* the staging directory to `openspec/changes/<name>/`.
- **Modified `/opsx:archive`** — preserves upstream archive behavior; auto-invokes `/opsx:audit-drift` as a pre-archive sweep (opt-out via `--skip-audit`); writes archive run summary to `.orbit-runs/` capturing audit outcome and user decision.
- **New marker convention** `@review: <content>` — single inline-review marker syntax across markdown, source code, configs (each file type's own comment syntax wraps it where needed).
- **New project-level structure** `openspec/lenses/` — judgment layer containing `perspectives.md` and `critical-paths.md`; grown via explore capture triggers; consumed by system-mode Passes 4 and 5.
- **New per-change persistence** `openspec/changes/<name>/.orbit-runs/` — committed, dot-prefixed directory holding internal-run summaries (JSON) and external-review findings (markdown). Travels with the change into archive.

## Capabilities

### New Capabilities

- `orbit-review`: The unified `/opsx:review <name> [--as proposal|system]` command — both modes (proposal-side 9 passes, system-side `verify-change` + 6 system-wide passes), shared 3-dimension scorecard, shared flag family (`--fast`/`--full`/`--thorough`/`--parallel`/`--focus`/`--fresh`/`--strict`), mode-specific flags (`--mark` for proposal, `--skip-verify` for system), state-based mode inference.
- `orbit-review-external`: The `/opsx:review-external` command — `--as` mode flag with state-inference default, mode-specific prompt content, defined external-findings file format, iteration counting.
- `orbit-audit-drift`: The `/opsx:audit-drift` command — 4 scan categories, library + standalone + auto-invoke composition, drift-pattern sourcing from archived deltas.
- `orbit-address-reviews`: The `/opsx:address-reviews` command (lean v1) — marker discovery, pushback discipline, classification, marker removal, `--from-file` ingest of external findings, resolution log output.
- `orbit-explore-modifications`: The modifications to `/opsx:explore` — 5 capture types with trigger patterns, 3 invocation modes including the crystallization heuristic, `explore.md` five-section convention.
- `orbit-propose-modifications`: The modifications to `/opsx:propose` — consume mode (read `explore.md`, handle Open questions, move staging dir to change dir), standalone mode preserved.
- `orbit-archive-modifications`: The modifications to `/opsx:archive` — pre-archive `audit-drift` hook, critical-drift prompt (not gate), archive run summary, `.orbit-runs/` move with the change.
- `orbit-conventions`: Cross-cutting conventions consumed by multiple commands — `@review:` marker syntax, `openspec/lenses/` directory structure and content shape, `.orbit-runs/` persistence layout, internal-run JSON summary format, external-review markdown findings format.

### Modified Capabilities

None. orbit ships as a downstream overlay; upstream OpenSpec capabilities are not changed in this repo. The `orbit-*-modifications` capabilities above describe orbit's overlay behavior, which sits on top of upstream commands at runtime via `.claude/skills/` and `.claude/commands/opsx/` overrides.

## Impact

- **`.claude/` overlay**: ships modifications to `openspec-explore`, `openspec-propose`, and `openspec-archive-change` SKILL.md files (plus corresponding `.claude/commands/opsx/*.md` slash command bodies); adds five new SKILL.md + command body pairs.
- **New directories at runtime**: `openspec/explore/<name>/` (staging for in-progress explorations), `openspec/lenses/` (project judgment layer), `openspec/changes/<name>/.orbit-runs/` (per-change iteration history).
- **No upstream CLI changes**: the `openspec` binary itself is unchanged. orbit's behavior lives in markdown prompts read by the AI.
- **No breaking changes**: standalone mode of `/opsx:propose` preserves upstream behavior. `--skip-audit` opts out of the new archive hook. Empty `openspec/lenses/` gracefully degrades. Adopters can install orbit incrementally; partial adoption works.
- **External-AI integration**: orbit defines but does not own the external review surface. Compatible with codex (primary target), fresh Claude, GPT, or any AI capable of reading a repo and writing a file.
- **License**: MIT (matches upstream).
- **Repo**: `github.com/las-sal/openspec-orbit` (currently private during v1 development).
