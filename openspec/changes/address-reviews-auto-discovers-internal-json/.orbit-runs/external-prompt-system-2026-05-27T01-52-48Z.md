# External Review: address-reviews-auto-discovers-internal-json (iteration 1, system mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

This is a **system-mode** review (post-apply): the change has been implemented and you are verifying the whole product state — code-vs-spec coherence, baseline compliance, surface walk, drift residue.

## Repo

https://github.com/las-sal/openspec-orbit

## Project context (read first)

- `CLAUDE.md` — orbit handoff orientation
- `openspec/project.md` — project goals + stack (if present)
- `openspec/lenses/perspectives.md` — does NOT exist for orbit; Pass 4 N/A
- `openspec/lenses/critical-paths.md` — does NOT exist for orbit; Pass 5 N/A
- `openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/` — iteration history (12 files: 1 propose, 2 review-proposal, 2 address-reviews, 1 review-external T0, 1 external-prompt, 1 external-proposal findings, 3 apply, 1 review-system)

## Cycle context

- **Iteration**: 1 (first external SYSTEM-mode review for this change; proposal-mode had 1 external by Codex earlier)
- **Prior internal review findings still open**: 0
  - Proposal iter-1 internal `--fresh`: 0/2/3 → 5 resolved inline
  - Proposal iter-2 internal `--fresh`: 0/0/1 cosmetic → inline-fixed (convergence)
  - System iter-1 internal `--fresh` (just ran): 0/0/1 cosmetic → inline-fixed (Source: template enum drift)
- **Prior external review findings still open**: 0
  - Proposal iter-1 external (GPT-5 Codex): 0/4/0 → all 4 resolved via address-reviews iter-2 walk
- **Resolved since last external review**: 4 substantive design + spec edits via iter-2 address-reviews walk (`address-reviews-2026-05-27T01-21-40Z.json`):
  - WARN 1 — archived change-name resolution algorithm (active → archive regex `^\d{4}-\d{2}-\d{2}-<name>$` → path/pattern fallback) codified normatively in spec intro + task 1.1
  - WARN 2 — new spec scenario `Auto-discovery resolution log captures audit-trail evidence` codifying 5 normative fields (`source`, `source_path`, `source_command`, `source_token`, `latest_apply_token`) + optional `tie_break_rationale`
  - WARN 3 — `internal-findings-format.md:119` strictness residue (`command` must be `"review"`) fixed (broadened to `"review" OR "audit-drift"`)
  - WARN 4 — `--from-file` exclusivity codified per Option A: new normative block in spec intro + scenario rewritten

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What this change does (one-paragraph)

