# External Review: address-reviews-auto-discovers-internal-json (iteration 1, proposal mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

This is a **proposal-mode** review (pre-apply): the change has been proposed and reviewed internally; you are verifying the artifacts (proposal, design, spec deltas, tasks) before any implementation work happens.

## Repo

https://github.com/las-sal/openspec-orbit

## Project context (read first)

- `CLAUDE.md` — handoff orientation for openspec-orbit
- `openspec/project.md` — project goals + stack (if present)
- `*_convention.md` at repo root (if any — orbit currently has none)
- `openspec/lenses/perspectives.md` — does NOT exist for orbit; out of scope for proposal-mode passes
- `openspec/lenses/critical-paths.md` — does NOT exist for orbit
- `openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/` — iteration history (4 files: propose JSON + internal review JSON + address-reviews JSON + this external prompt)

## Cycle context

- **Iteration**: 1 (FIRST external review for this change in either mode)
- **Prior internal findings still open**: 0
  - Internal proposal iter-1 `--fresh`: 0 critical / 2 warning / 3 suggestion → all 5 resolved via inline-walk (the meta-irony pattern: this change is exactly what closes the workaround). Resolution log at `address-reviews-2026-05-27T01-04-52Z.json`.
- **Prior external findings**: none — first external review
- **Resolved since last review (internal iter-1)**: 5 findings — count drift across artifacts reconciled; markers-precede-discovery rule codified in spec intro; TS-collision tie-break specified; path-vs-name disambiguation rule added to spec intro; task 2.4 broadened to cover both schema enum + field-note prose

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What this change does (one-paragraph)

