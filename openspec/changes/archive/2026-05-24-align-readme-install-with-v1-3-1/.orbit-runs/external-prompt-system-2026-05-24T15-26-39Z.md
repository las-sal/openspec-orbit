# External Review: align-readme-install-with-v1-3-1 (iteration 1, system mode)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone the `main` branch; the change directory + edits are on a head commit — confirm latest is what you're reviewing).

## Project context (read first)

- `CLAUDE.md` — handoff orientation for the openspec-orbit repo. Defines orbit as a workflow tool that owns `.claude/` and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine.
- `README.md` — user-facing documentation. **This change rewrites the Installation section (currently roughly L898 through the end of `### Uninstalling`).** That rewrite IS the substantive deliverable of this change.
- `openspec/changes/align-readme-install-with-v1-3-1/.orbit-runs/` — iteration history. Includes propose JSON (T0), 2 review-proposal JSONs (iter-1 in-context + iter-2 fresh subagent), 1 address-reviews JSON (walked iter-1 findings), 2 apply chunk-end JSONs (chunk 1 + chunk 2).
- `openspec/specs/orbit-conventions/spec.md` — baseline orbit-conventions requirements. This change ADDs one new requirement (`Install documentation describes actual install surface`) via the delta at `openspec/changes/align-readme-install-with-v1-3-1/specs/orbit-conventions/spec.md`. The ADDED requirement is NOT yet synced to baseline — sync happens on archive. For your review, consider both the baseline as-is AND what it will become after archive sync.
- `openspec/lenses/` — does not exist yet for this repo (lens layer is empty; perspective + critical-path reviews will skip gracefully per the change's design).
- `openspec/project.md` — does not exist; not used by this repo.

## Cycle context

- **Iteration**: 1 (first external review for this change in any mode).
- **Prior internal findings still open**: 0. Review-proposal iter-2 (fresh subagent) confirmed zero new findings; iter-1's 4 findings were resolved, 2 declined, 1 out-of-scope.
- **Prior external findings still open**: N/A (first external review).
- **Resolved since last review**: see `openspec/changes/align-readme-install-with-v1-3-1/.orbit-runs/address-reviews-2026-05-24T15-02-20Z.json` for the 7 iter-1 findings walked. Also note: apply chunk 2's sandbox verification surfaced an inline-fixed premise problem — README count claims were 14+14, actual reality is 15+16; corrected mid-chunk with sandbox re-verification.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as system)

Mandatory:

- `openspec/changes/align-readme-install-with-v1-3-1/proposal.md` — Why + What Changes + Capabilities + Impact
- `openspec/changes/align-readme-install-with-v1-3-1/design.md` — 7 decisions (D-arch-1..3, D-pegging-anchor, D-conventions-1, D-verification-1..3) + alternatives + risks
- `openspec/changes/align-readme-install-with-v1-3-1/tasks.md` — 3 chunks of work; chunks 1+2 complete (13 of 16 tasks done); chunk 3 in user-validation handoff (1 of 3 done — task 3.1 validate; 3.2 + 3.3 are user-validation pending)
- `openspec/changes/align-readme-install-with-v1-3-1/specs/orbit-conventions/spec.md` — 1 ADDED requirement with 5 scenarios (codifies the README-matches-install-surface rule)
- `openspec/changes/align-readme-install-with-v1-3-1/explore.md` — historical record from explore phase (D1-D7 decisions, including the inline-sandbox-resolved Q1/Q2/Q4 + the user-declined Q3)

**The actual product state being reviewed**:

- `README.md` — read the entire Installation section. Section starts at `## Installation` (around L898) and runs through `### Uninstalling / reverting to upstream-only` (around L1040 after this change's insertions). This is the material change. Sub-sections in order: `### Prerequisites` → `### Pegging strategy` (new) → `### 1. Initialize upstream OpenSpec` → `### 2. Overlay orbit` → `### 3. (Recommended) Add the orbit discipline reminder to your CLAUDE.md` → `### What you should see after install` → `### Partial adoption is fine` → `### Working alongside existing .claude/ customizations` → `### Common gotchas` → `### Updating orbit` → `### Uninstalling / reverting to upstream-only`.
- The 3 forward-references to `[Pegging strategy](#pegging-strategy)` should now all resolve (previously dead anchors). Locations: README L54 (intro), L902 (install opening), idempotent-overlay note around L960. Additional references added by this change: one in Prerequisites (around L907) and one in Common gotchas. Verify each resolves to a real heading.

**Baseline + archive context**:

- `openspec/specs/orbit-conventions/spec.md` — current baseline. Note the recently-synced requirements (post-2026-05-24 archive of `lean-overlay-and-add-orbit-onboard`): `Distribution model — pegged engine + orbit-owned surface`, `Upstream version pinning`, `Overlay file disposition`. The new ADDED requirement from THIS change should compose cleanly with these.
- `openspec/changes/archive/2026-05-24-lean-overlay-and-add-orbit-onboard/` — the immediately prior archived change, of which this change is a follow-up. Read its proposal.md + design.md if you want context on why this change exists (it picks up the deferred chunks 3-5 README rewrite).
- `openspec/changes/archive/2026-05-21-emit-run-summary-jsons-from-workflow-commands/` — the change that added `# Orbit additions` to 7 more skills (making the original README's "3 modified skills" claim stale). Relevant if you're checking the orbit-modified skill count framing.

**Code/file paths touched by the change**:

- `README.md` (one section)
- That's it. This is a doc-only change with one spec delta. No skill rewrites, no command rewrites, no .claude/ overlay changes, no openspec/specs/ changes (the delta lives in the change directory, syncs to baseline only on archive).

**Sandbox-verification facts you should know** (so you can spot if the README contradicts them):

- `npx -y @fission-ai/openspec@1.3.1 init --tools claude` installs 10 skills + 10 commands. Bare `init` (no `--tools` flag) errors non-interactively.
- `openspec init --force --tools claude` overwrites upstream-installed skills/commands but preserves user-created files (e.g., `.claude/custom/*`) and all `openspec/` content. Sandbox-verified end-to-end via md5 hash check + file-existence checks.
- `cp -r` from orbit's overlay onto a fresh upstream-init state produces 15 skills + 16 commands (after `rm -rf .claude/skills/feedback` prune). The 15 skills = 10 upstream-modified + 4 orbit-authored + 1 upstream-required primitive (`openspec-sync-specs`). The 16 commands = 9 upstream-modified (the 10 minus ff.md) + 1 upstream-untouched (ff.md, retained because orbit ships `fast-forward.md` instead) + 4 orbit-authored + 2 additional orbit (`sync.md`, `fast-forward.md`).
- `git tag -l 'v0.1*'` returns `v0.1.0` — orbit's version-tag reference at the README's "Updating orbit" subsection is valid.

## What to look for (7 system-mode passes)

0. **verify-change-style structural check** — tasks done, spec coverage, design adhered. Run your equivalent of upstream `verify-change`. Note: 2 tasks (3.2, 3.3) are user-validation handoffs — they're informational and expected to be unchecked at this stage of apply. The other 14 should be done.

1. **Baseline Compliance** — does this change break any archived `openspec/specs/` requirement? Specifically check:
   - `Distribution model — pegged engine + orbit-owned surface` (orbit-conventions L602) — does the README rewrite contradict any scenario?
   - `Upstream version pinning` (orbit-conventions L626) — does the README's `### Pegging strategy` subsection align with all 5 scenarios of this requirement?
   - `Overlay file disposition` (orbit-conventions L655) — note this baseline has known pre-existing drift (lists 9 orbit-modified skills, actual is 10) per the just-archived change's review iter-1 finding 7 — but that's NOT this change's problem to fix. Flag it only if the README contradicts the baseline in some other way.

2. **Cohesion** — callers/dependents outside the tasks list. The README's claims about file counts, CLI invocations, and overlay disposition need to compose with how other parts of the orbit codebase describe themselves (CLAUDE.md narrative, the orbit-conventions spec, skill content). Spot signature drift, ripple effects, contradictions.

3. **Surface Walk** — every CLI / MCP / HTTP / public-function surface still coherent? For this change: the README documents specific CLI invocations (`openspec init --tools claude`, `cp -r`, `rm -rf .claude/skills/feedback`, `openspec init --force --tools claude`, `npx @fission-ai/openspec@1.3.1 <subcommand>`, `grep -l "^# Orbit additions" .claude/skills/openspec-*/SKILL.md`, `ls .claude/skills/ | wc -l`). Each should be executable as written. The README's enumerated file lists (10 orbit-modified skills, 4 orbit-authored, etc.) should match what a fresh sandbox install actually produces.

4. **Perspective Reviews** — `openspec/lenses/perspectives.md` does not exist for this repo. Skip Pass 4 with the note `no perspectives defined; skip Pass 4` (per the orbit-review skill's graceful-degradation rule).

5. **Critical-Path Scan** — `openspec/lenses/critical-paths.md` does not exist. Skip Pass 5 similarly.

6. **Drift / Residue** — old vocabulary still lingering anywhere the rewrite touched? Spot residue from pre-pegging or pre-emit-change framings: "11 upstream skills", "the other 8 upstream", "three upstream skill files", "(plus the `feedback` skill)", "Plus `feedback/` separately", "modifies three", anywhere in the README or in the change's own artifacts (proposal/design/tasks/specs). The authoring AI's iter-1 + iter-2 reviews + address-reviews walk should have cleared these from the change artifacts; you're double-checking.

## Output format — write to:

`openspec/changes/align-readme-install-with-v1-3-1/.orbit-runs/external-system-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format — pick a fresh one so this file doesn't overwrite prior reviews.)

Use this exact markdown structure:

```markdown
# External Review: align-readme-install-with-v1-3-1 (iteration 1)

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

1. **Output the full findings markdown in chat** — in addition to writing the findings file, output the COMPLETE findings markdown in this chat. Same content as the file: every severity section (`## CRITICAL` / `## WARNING` / `## SUGGESTION`), every `### Title` entry, every `**File**:` and `**Description**:` field. Do NOT abbreviate or summarize — the chat output is the immediately-visible read for the user (they should be able to evaluate every finding without opening the file). The file remains the canonical record for `--from-file` parsing.

2. **Commit and push the findings file** (if your environment supports git):

   ```bash
   git add openspec/changes/align-readme-install-with-v1-3-1/.orbit-runs/external-system-<TS>.md
   git commit -m "External review (system, iter 1): align-readme-install-with-v1-3-1

   <one-line summary: severity counts + headline finding if any>"
   git push
   ```

If you don't have git access, just output the findings markdown in this chat (per the chat-only fallback above) and the user will commit it manually.
