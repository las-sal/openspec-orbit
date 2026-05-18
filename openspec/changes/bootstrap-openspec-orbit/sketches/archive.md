# Sketch: `/opsx:archive` modifications

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — preserves upstream behavior; adds a pre-archive sweep hook), principle 2 (cost up front — pay for the audit before archiving, not after).

## What stays the same

Upstream's `/opsx:archive` does the work of moving an applied change into the historical archive:

- Moves `openspec/changes/<name>/` → `openspec/changes/archive/<name>/`
- Runs `sync-specs` to merge delta specs into the canonical `openspec/specs/<capability>/spec.md`
- Marks the change as archived

This base behavior is preserved. Orbit's modifications add a pre-archive sweep that happens *before* the move.

## What changes

Two additions:

1. **Pre-archive drift audit** — `/opsx:archive` auto-invokes `/opsx:audit-drift` before completing. Findings are surfaced in the archive output. If CRITICAL drift is found, the user is prompted (not blocked) to address before archiving.
2. **Archive run summary** — writes a small JSON to `.orbit-runs/archive-<TS>.json` capturing the archive event, audit findings summary, and the user's decision (proceeded / addressed first / aborted). Provides closure on the iteration history.

## The pre-archive audit hook

```
/opsx:archive <change-name>
        │
        ▼
   1. Validate change is ready (upstream checks)
        │
        ▼
   2. Run /opsx:audit-drift (library call)
        │  Same as /opsx:review --as system Pass 6's
        │  composition — internal invocation, full
        │  output folded into archive's report.
        │
        ▼
   3. Drift findings present?
        │
        ├── No CRITICAL → proceed silently (warnings logged)
        │
        └── ≥1 CRITICAL → prompt user via AskUserQuestion:
              "X critical drift issue(s) found. Address now,
               proceed with archive, or abort?"

                Options:
                ─────────
                a. Address now (Recommended) — show findings;
                   user fixes; archive does NOT proceed.
                   User can re-invoke archive after fixing.

                b. Proceed with archive — known drift is
                   acceptable; document why in the archive
                   run summary.

                c. Abort — cancel the archive entirely.
        │
        ▼
   4. (If proceeding) Standard upstream archive:
      - Move change dir to archive
      - Run sync-specs to merge deltas
      - Mark archived
        │
        ▼
   5. Write archive run summary to
      openspec/changes/archive/<name>/.orbit-runs/
      archive-<TS>.json
        │
        ▼
   6. Report completion
```

**Not a hard gate**. User confirms even with CRITICAL findings. Reason: users may legitimately archive with known drift (e.g., a follow-up commit is planned to fix the residue). orbit captures the *decision* in the archive run summary so it's traceable.

## The `--skip-audit` flag

```
/opsx:archive <change-name> --skip-audit
```

Bypasses Step 2 entirely. Used when:
- The user already ran `/opsx:audit-drift` separately and addressed everything
- The user is in a hurry and willing to accept the risk
- CI / automated archive paths where the audit was a separate step

`--skip-audit` is logged in the archive run summary (so "did we audit this archive?" remains traceable).

## Archive run summary format

Written to `openspec/changes/archive/<name>/.orbit-runs/archive-<TS>.json` after the archive completes:

```json
{
  "command": "archive",
  "timestamp": "2026-05-18T11:45:00Z",
  "change": "foo",
  "audit": {
    "ran": true,
    "skipped_via_flag": false,
    "findings_summary": {
      "critical": 0, "warning": 2, "suggestion": 1
    }
  },
  "user_decision": "proceeded_with_no_critical",
  "sync_specs": {
    "capabilities_updated": ["host-lifecycle", "matrix-config"],
    "added": 3, "modified": 2, "removed": 1, "renamed": 0
  }
}
```

Possible `user_decision` values: `proceeded_with_no_critical`, `proceeded_despite_critical`, `aborted`, `audit_skipped_via_flag`. Provides closure on the iteration history that `.orbit-runs/` tracks for each change.

