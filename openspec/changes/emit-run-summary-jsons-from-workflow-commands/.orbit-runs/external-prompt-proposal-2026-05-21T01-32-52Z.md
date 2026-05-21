# External Review: emit-run-summary-jsons-from-workflow-commands (iteration 1)

You are reviewing an OpenSpec change as a second pair of eyes. Your value is your independent take — be thorough; flag anything that looks wrong, inconsistent, or unclear. Don't be charitable to the authoring AI's reasoning.

## Repo

https://github.com/las-sal/openspec-orbit (clone fresh; the change is on `main` at commit `874e6d1` or later)

## Project context (read first)

- `CLAUDE.md` — orbit's authoring conventions (three execution disciplines: read-before-reference / change-completeness / pushback)
- `README.md` — orbit's user-facing surface and install model (overlay-on-upstream-openspec)
- `openspec/specs/orbit-conventions/spec.md` — cross-cutting orbit conventions (especially `Requirement: Internal-run JSON summary format` at line 159, which this change MODIFIES)
- `openspec/specs/orbit-*/spec.md` — other established orbit capability specs
- `openspec/changes/archive/2026-05-18-bootstrap-openspec-orbit/` — the prior change that established orbit-conventions, the 3 per-skill schema reference files, and the editorial-command emit pattern (prior art for this change)
- `.claude/skills/openspec-<skill>/references/run-summary-schema.md` for `openspec-review`, `openspec-audit-drift`, `openspec-address-reviews` — the 3 existing per-skill schema references this change interacts with
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/` — iteration history; see specifically:
  - `review-proposal-2026-05-21T00-18-14Z.json` (internal review iter-1: 19 findings)
  - `address-reviews-2026-05-21T01-29-34Z.json` (resolution log for those 19 findings; see what was actually changed)

## Cycle context

- **Iteration**: 1 (first external review for this change)
- **Prior internal findings still open**: 0 (all 19 from internal review iter-1 were resolved during the address-reviews walk; see `address-reviews-2026-05-21T01-29-34Z.json` for the per-finding resolutions)
- **Prior external findings still open**: 0 (this is the first external pass)
- **Resolved since last review**: 19 internal findings — 5 CRITICAL (C1–C5), 7 WARNING (W1–W7), 7 SUGGESTION (S1–S7). Notable: C1's resolution unified `orbit-conventions`'s `Internal-run JSON summary format` requirement to define a universal spine across workflow/editorial/lifecycle kinds, adding `orbit-conventions` to the change's Modified Capabilities (the original proposal had "no modified capabilities").

**Authoring/internal-reviewer context (for cross-AI anchoring awareness)**: The proposal artifacts and the internal review iter-1 were both authored by Claude Opus 4.7 in a single in-context session. The orbit-status retrospective evidence (`openspec-orbit#20`) documented that in-context system review missed 3/3 real bugs that external review caught; while this is proposal mode (less anchored than system mode), the same pattern of anchoring may have suppressed findings. Your value as a fresh-context reviewer is exactly that — flag what the in-context AI might have anchored away from.

Do not push back on stale findings — pushback discipline is enforced on resolution, not review. Just flag what you observe.

## What to read for THIS review (--as proposal)

- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/proposal.md`
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/design.md`
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/tasks.md`
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-run-summary-emit/spec.md` (new capability)
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/specs/orbit-conventions/spec.md` (MODIFIED capability — the unified spine work)
- `openspec/changes/emit-run-summary-jsons-from-workflow-commands/explore.md` (decision history with 15 captured decisions)
- `openspec/specs/orbit-conventions/spec.md` (archived baseline — to check the MODIFIED delta against)
- `openspec/specs/orbit-*/spec.md` (other archived capabilities, esp. those whose schema refs are mentioned in tasks 1.2 and 1.4: `orbit-review`, `orbit-audit-drift`, `orbit-address-reviews`, `orbit-archive-modifications`)

## What to look for

Apply ALL 9 proposal-mode passes. For each finding you raise, cite the file + line + a specific recommendation. Don't be charitable.

1. **Structure & Delta Integrity** — required artifacts present; ADDED/MODIFIED/REMOVED/RENAMED operations used correctly; `openspec validate --strict` would pass. Check: does the orbit-conventions MODIFIED delta correctly copy the full original requirement before editing (per the MODIFIED workflow rule)?

2. **Internal Coherence** — proposal aligns with design aligns with specs aligns with tasks. Count/label consistency. Watch for: claims in proposal/design vs. what specs actually require; capability lists in proposal vs. spec dirs present; tasks references match requirement names.

3. **Cross-Doc Coherence** — `CLAUDE.md`, conventions, `*_convention.md` (none at repo root currently), and the 3 existing per-skill `run-summary-schema.md` reference files. After this change lands, are those reference files coherent with the new universal spine? Tasks 1.2 and 1.4 say they'll be updated during apply — are the changes well-specified?

4. **Archive Consistency** — the MODIFIED orbit-conventions requirement: does it correctly extend the archived baseline at `openspec/specs/orbit-conventions/spec.md:159`? Does it preserve the existing required minimum fields while ADDING per-kind extensions, or does it weaken/break existing guarantees? Check the bootstrap archive at `openspec/changes/archive/2026-05-18-bootstrap-openspec-orbit/` for prior emit-architecture decisions that might contradict this change.

5. **Codegen Readiness** — could a fresh AI implement this change from these specs alone? Specifically check:
   - The Universal emit shape / Workflow-kind emit shape requirements — are the field types crisp enough?
   - The Emit timing semantics requirement (added late in the address-reviews walk) — are the "conversation boundary" signals concrete enough to implement reliably?
   - The Apply per-chunk-end emission rules — is the tasks.md preamble parsing format unambiguous?
   - The Verify fail-mode classification — can an emit-layer reliably distinguish modes ①/②/③ from verify-change's output?
   - The bare-mode explore crystallization warning — is the warning text in the spec verbatim enough for an AI to surface consistently?

6. **Gap Hunt** — what's missing?
   - Are all 9 in-scope commands actually covered with concrete requirements (or only some)?
   - Error paths: what happens if the emit-layer fails to write (filesystem error)?
   - Concurrency: what if two emits race?
   - Existing emit-producing commands gaining `kind` — is the migration path crisp for `kind`-less legacy emits already in production `.orbit-runs/` dirs of other archived changes?
   - Multi-change projects: do bare-mode-explore consequences (the "becomes visible to orbit-status" warning) actually hold when `openspec list` is project-wide?

7. **Drift Hunt** — old vocabulary lingering:
   - "emit JSON" vs "run-summary JSON" — was the W7 standardization actually completed across all artifacts?
   - "Internal-run JSON summary format" (orbit-conventions title) vs other usages — is the cross-referencing consistent?
   - The 3 per-skill reference files use "<command> run-summary schema" — does the change harmonize cleanly with that terminology?

8. **Inline Review Marker Residue** — any `@review:` markers in the change directory? (Should be zero — this is post-address-reviews state.)

9. **Pre-Handoff Sweep** — anything else before this ships to apply?
   - Scope check: 14 deltas (13 ADDED in orbit-run-summary-emit + 1 MODIFIED in orbit-conventions). Above OpenSpec's "consider splitting at >10" soft threshold. Is the bundling justified, or should this split (e.g., spine work separately from per-command behaviors)?
   - The address-reviews walk added a `Requirement: Emit timing semantics` mid-walk that wasn't in the original explore decisions — is it well-integrated?
   - The change introduces a `bridge` markdown findings file (`review-proposal-...-as-findings.md`) committed in `.orbit-runs/` because `address-reviews --from-file` doesn't accept JSON yet (`openspec-orbit#4`). Is that bridge file appropriately scoped, or does it pollute the audit trail?

## Output format — write to:

`openspec/changes/emit-run-summary-jsons-from-workflow-commands/.orbit-runs/external-proposal-<TS>.md`

(Where `<TS>` is today's timestamp in ISO format like `2026-05-21T02-15-00Z`. Pick a fresh timestamp so this file doesn't overwrite prior reviews.)

Use this exact markdown structure (the section headers + field labels are parsed by `/opsx:address-reviews --from-file`; deviating breaks ingestion):

```markdown
# External Review: emit-run-summary-jsons-from-workflow-commands (iteration 1)

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

After writing the file, mention the file path in your final message so the user can find it for the `/opsx:address-reviews --from-file` step.
