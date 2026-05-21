# External Review: emit-run-summary-jsons-from-workflow-commands (system mode, iteration 1)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone fresh; the change is on `main` at commit `ffa0301` or later)

## Project context (read first)

- `CLAUDE.md` — orbit's authoring conventions (three execution disciplines: read-before-reference / change-completeness / pushback)
- `README.md` — orbit's user-facing surface; specifically the new "Run-summary JSON emission" section added by this change
- `openspec/specs/orbit-conventions/spec.md` — the BASELINE before this change's MODIFIED delta. Specifically: `Requirement: Internal-run JSON summary format` at line 159 (baseline), `Requirement: .orbit-runs/ per-change persistence` at line 135 (baseline). Compare against `openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-conventions/spec.md` (the delta).
- `openspec/specs/orbit-*/spec.md` — other established orbit capability specs
- `openspec/changes/archive/2026-05-18-bootstrap-openspec-orbit/` — prior archived change that established orbit-conventions, the 3+1 per-skill schema reference files, and the editorial-command emit pattern (prior art)
- `.claude/skills/openspec-<skill>/references/*-schema.md` — the per-skill schema reference files (4 of them: run-summary-schema for review/audit-drift/address-reviews + archive-summary-schema for archive-change), ALL modified by this change to add universal spine inheritance + spine fields
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/` — extensive iteration history (4 address-reviews, 7 apply chunk emits, 3 proposal reviews + 1 external Codex, 1 verify run)

## Cycle context

- **Iteration**: 1 (first external system-mode review for this change)
- **Prior internal proposal-mode reviews**: 3 (iter-1 in-context: 19 findings; iter-2 fresh-context: 5 findings; iter-3 in-context --thorough: 3 findings) — all resolved
- **Prior external proposal-mode review**: iter-1 GPT-5 Codex (9 findings) — all resolved
- **Total prior findings**: 36 across all 4 proposal-mode modes; all resolved per their address-reviews resolutions
- **Apply phase**: complete on the AI side. 38/44 tasks done; 6 user-validation tasks remain unchecked (acknowledged orbit convention for live-tests + handoff).
- **Prior verify**: iter-1 verdict=warn, 2 findings (1 WARNING for the unchecked user-validation tasks per convention; 1 SUGGESTION already resolved). Validates --strict.
- **Resolved this verify cycle**: 1 SUGGESTION (stale Impact bullet in proposal.md).

**Authoring/internal-reviewer context (for cross-AI anchoring awareness)**: ALL proposal artifacts, apply implementation, and all 4 proposal-mode reviews were authored by Claude Opus 4.7 (3 in-context including iter-3 --thorough; 1 fresh-context subagent). The only cross-AI surface so far has been GPT-5 Codex's iter-1 external proposal-mode review (which caught 8 anchoring artifacts plus 1 net-new gap). Your value as a fresh-context system-mode reviewer: catch what same-model anchoring suppressed across the apply phase AND across all 4 proposal-mode review cycles. Per `openspec-orbit#20` empirical evidence on the prior orbit-status dogfood: 3/3 real bugs were caught only by external system-mode review (none by in-context system review). Be thorough especially on cohesion (Pass 2) and drift (Pass 6).

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as system)

System mode reviews the IMPLEMENTATION against the spec + the change's coherence with the broader orbit codebase:

- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/{proposal.md, design.md, tasks.md, specs/}`
- `git diff 0de2de9..HEAD -- .claude/ openspec/changes/emit-run-summary-jsons-from-workflow-commands/` (the change's commit range — `0de2de9` was the parent commit when the change was created)
- `openspec/specs/orbit-conventions/spec.md` (archived baseline — compare to MODIFIED delta in change)
- `openspec/specs/orbit-*/spec.md` (other capabilities, esp. those whose schema refs were updated)
- All 10 `.claude/skills/openspec-*/SKILL.md` files modified by this change (look for the `# Orbit additions` sections):
  - `openspec-explore`, `openspec-propose` (extended existing additions)
  - `openspec-new-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-review-external`, `openspec-audit-drift` (gained new additions — or extended existing for orbit-authored)
  - `openspec-bulk-archive-change`, `openspec-onboard` (gained scope-enforcement-only additions)
- The 4 schema reference files: `.claude/skills/openspec-{review,audit-drift,address-reviews}/references/run-summary-schema.md` + `.claude/skills/openspec-archive-change/references/archive-summary-schema.md` (all updated with universal-spine inheritance section + spine fields in JSON examples)
- The 7 manual `apply-<TS>.json` files in `.orbit-runs/` (dogfood evidence that the apply chunk-end emit convention is implementable — written by hand during apply since the emit-layer being SPEC'd doesn't exist yet)

## What to look for

Apply ALL 7 system-mode passes. For each finding raised, cite file + line + a specific recommendation. Don't be charitable to the authoring AI's reasoning.

**Pass 0 — verify-change-style structural check**: tasks done (note: 38/44 with 6 user-validation acknowledged), spec coverage, design adhered to. `openspec validate --strict` should pass.

**Pass 1 — Baseline Compliance**: does this change break any archived `openspec/specs/` requirement? Specifically check the MODIFIED orbit-conventions delta: does it correctly COPY the original requirements before editing (per the MODIFIED workflow rule)? Did baseline scenarios get preserved (with broader wording where needed)?