Closes [GitHub openspec-orbit#10](https://github.com/las-sal/openspec-orbit/issues/10). Adds auto-discovery layer on top of the just-archived parent change `address-reviews-accepts-internal-json` (closes #4) parser machinery. `/opsx:address-reviews <change-name>` now auto-discovers the most-recent `review-<mode>-*.json` OR `audit-drift-*.json` in the change's `.orbit-runs/` when no `@review:` markers are found AND no `--from-file` is specified. Strictly additive — whole-repo invocation, path/pattern scope, explicit `--from-file`, and `--mark` usage all byte-identical from user's perspective. Discovery rule (Option A per pre-decision): filename `<TS>` token, lexical sort, single global most-recent across both candidate types (review and audit-drift compete on the same recency axis with no class preference). Spec delta: 1 MODIFIED requirement (`Address-reviews command available`) with 11 scenarios. Implementation surface: SKILL Step 1 (rewrite) + opsx command file mirror + 2 reference docs + 2 onboard files + README — 6 touched surfaces. The proposal phase had 1 internal + 1 external (Codex) + 1 internal convergence-pass; all 9 findings resolved across 2 address-reviews walks.

## What to read for THIS review (`--as system`)

```
- openspec/changes/address-reviews-auto-discovers-internal-json/proposal.md
- openspec/changes/address-reviews-auto-discovers-internal-json/design.md (6 decisions; D-shape-1, D-discovery-1, D-priority-1, D-recency-1, D-explicit-override, D-no-mark-prerequisite, D-no-stale-detection)
- openspec/changes/address-reviews-auto-discovers-internal-json/tasks.md (13/13 tasks checked)
- openspec/changes/address-reviews-auto-discovers-internal-json/specs/orbit-address-reviews/spec.md (1 MODIFIED requirement, 11 scenarios)
- The change's commit range: git diff main..HEAD over the .claude/ + README + openspec/changes/address-reviews-auto-discovers-internal-json/ paths
- openspec/specs/orbit-address-reviews/spec.md (synced baseline — pre-sync version of the MODIFIED target is at L9-21 with 2 scenarios; this change adds 9 net-new + 2 normative intro paragraphs)
- openspec/specs/orbit-run-summary-emit/spec.md (Audit-drift standalone recommendations requirement at L396-440 — verify this change's auto-discovery correctly handles the audit-drift JSON shape that requirement mandates)
- openspec/changes/archive/2026-05-26-address-reviews-accepts-internal-json/ (parent change — this change orchestrates on top of its parser machinery)
- .claude/skills/openspec-address-reviews/SKILL.md (Step 1 Discover comprehensively rewritten; Step 5 Report template extended with auto-discovered enum)
- .claude/commands/opsx/address-reviews.md (mirror)
- .claude/skills/openspec-address-reviews/references/internal-findings-format.md (title broadened + L119 residue fixed)
- .claude/skills/openspec-address-reviews/references/run-summary-schema.md (source enum extended + 4 new audit-trail fields + field-note prose)
- .claude/skills/openspec-onboard/SKILL.md + .claude/commands/opsx/onboard.md (command-table rows appended)
- README.md L428 (new Auto-discovery paragraph)
```

## What to look for (system-mode passes)

0. **verify-change-style structural check** — 13/13 tasks checked; spec coverage; design adhered.
1. **Baseline Compliance** — does this change break any archived `openspec/specs/` requirement? Note: orbit-address-reviews baseline still describes the OLD pre-sync framing; MODIFIED resolves on archive. Pre-sync divergence is expected.
2. **Cohesion** — 6 surfaces touched. Are there callers/dependents outside these files that should be updated? (e.g., other SKILL files that describe `/opsx:address-reviews`; orbit-status code that parses address-reviews JSONs and might need updating for the 4 new audit-trail fields).
3. **Surface Walk** — `/opsx:address-reviews` CLI surface: same flags (`--keep-resolved-markers` + `--from-file`); behavior contract extended for change-name positional + no-markers + no-from-file case. Is this a breaking change for any consumer? Are the 4 new audit-trail JSON fields parseable by orbit-status's tier-1 best-effort parser?
4. **Perspective Reviews** — N/A (no lenses).
5. **Critical-Path Scan** — N/A (no lenses).
6. **Drift / Residue** — cross-doc consistency across the 6 touched surfaces. Specifically: is the auto-discovery feature described coherently in SKILL + command + 2 reference docs + 2 onboard files + README? Any leftover "review JSON only" or "must pre-run --mark" residue?

**Specific scrutiny areas** (areas I want second-opinion validation):

- **Cross-archive coherence with the just-archived #4**. The change references `--from-file ingest of review findings file` requirement multiple times. Verify those references resolve cleanly in the synced baseline.

- **The 4 new audit-trail fields in run-summary-schema.md** — `source_command`, `source_token`, `latest_apply_token`, `tie_break_rationale`. Are they coherent with the spec's `Auto-discovery resolution log captures audit-trail evidence` scenario? Are the field-note prose explanations sufficient for a fresh implementer to emit them correctly?

- **Positional resolution algorithm precision**. SKILL.md L58-62 enumerates: active match first → archive regex `^\d{4}-\d{2}-\d{2}-<scope>$` with latest-date tie-break → path/pattern fallback. Is this algorithm implementable without ambiguity? What happens if a path/pattern coincidentally LOOKS like a change name (e.g., user invokes `/opsx:address-reviews proposal.md` and there's no `openspec/changes/proposal.md/` — resolves to path/pattern correctly per the rule, but is that obvious from the SKILL prose)?

- **`--from-file` exclusivity normative block**. SKILL.md L44 + spec intro both codify that `--from-file` when present means marker grep does NOT run, auto-discovery does NOT run, and positional is used only for emit-path resolution. Is the emit-path-only-positional semantics surfaced clearly? Would a user invoking `/opsx:address-reviews <change-name> --from-file <path>` understand that the resolution log gets written to `<change-name>/.orbit-runs/`?

- **D-no-stale-detection design choice**. The change deliberately doesn't refuse stale JSON at discovery; instead records `latest_apply_token` in the resolution log. Is this defensible given the parent `harden-review-mode-recommendations` adopts a different stance for system-mode review's State 5 (refuse + recommend re-run-external)? The asymmetry is: review-skill detects stale; address-reviews doesn't. Documented in design.md D-no-stale-detection.

- **Worked example in SKILL.md**. The two adjacent worked examples (markdown + JSON) demonstrate `--from-file` paths but NOT auto-discovery. Would a third example (showing `/opsx:address-reviews <change-name>` auto-discovering the JSON) materially improve codegen-readiness, or is it covered by the spec scenarios + SKILL prose?

- **README discoverability**. The new Auto-discovery paragraph at L428 sits inside the `/opsx:address-reviews` command-reference block. Is it discoverable for new users skimming the TOC, or buried? Consider whether the README's `## Choosing a review mode` section (added by harden-review-mode-recommendations) should also reference auto-discovery as the canonical 2-command workflow.

## Output format — write to:

`openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format with colons replaced — `YYYY-MM-DDTHH-MM-SSZ`. Pick a fresh timestamp so this file doesn't overwrite any prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: address-reviews-auto-discovers-internal-json (iteration 1, system mode)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(Repeat for each. Use `None.` if no findings at this severity.)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(Same. Use `None.` if no findings.)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(Same. Use `None.` if no findings.)

## Notes

<Optional. Omit if nothing to add.>
```

## After completing the review

1. **Output the full findings markdown in chat** in addition to writing the file — every section, every `### Title`, every `**File**:` / `**Description**:` field. Do NOT abbreviate.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/external-system-<TS>.md
   git commit -m "External review (system, iter 1): address-reviews-auto-discovers-internal-json

   <one-line summary>"
   git push
   ```

If you don't have git access, output the findings markdown in chat and the user will commit it manually.
