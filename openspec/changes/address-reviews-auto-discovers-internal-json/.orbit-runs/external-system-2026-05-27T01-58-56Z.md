# External Review: address-reviews-auto-discovers-internal-json (iteration 1, system mode)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-27

## CRITICAL

None.

## WARNING

### Graceful-degradation section still short-circuits the auto-discovery path
**File**: .claude/skills/openspec-address-reviews/SKILL.md:323
**Description**: Step 1 now correctly says a change-name invocation with zero inline markers must look for the newest `review-<mode>-*.json` or `audit-drift-*.json` before deciding there is no work. The later "Graceful degradation" bullet still says "No markers found" emits `No @review: markers in scope. Nothing to do.` and exits cleanly, with no distinction between bare/path scans and change-name scans. A fresh implementer using this bottom checklist could preserve the original bug: `/opsx:address-reviews <change-name>` after `/opsx:review <change-name>` finds zero markers and exits without checking `.orbit-runs/`. Rewrite this bullet to match the new priority order, e.g. "No markers found in bare/path scope" keeps the old message, while "change-name scope has no markers and no candidate JSON" emits the new internal JSON no-work message.

### Skill metadata still presents `--mark` as the paired review workflow
**File**: .claude/skills/openspec-address-reviews/SKILL.md:3
**Description**: The executable prose now says `--mark` is optional, but the skill frontmatter still advertises the skill as "Use after `/opsx:review --mark` drops markers" and the compatibility field says it "Pairs with `/opsx:review --mark`." Skill metadata is part of the product surface because clients and fresh AIs often see it before the body. Leaving it stale undercuts the change's main promise that `/opsx:review <name>` followed by `/opsx:address-reviews <name>` works without pre-marking. Update the description/compatibility text to mention auto-discovered internal review/audit-drift JSON and explicitly frame `--mark` as optional.

## SUGGESTION

### README walkthrough still teaches the old marker-only follow-up
**File**: README.md:531
**Description**: In the "full cycle" walkthrough, the authoring-session follow-up after `/opsx:review foo --as proposal` is still shown as bare `/opsx:address-reviews` that "walks any @review: markers you've left in artifacts." That misses the new canonical two-command flow and uses the wrong shape for auto-discovery, which requires the change-name positional (`/opsx:address-reviews foo`). The dedicated auto-discovery paragraph later in the command reference is correct, but a skimming user following this main workflow block will still infer that they need markers. Update the walkthrough to show `/opsx:address-reviews foo` and note that it auto-discovers the just-written internal JSON when no inline markers exist.

### Audit-drift recommendations still hard-code the now-optional `--from-file` path
**File**: openspec/specs/orbit-run-summary-emit/spec.md:403
**Description**: The change deliberately makes change-scoped audit-drift JSON discoverable by `/opsx:address-reviews <name>`, but the audit-drift emit contract still requires findings recommendations to begin with `/opsx:address-reviews <name> --from-file <this-json>`, and line 412 still says the flag becomes optional "once openspec-orbit#10 lands." That explicit recommendation remains valid, so this is not blocking, but after this change lands it becomes stale user-facing guidance and leaves the new auto-discovery path unused by the command that emits `audit-drift-*.json`. Consider updating the change-scoped recommendation to `/opsx:address-reviews <name>` while keeping project-wide audit-drift on explicit `--from-file` because there is no change-directory anchor.

### External findings reference still calls the JSON sibling "internal review JSON"
**File**: .claude/skills/openspec-address-reviews/references/external-findings-format.md:3
**Description**: The external markdown reference says the other supported `--from-file` format is "internal review JSON." The parser contract now supports both review JSON and audit-drift JSON, and the internal reference was broadened to "internal findings JSON." This small residue does not break parsing, but it is exactly the old review-only vocabulary the change is trying to retire. Change the line to "internal findings JSON" and mention `review` or `audit-drift` if useful.

## Notes

`openspec validate address-reviews-auto-discovers-internal-json --strict` passed. `openspec validate --all --strict` also passed with 11/11 items valid. `openspec/project.md`, `openspec/lenses/perspectives.md`, and `openspec/lenses/critical-paths.md` are absent in this repo as the prompt expected, so perspective and critical-path passes were not applicable.
