# External Review: address-reviews-accepts-internal-json (iteration 1, system mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

This is a **system-mode** review (post-apply): the change has been implemented and you are verifying the whole product state — code-vs-spec coherence, baseline compliance, surface walk, drift residue. Distinct from proposal-mode (pre-apply, artifacts only).

## Repo

https://github.com/las-sal/openspec-orbit

## Project context (read first)

- `CLAUDE.md` — handoff orientation for openspec-orbit
- `openspec/project.md` — project goals + stack (if present)
- `*_convention.md` at repo root (if any — orbit currently has none)
- `openspec/lenses/perspectives.md` — does NOT exist for orbit; Pass 4 is N/A
- `openspec/lenses/critical-paths.md` — does NOT exist for orbit; Pass 5 is N/A
- `openspec/changes/address-reviews-accepts-internal-json/.orbit-runs/` — full iteration history (8 files: propose JSON, 2 review-proposal JSONs, 1 address-reviews JSON, 3 apply JSONs, 1 review-system JSON)

## Cycle context

- **Iteration**: 1 (this is the FIRST external review for this change — both proposal-mode and system-mode externals are new).
- **Why no proposal-mode external**: the proposal phase used an **inline-walk workaround** instead of `/opsx:review-external` + `/opsx:address-reviews --from-file`. Reason: v1 of `--from-file` doesn't accept JSON, and this change's proposal-mode iter-1 review (`review-proposal-2026-05-26T20-39-14Z.json`) lived in JSON. The user invoked `/opsx:address-reviews` bare, which then walked the findings inline. That's exactly the gap this change closes. This is documented in the address-reviews JSON's `source_walk_workaround` field.
- **Prior internal review findings still open**: 0.
  - Proposal iter-1 `--fresh`: 1 CRITICAL / 2 WARNING / 4 SUGGESTION → 5 resolved + 2 deferred via inline-walk.
  - Proposal iter-2 `--fresh`: 0/0/0 (convergence confirmed).
  - System iter-1 `--fresh`: 0/0/2 → both SUGGESTIONs inline-fixed (README directory tree comment + SKILL worked-example archived-path parenthetical).
- **Prior external review findings still open**: 0 — first external review for this change.
- **Resolved since last review**: see `address-reviews-2026-05-26T23-37-11Z.json` for the 5+2 walk; in particular: phantom `# Internal Review:` markdown header removed (CRITICAL), README scope captured in change Impact + task 1.5 (WARN 1), empty/missing `findings[]` covered by 2 new spec scenarios (WARN 2).

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What this change does (one-paragraph)

