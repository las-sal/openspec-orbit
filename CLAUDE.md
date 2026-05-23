# CLAUDE.md — openspec-orbit

This is the source repository for [openspec-orbit](https://github.com/las-sal/openspec-orbit), a workflow tool that owns the `.claude/` surface (skills, commands, supporting docs) and uses `@fission-ai/openspec` at a pinned version (currently **`@fission-ai/openspec@1.3.1`**) as a CLI engine. Orbit ships its own editorial review, drift audit, and capture layers as part of that surface; the upstream CLI binary is unchanged and version-pegged.

Orbit is NOT a fork, NOT an automatic-update overlay, and NOT a thin augmentation of upstream. Treat the pegged upstream version as a deliberate dependency — upstream improvements after 1.3.1 do not auto-propagate; a version upgrade is its own deliberate change proposal.

The repo dogfoods orbit's own conventions: changes to orbit are themselves authored as openspec changes (see `openspec/changes/`), reviewed with the very review commands orbit ships, and archived via `/opsx:archive`.

## Working with orbit

Orbit codifies three cross-cutting execution disciplines, one for each phase of authoring work. They are bracketing requirements: they apply whether or not an orbit slash command is the active context. Per-command SKILL.md files repeat them self-contained (intentional duplication for reliability); this file reinforces them at the project level.

**Read-before-reference (authoring-time)**. When you generate code, tests, specs, or documentation that names a specific construct in this codebase — a function, type, interface, field, file path, spec requirement name, CLI flag, capability name — read the actual definition first. Use `Read`, `grep`, or `openspec instructions` as appropriate. Do NOT assume the shape based on common patterns or training-data conventions. If you can't verify the reference, ask or flag with `@review:`; never invent a plausible-looking reference and proceed silently. Conceptual reasoning (architectural patterns, design style) does not require this verification — the discipline kicks in when a specific named construct enters the output.

**Change completeness (modification-time)**. Substantive modifications to a change-in-flight (renames, capability merges, scope changes, marker conventions, file paths) must be applied fully across ALL affected artifacts before being declared done — specs, sketches, README, `design.md`, `tasks.md`, `explore.md`, command bodies, and any documentation. After mechanical replacement (sed, find-replace), do a manual sweep for residue patterns AND new bugs the mechanical replacement may have introduced (corrupted filenames, doubled words, broken references). Known residue MUST NOT be left for a downstream review to catch — that violates orbit's cost-up-front principle and corrupts the review signal. For external review, cleanup MUST complete BEFORE the prompt file is pushed to the remote.

**Pushback (review-time)**. When responding to flagged issues — from inline `@review:` markers, `/opsx:address-reviews --from-file` external findings, in-chat review summaries, or any other source — verify each claim against current state before acting. Use `grep`, file inspection, or `git log` to confirm the finding still applies. Stale findings (issue already fixed) get reported with evidence and suppressed; don't re-edit already-fixed state.

## Marker convention

The repo uses `@review: <text>` as the inline-review marker syntax (markdown carries it bare; source code and configs carry it inside the file type's comment syntax). Adjacent forms:

- `@review: <text>` — needs review/decision
- `@review(escalated): <text>` — escalated; needs human decision
- `@todo: <text>` — known follow-up work, not a review item

Markers are resolved via `/opsx:address-reviews` (lean v1: discover → triage → walk → ripple flag → report; removes markers on resolution).

## Persistence layout

- `openspec/lenses/perspectives.md` and `openspec/lenses/critical-paths.md` — judgment layer (subjective per-system "which callers matter" / "which flows are critical"); grown via `/opsx:explore` capture triggers; consumed by `/opsx:review --as system` Passes 4 and 5.
- `openspec/changes/<name>/.orbit-runs/` — per-change iteration audit trail (review-summary JSON files, external-review findings markdown, archive run summaries). Committed, dot-prefixed, travels with the change into `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/` (date prefix added by upstream's archive move).
- `openspec/explore/<name>/` — pre-propose staging for in-progress explorations. `/opsx:propose <name>` consume mode reads `explore.md`, generates artifacts, and *moves* the staging directory to `openspec/changes/<name>/`.

## When in doubt

- Orbit's behavior is defined in markdown prompts under `.claude/skills/openspec-*/SKILL.md` and `.claude/commands/opsx/*.md`. When implementing orbit changes, those are the source of truth.
- Capability specs in `openspec/specs/<capability>/` (or `openspec/changes/<name>/specs/<capability>/spec.md` for in-flight changes) are the normative contract that SKILL.md content must satisfy.
- The README documents the user-facing surface; CLAUDE.md (this file) is what loads automatically when an AI session starts in this repo.