**Pass 2 — Cohesion**: callers/dependents outside the tasks list. Specifically:
- The new `# Orbit additions` sections in 10 SKILL.md files — do they interact correctly with each other? E.g., the verify SKILL.md says "delegates to address-reviews on mode ②"; does the address-reviews SKILL.md actually handle this case?
- The universal spine in orbit-conventions vs. the per-skill schema references: are the 4 schema refs' JSON examples consistent with the spine they claim to inherit?
- /opsx:continue's "isComplete defer" — does the actual upstream openspec-continue-change behavior support this? (Verify via grep of the actual upstream SKILL.md.)
- The propose-shaped variants (new, continue, ff) — are their `# Orbit additions` sections consistent with each other and with propose's existing additions?

**Pass 3 — Surface Walk**: every CLI / public surface in `openspec/specs/` still coherent? Capabilities ARE surfaces. Specifically:
- Does the new orbit-run-summary-emit capability surface contradict or duplicate any orbit-conventions surface?
- Does the MODIFIED orbit-conventions persistence requirement correctly enumerate all 3 paths used by the rest of the change?
- Do the 4 schema reference files' Universal spine inheritance notes correctly cite orbit-conventions (matching file + requirement name)?

**Pass 4 — Perspective Reviews**: simulate the caller's POV. Concrete perspectives worth walking:
- **"Fresh AI invoking /opsx:propose for the first time after this change lands"** — does it know to emit propose-<TS>.json? The SKILL.md additions tell it to; the schema reference example shows the shape. Is there any ambiguity that could cause emit failure?
- **"orbit-status reading the new emit JSONs"** — its tier-1 reader expects `next_recommended` as a verbatim string. Does the universal spine + per-kind extensions produce JSONs orbit-status can parse natively? (Cross-reference: orbit-status's `orbit-status-recommendation/spec.md:7` — orbit-status is in a different repo; check the path matches if you can.)
- **"User looking at the 7 chunk apply emits in .orbit-runs/"** — do those emits self-describe the convention they followed? Are they coherent with each other?

**Pass 5 — Critical-Path Scan**: walk end-to-end flows. Critical paths to walk:
- `/opsx:explore foo` (named-mode) → emits → `/opsx:propose foo` → consumes + emits → `/opsx:review foo` → emits → `/opsx:address-reviews foo` → emits → `/opsx:apply foo` (chunked) → 7 chunk emits → `/opsx:verify foo` → emits → `/opsx:review --as system foo` → emits → `/opsx:archive foo` → emits. At each step, does the previous emit's `next_recommended` actually point at the next command?
- `/opsx:explore` (bare-mode) → no emit → crystallization warning → user confirm → `/opsx:explore <name>` (named-mode) → first emit. Does the warning convention actually prevent silent crystallization?
- /opsx:audit-drift (project-wide standalone, no args) → openspec/.orbit-runs/audit-drift-<TS>.json → does the change-scoped vs project-wide split in orbit-run-summary-emit + audit-drift SKILL.md actually map correctly to the existing schema reference's two-mode documentation?

**Pass 6 — Drift / Residue**: vocabulary residue, stale references, archive-coherence misses. Specifically:
- "emit JSON" vs "run-summary JSON" terminology: was the W7+T3 sweep complete across all artifacts AND the modified SKILL.md files?
- `/opsx:ff` vs `/opsx:ff-change` (EC1's fix): is the new command name consistent everywhere except where the SKILL filename intentionally uses `openspec-ff-change`?
- Any references to `openspec/conventions/` (the path eliminated by C2): is the cleanup actually complete?
- Any references to `findings_summary` / `finding_titles` in editorial schemas that contradict EW2's "omit at T0 for review-external"?
- The bridge files (`review-proposal-*-as-findings.md`) committed in .orbit-runs/ per openspec-orbit#4 — are they appropriately marked as transitional?

## Output format — write to:

`openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format like `2026-05-21T04-30-00Z`. Pick a fresh timestamp so this file doesn't overwrite prior reviews.)

Use this exact markdown structure (the section headers + field labels are parsed by `/opsx:address-reviews --from-file`; deviating breaks ingestion):

```markdown
# External Review: emit-run-summary-jsons-from-workflow-commands (system mode, iteration 1)

**Reviewer**: <your model name, e.g., "GPT-5 Codex" or "Claude Sonnet 4.5">
**Date**: 2026-05-21

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(For each additional CRITICAL finding, repeat the `### <Title>` + `**File**:` + `**Description**:` triple. Use `None.` as a single-word body if there are no findings at this severity.)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(Same shape as CRITICAL. Use `None.` if no findings.)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(Same shape. Use `None.` if no findings.)

## Notes

(Optional: overall impression, broader concerns, things you noticed that didn't fit a specific finding.)
```

After writing the file, do ALL THREE of the following:

1. **Echo the findings in your chat response** — output the full markdown (the same content as the file you just wrote) so the user can read findings directly in chat without pulling the file. This is important because the user may want to scan findings in their chat session before triggering `/opsx:address-reviews`.

2. **Commit and push the findings file to the remote** — run `git add <path> && git commit -m "external review: emit-run-summary-jsons-from-workflow-commands iter-1 (<your reviewer model name>)" && git push` (or equivalent for your environment). The user pulls the findings file via `git pull` to trigger `/opsx:address-reviews --from-file`. If you cannot push from your environment (no remote write access, sandboxed, etc.), explicitly say so in chat so the user knows to commit + push manually.

3. **Mention the file path** in your final chat message so the user can find it for the `/opsx:address-reviews --from-file` step.