Closes [GitHub openspec-orbit#10](https://github.com/las-sal/openspec-orbit/issues/10). Adds auto-discovery layer on top of the just-archived `address-reviews-accepts-internal-json` (closes #4) parser machinery. `/opsx:address-reviews <change-name>` now auto-discovers the most-recent `review-<mode>-*.json` OR `audit-drift-*.json` in the change's `.orbit-runs/` directory when no `@review:` markers are found in the change directory AND no `--from-file` flag is specified. Strictly additive: whole-repo invocation, path/pattern scope, explicit `--from-file`, and `--mark` usage all unchanged. Discovery rule (D-recency-1, Option A): filename `<TS>` token, lexical sort, single global most-recent across both candidate types — review and audit-drift JSON compete on the same recency axis with no class preference. Identical `<TS>` token collision resolved via stable lexicographic sort. Discovery does NOT run for bare invocations OR path/pattern scope (no `.orbit-runs/` anchor) OR explicit `--from-file` (user intent wins). Spec delta has 1 MODIFIED requirement with 10 scenarios (2 existing-equivalent + 8 net-new = 1 happy path + 7 edge cases). Implementation: SKILL prose + opsx command file mirror + 1 reference doc note + README update + onboard command-table row; no CLI flag changes; no executable code (this is documentation-driven behavior in the address-reviews skill).

## What to read for THIS review (`--as proposal`)

```
- openspec/changes/address-reviews-auto-discovers-internal-json/proposal.md
- openspec/changes/address-reviews-auto-discovers-internal-json/design.md (6 design decisions)
- openspec/changes/address-reviews-auto-discovers-internal-json/tasks.md (3 chunks, 13 tasks)
- openspec/changes/address-reviews-auto-discovers-internal-json/specs/orbit-address-reviews/spec.md (1 MODIFIED requirement, 10 scenarios; 2 normative paragraphs in requirement intro)
- openspec/specs/orbit-address-reviews/spec.md (CURRENT baseline — the MODIFIED target lives at L9-21 — `Address-reviews command available` requirement with 2 scenarios; this change adds 8 net-new scenarios and 2 normative intro blocks)
- openspec/specs/orbit-address-reviews/spec.md (broader baseline — includes the `--from-file ingest of review findings file` requirement at L23 that auto-discovery feeds INTO; sync from the just-archived #4)
- openspec/changes/archive/2026-05-26-address-reviews-accepts-internal-json/ (the just-archived parent change — this change orchestrates on top of its parser machinery)
- .claude/skills/openspec-address-reviews/SKILL.md (the file the change will modify — Step 1 Discover section is the primary target)
- .claude/skills/openspec-address-reviews/references/internal-findings-format.md (parser contract referenced by spec scenarios — auto-discovery feeds JSON through this same parser)
- .claude/skills/openspec-address-reviews/references/run-summary-schema.md (the `source` field enum to be extended — task 2.4)
- README.md (~L417 area `/opsx:address-reviews` command-reference description — task 2.2)
```

## What to look for (proposal-mode passes — 9)

1. **Structure & Delta Integrity** — artifacts present; MODIFIED operation valid; `openspec validate --strict` would pass.
2. **Internal Coherence** — proposal aligns with design aligns with spec aligns with tasks. Count consistency. No scope creep beyond #10.
3. **Cross-Doc Coherence** — `CLAUDE.md`, baseline specs, the just-archived #4 work all stay consistent post-this-change.
4. **Archive Consistency** — the MODIFIED `Address-reviews command available` requirement consistent with the synced baseline + the `--from-file ingest of review findings file` requirement it references in scenarios. The just-archived #4 SHALL contracts remain honored.
5. **Codegen Readiness** — could a fresh AI implementer read the spec + design + reference docs and produce a correct implementation without asking questions? Specifically:
   - Is the discovery priority order unambiguous? (markers → JSON → empty)
   - Is the recency rule precise? (filename `<TS>` token, lexical sort, with explicit tie-break for TS collisions)
   - Is the change-name-vs-path resolution rule unambiguous? (directory existence at `openspec/changes/<name>/` or archive equivalent)
   - Is the `source: "auto-discovered"` enum extension scope clear?
6. **Gap Hunt** — could a fresh AI implementer miss edge cases? Specifically:
   - What happens when discovery finds a candidate JSON but the parser fails (parse failure, missing findings[], wrong command)? Spec doesn't explicitly cover the parser-failure-after-discovery path — does it implicitly fall through to the existing parser malformed-input scenarios?
   - What happens when the change directory exists but `.orbit-runs/` doesn't? (Scenario "no candidate JSON" covers it but verify.)
   - What if the user invokes `/opsx:address-reviews <change-name>` on an archived change with a 6-month-old JSON — does discovery still pick it? (Spec says yes; D-no-stale-detection codifies the don't-detect-stale design choice. Is this defensible?)
7. **Drift Hunt** — old vocabulary lingering? Specifically:
   - Does the change still describe `--mark` as "the way to mark findings for address-reviews" anywhere it's not been updated?
   - Are there leftover references to the v1 #4 framing ("v1 accepts only review JSON") that should be updated to reflect both supported commands?
8. **Inline Review Marker Residue** — any `@review:` markers still present in the change directory?
9. **Pre-Handoff Sweep** — anything else worth flagging? Specifically: the proposal.md "Subsumes #4" framing — is it accurately scoped? (#4 added the parser; #10 adds the discovery layer; both can coexist as separate archives.)

**Specific scrutiny areas** (areas I want second-opinion validation):

- **The just-archived parent's existence**. This change orchestrates on top of `--from-file ingest of review findings file` from the just-archived #4 archive. Cross-archive coherence: does the auto-discovery scenario "Auto-discovered JSON walks the standard ingest lifecycle" correctly route through the parent's parser contract? Should there be a more explicit cross-reference to the parent baseline requirement, or is the current "(per the `--from-file ingest of review findings file` requirement's existing parser contract)" reference sufficient?
- **The 2 normative paragraphs in the requirement intro**. They codify two rules (Discovery priority order + Positional argument resolution) that the scenarios depend on. Are they correctly placed in the intro (rather than as scenarios themselves)? Is the intro location stable across openspec spec-rendering conventions?
- **`source: "auto-discovered"` enum addition**. The change extends the address-reviews JSON's `source` enum at the reference-doc level. The reference doc is `references/run-summary-schema.md` (documentary), not a normative spec. Is documenting this enum extension in the reference doc + task 2.4 sufficient, or should the spec also normatively declare the new enum value?
- **D-no-stale-detection trade-off**. The change deliberately doesn't detect stale JSON at discovery time (delegates to lifecycle pushback). The just-archived `harden-review-mode-recommendations` has stale-detection logic for system-mode review's State 5. Is the asymmetry defensible (review-skill-detects-stale vs address-reviews-doesn't)?
- **`<change-name>` resolution in the archive case**. The "Auto-discovery respects archive location" scenario says discovery looks in `openspec/changes/archive/<YYYY-MM-DD>-<name>/.orbit-runs/`. But the user types `/opsx:address-reviews <change-name>` — they don't include the date prefix. How does the resolver disambiguate? (Implicitly: scan both `openspec/changes/<name>/` AND `openspec/changes/archive/*/<name>/` paths.) Spec doesn't specify the search order. Codegen-readiness concern.

## Output format — write to:

`openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/external-proposal-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format with colons replaced — `YYYY-MM-DDTHH-MM-SSZ`. Pick a fresh timestamp so this file doesn't overwrite any prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: address-reviews-auto-discovers-internal-json (iteration 1, proposal mode)

**Reviewer**: <your model name>
**Date**: <YYYY-MM-DD>

## CRITICAL

### <Finding title>
**File**: <path>:<line>
**Description**: <what's wrong + specific recommendation>

(For each additional CRITICAL finding, repeat. Use `None.` as a single-word body if no findings.)

## WARNING

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(Same shape. Use `None.` if no findings.)

## SUGGESTION

### <Finding title>
**File**: <path>:<line>
**Description**: <text>

(Same shape. Use `None.` if no findings.)

## Notes

<Optional: overall impression, broader concerns. Omit if you have nothing to add.>
```

## After completing the review

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section, every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/address-reviews-auto-discovers-internal-json/.orbit-runs/external-proposal-<TS>.md
   git commit -m "External review (proposal, iter 1): address-reviews-auto-discovers-internal-json

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat and the user will commit it manually.