Closes [GitHub openspec-orbit#4](https://github.com/las-sal/openspec-orbit/issues/4). Extends `/opsx:address-reviews --from-file <path>` to auto-detect format (markdown vs JSON) and ingest internal review JSON (`openspec/changes/<name>/.orbit-runs/review-<mode>-<TS>.json`) as virtual markers. V1 accepts `command: "review"` JSON only; other internal JSONs (`audit-drift`, `address-reviews`, `apply`, `archive`, `propose`) rejected with a clean error. New reference doc `references/internal-findings-format.md` symmetric with existing `external-findings-format.md`. Markdown ingest path is byte-identical from the user's perspective. Spec delta: 1 capability (`orbit-address-reviews`) — 2 MODIFIED requirements with 13 scenarios total. Implementation: SKILL prose + opsx command file mirror + 2 reference docs + README updates at 2 sites; no CLI flag changes; no code behavior changes outside the address-reviews skill prose.

## What to read for THIS review (`--as system`)

```
- openspec/changes/address-reviews-accepts-internal-json/proposal.md
- openspec/changes/address-reviews-accepts-internal-json/design.md
- openspec/changes/address-reviews-accepts-internal-json/tasks.md (13/13 boxes checked)
- openspec/changes/address-reviews-accepts-internal-json/specs/orbit-address-reviews/spec.md (2 MODIFIED requirements, 13 scenarios)
- git diff from main since ba7cd8e^ — the change's full commit range
- openspec/specs/orbit-address-reviews/spec.md (archived baseline — note still has the OLD 2-requirements framing pre-sync; MODIFIED resolves on archive via sync-specs)
- openspec/specs/orbit-conventions/spec.md (universal spine for the JSON shape)
- openspec/specs/orbit-review/spec.md (the run-summary-schema source-of-truth that the new internal-findings-format references)
- .claude/skills/openspec-address-reviews/SKILL.md (the rewritten --from-file ingest section + 2 worked examples + expanded graceful-degradation)
- .claude/skills/openspec-address-reviews/references/internal-findings-format.md (NEW; primary parser-contract doc)
- .claude/skills/openspec-address-reviews/references/external-findings-format.md (cross-reference at top)
- .claude/commands/opsx/address-reviews.md (mirror of SKILL changes)
- .claude/skills/openspec-review/references/run-summary-schema.md (the canonical JSON shape the parser ingests)
- README.md (--from-file flag description L422 + new "Internal review JSON findings format" sub-header L771)
```

## What to look for (system-mode passes)

0. **verify-change-style structural check** — 13/13 tasks checked, spec coverage, design adhered. Run your own equivalent of upstream verify-change.
1. **Baseline Compliance** — does this change break any archived `openspec/specs/` requirement? Note: orbit-address-reviews baseline still describes the OLD 2-requirements framing pre-sync; the MODIFIED in the change delta resolves this on archive. Is the in-flight divergence acceptable, or is there a baseline contract being violated?
2. **Cohesion** — the change touched `.claude/skills/openspec-address-reviews/SKILL.md`, `.claude/commands/opsx/address-reviews.md`, 2 reference docs, and `README.md` at 2 sites. Are there callers/dependents outside these files that should be updated? (e.g., other SKILL files that reference `--from-file` semantics; orbit-status code that parses run-summary JSONs and might need updating if the JSON shape changed; consumers of `external-findings-format.md` that should also know about `internal-findings-format.md`).
3. **Surface Walk** — `/opsx:address-reviews --from-file <path>` CLI surface: same name, same flag, same `--keep-resolved-markers` accompaniment. Behavior contract: previously markdown-only, now markdown OR JSON via content sniff. Is this a breaking change for any consumer? (No — the markdown path is byte-identical; new path is additive. But verify.) Are the new graceful-degradation error paths clear enough that a confused user can self-diagnose?
4. **Perspective Reviews** — N/A (no lenses defined).
5. **Critical-Path Scan** — N/A (no lenses defined).
6. **Drift / Residue** — vocabulary residue, stale references, archive-coherence misses. Specifically: does any file still describe `--from-file` as markdown-only? Are the 2 worked examples (markdown + JSON) labeled clearly so users don't confuse them? Does the directory-tree comment in README accurately list both reference docs? Are there any leftover `# Internal Review:` references (the proposal-iter-1 CRITICAL fix needs to hold)?

**Specific things worth scrutiny on this change** (areas I want second-opinion validation):

- **Content-sniff heuristic robustness**. Design D-detect-1 uses leading-character sniff (leading `{` → JSON; leading `# External Review:` → markdown). What about edge cases? Files with BOM markers, leading whitespace beyond the spec's "first non-whitespace character" rule, Windows CRLF line endings, files with `<!-- comments -->` at the top before the title. The spec at `specs/orbit-address-reviews/spec.md` lines 15+25+34 references "first non-whitespace token" — is that precise enough that two implementers would produce the same parser?
- **Parser-contract precision in internal-findings-format.md**. Is the parser contract (field-mapping table at L31-37 + strict/lenient split at L73-92) unambiguous? Could a fresh AI implementer read just the reference doc + design.md and produce a correct parser without asking questions?
- **V1 scope discipline (`command: "review"` only)**. The design rejects `audit-drift` JSON ingest as out of scope. But audit-drift's JSON has the same `findings[]` shape — accepting it would be ~1 line. Is the deliberate exclusion principled, or is it scope-discipline-theater?
- **Fresh-pushback semantics for JSON virtual markers**. The MODIFIED `Pushback discipline applied per marker` requirement adds 1 scenario (line 71-74 of the delta spec) saying fresh pushback IS applied to JSON virtual markers even though the JSON's own `stale_suppressed[]` already filtered at review time. Is this double-pushback the right call, or should the parser respect the JSON's prior filter? Compare against the cost-up-front principle.
- **Empty `findings[]` is NOT an error**. The new scenario at spec line 32-35 says an empty `findings: []` JSON should succeed with zero virtual markers and an informational resolution log. Is the "informational not error" framing correct? Could a user be confused by running `--from-file` on a clean-pass JSON and getting an apparent no-op?
- **Two adjacent worked examples (markdown + JSON) in SKILL.md** — does the labeling distinguish them clearly enough? The markdown example header now reads `--from-file markdown ingest` and the JSON example reads `--from-file JSON ingest`. Worth verifying.

## Output format — write to:

`openspec/changes/address-reviews-accepts-internal-json/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format with colons replaced — `YYYY-MM-DDTHH-MM-SSZ`. Pick a fresh timestamp so this file doesn't overwrite any prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: address-reviews-accepts-internal-json (iteration 1, system mode)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

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

<Optional: overall impression, broader concerns. Omit the whole `## Notes` section if you have nothing to add.>
```

If your environment doesn't support file writes (chat-only interface), output the markdown directly and the user will save it.

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/address-reviews-accepts-internal-json/.orbit-runs/external-system-<TS>.md
   git commit -m "External review (system, iter 1): address-reviews-accepts-internal-json

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