The `.orbit-runs/` directory **moves with the change** during archive (from `openspec/changes/<name>/.orbit-runs/` to `openspec/changes/archive/<name>/.orbit-runs/`). All prior internal-run and external-review summaries persist as archived history.

## Edge cases

| Case | Handling |
|---|---|
| `audit-drift` fails to run (parse error, internal exception) | Warn, but allow archive to continue. Log the failure in the archive run summary so traceability is preserved. |
| `audit-drift` finds CRITICAL drift but `--skip-audit` is set | The flag wins — archive proceeds. Summary still records that the audit was skipped. |
| Change already archived | Halt with clear error: "Change <name> is already at openspec/changes/archive/<name>/." |
| Change has incomplete tasks | Upstream handles (`verify-change`-style check). orbit doesn't change this. |
| User aborts at the prompt | No archive happens; no summary written (nothing to summarize). Change remains in `openspec/changes/<name>/` ready for next attempt. |

## Heuristics

- **Drift findings are informational, not blocking by default.** Critical-drift prompt is the only point where archive pauses for user input.
- **Don't auto-fix.** Archive doesn't try to resolve drift on its own. That's `/opsx:address-reviews` territory. archive's only options are surface-prompt-proceed.
- **Preserve the audit context.** When prompting on CRITICAL drift, show the full audit findings (not just counts). User needs to see what's flagged to decide.

## Open design questions

1. **Should archive also auto-invoke `/opsx:review --as system` if it hasn't run?** Lean: no. System-mode review is the user's gate before archive; if they skipped it, that's their decision. Archive auditing focuses on drift (the OPENSPEC_LESSONS lesson), not on re-running all the review passes.
2. **What about the explore.md still-Open-questions case?** If `openspec/changes/<name>/explore.md` has Open questions when archive runs, that suggests something was left unresolved during propose. Lean: warn but don't block. The change made it to apply and system-mode review; if Open questions in explore.md weren't material, that's user judgment. The warning surfaces it; user decides.
3. **Auto-promote `@review:` markers to `@todo:`?** If markers still exist in change dir when archive runs (haven't been resolved), should they convert to permanent `@todo:` so they don't disappear into the archive? Lean: warn with explicit "N unaddressed `@review:` markers will land in archive — convert to `@todo:` or address before archiving?" prompt.

## Composition with related commands

```
/opsx:apply <change>
        │
        ▼
code generated; tasks marked
        │
        ▼
/opsx:review <change> --as system           (internal review)
        │
        ▼
/opsx:review-external <change> --as system  (optional external review)
        │
        ▼
/opsx:address-reviews                  (resolve findings)
        │
        ▼ (cycle until ready)
        │
/opsx:archive <change>                 ◄── this command
        │
        ├── auto-invokes /opsx:audit-drift
        │     │
        │     └── findings surface to user
        │             │
        │             ├── proceed (no critical, or user accepts)
        │             │
        │             ├── abort (user wants to fix first)
        │             │
        │             └── address (user fixes; re-invoke archive)
        │
        ├── runs sync-specs (merge deltas to baseline)
        │
        ├── moves openspec/changes/<name>/
        │     → openspec/changes/archive/<name>/
        │
        ├── writes archive run summary to
        │   openspec/changes/archive/<name>/.orbit-runs/archive-<TS>.json
        │
        ▼
   change is archived
```

## What this means for the SKILL.md modifications

Upstream's `openspec-archive-change` SKILL.md handles the move + sync-specs. Orbit modifies it with:

- A pre-archive audit step (calls `audit-drift` as a library function)
- The critical-drift prompt + user-decision branching
- `--skip-audit` flag handling
- Archive run summary writing
- `@review:` marker check + warning

Standard archive behavior is preserved verbatim. Orbit's additions wrap around the upstream core; they don't replace it.

Same shape for the slash-command body (`.claude/commands/opsx/archive.md`) — append orbit-specific pre/post hooks around the upstream content.
