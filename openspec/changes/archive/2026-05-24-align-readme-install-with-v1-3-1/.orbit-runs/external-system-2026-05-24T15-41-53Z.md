# External Review: align-readme-install-with-v1-3-1 (iteration 1)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-24

## CRITICAL

### Uninstall leaves orbit-only files behind
**File**: README.md:1036
**Description**: The uninstall sequence promises "revert to upstream `@fission-ai/openspec@1.3.1` only", but it removes only the four orbit-authored review/audit/address commands and skills plus `feedback/`. In a fresh sandbox I ran the README install flow (`npx -y @fission-ai/openspec@1.3.1 init --tools claude`, orbit `cp -r` overlay, `rm -rf .claude/skills/feedback`) and then the documented uninstall flow. After `npx -y @fission-ai/openspec@1.3.1 init --force --tools claude`, the project still had `.claude/skills/openspec-sync-specs` and `.claude/commands/opsx/{sync,fast-forward}.md`, leaving 11 skills and 12 commands instead of the fresh-upstream 10 + 10. `init --force` overwrites upstream-installed files but does not delete target-only orbit files, so the section is not actually an upstream-only revert. Add explicit removals for the orbit-shipped target-only files (`.claude/skills/openspec-sync-specs`, `.claude/commands/opsx/sync.md`, `.claude/commands/opsx/fast-forward.md`) or narrow the section's claim; the current wording contradicts the install-surface requirement and will leave users with orbit commands still installed.

## WARNING

### Orbit additions heading level is documented incorrectly
**File**: README.md:956
**Description**: The README repeatedly says the modified upstream skills contain an `## Orbit additions` section, but the actual skill files and the baseline spec use `# Orbit additions`. In this checkout, `rg -l '^# Orbit additions' .claude/skills/openspec-*/SKILL.md` returns the expected 10 files, while `rg -l '^## Orbit additions' .claude/skills/openspec-*/SKILL.md` returns none. Update the prose at the prerequisite, overlay, post-install table, and customization warning sites to say `# Orbit additions`, or change the actual headings consistently. As written, the docs name a section that does not exist.

## SUGGESTION

None.

## Notes

Reviewed `main` at `b700d967ed8bd4f1579508c3ca62680ebb5588e0`; `git ls-remote origin refs/heads/main` returned the same commit. `openspec validate align-readme-install-with-v1-3-1 --strict` passes. Fresh-sandbox install/update checks matched the README's 15 skills + 16 commands post-overlay counts. no perspectives defined; skip Pass 4. no critical paths defined; skip Pass 5.
