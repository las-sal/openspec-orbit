---
name: openspec-address-reviews
description: "Resolve `@review:` markers anywhere in the repo (or review findings via `--from-file` markdown / JSON, or auto-discovered internal review/audit-drift JSON when invoked with a change name and no markers) by walking each through pushback → classify → fix → ripple-cascade → remove-marker. Walk-mode (per-finding) is the default; `--batch` opts in. Cascade-by-default auto-applies ripple edits to IN-set files (everything outside the four lifecycle-invariant OUT categories: audit trail, baseline specs, cross-change/archive dirs, safe-exclusions); `--no-cascade` opts out. Use after `/opsx:review` writes findings to `.orbit-runs/` (auto-discovered; no `--mark` pre-step needed), after `/opsx:review-external` returns external findings, after `/opsx:audit-drift` writes drift findings, or any time the repo has accumulated `@review:` annotations."
license: MIT
compatibility: Requires openspec CLI. Ingests findings via `--from-file` (external-review markdown OR internal `review-<mode>-*.json` OR `audit-drift-*.json`) OR via auto-discovery from `.orbit-runs/` when invoked with a change-name positional and no inline markers found. Default behaviors: walk-mode (per-finding); cascade-on (auto-applies ripples to IN-set files). `--batch` opts INTO batch-mode; `--no-cascade` opts OUT of cascade. `--mark` is optional, not prerequisite.
metadata:
  author: openspec-orbit
  version: "0.1"
  capability: orbit-address-reviews
---
Resolve `@review:` markers across the repo (or external-review findings from a file) with pushback discipline. The lean v1 lifecycle: **discover → triage → walk → ripple flag → report**. Markers are removed from their source files on resolution (the marker-removal invariant) so they don't leak into canonical artifacts.

**Generates resolutions; does not generate findings.** This is the counterpart to `/opsx:review`. Primary use case: close the cross-AI review cycle by ingesting external-AI findings via `--from-file`. Secondary use case: walk repo-scanned inline `@review:` markers with structured pushback.

**Input**: Optional scope (path or pattern) and optional flags. Default: whole-repo scan with safe exclusions.

## Three execution disciplines (apply throughout this command)

**Pushback (review-time). Primary discipline for this command.** Before fixing any marker, verify the claim against current state. Procedure:

1. Identify the marker's referenced symbol, name, or concept.
2. `grep -rn` for current presence in expected locations (the file the marker is in, related files, baseline specs).
3. If the symbol is absent where the marker expects it, run `git log -S "<symbol>" --since=<window>` (default: since the marker file's last modification) to confirm intentional removal.
4. Read the relevant file's current content.
5. Compare to the marker's claim and decide: **still applies** / **already fixed** / **partially applies**.
6. On already-fixed, report the commit hash or current-content evidence as part of suppression.

Do NOT re-edit already-fixed state. Stale markers get removed without further edit, with an evidence note in the resolution log.

**Read-before-reference (authoring-time)**. When a marker resolution edits a file (trivial fix or applied decision), reference the actual current content of the file — read first. Don't assume the structure based on what the marker text describes; the file may have evolved since the marker was placed. Re-read after any edit cycle to confirm the change landed correctly.

**Change completeness (modification-time)**. When a marker's resolution touches related artifacts (e.g., resolving a spec marker has downstream design.md / proposal.md implications), surface those as **ripple-flagged files** (Step 5 below). v1 does NOT auto-cascade — the user gets a list of files to check, not silent edits. After mechanical replacement (e.g., a find-replace as part of a trivial fix), sweep for residue: doubled words, broken references, content overwrites. Known residue must not be left for a downstream review to catch.

## Steps

### 1. Discover (or ingest)

**Discovery priority order**:

1. `@review:` markers in scope (always tried first when markers might exist — preserves current behavior)
2. If positional resolves to a change name AND no markers found AND no `--from-file`: auto-discover most-recent internal review JSON or audit-drift JSON in the change's `.orbit-runs/`
3. If neither markers nor JSON: clean "no findings" exit

**Explicit `--from-file <path>` is always exclusive**: when specified, marker grep does NOT run and auto-discovery does NOT run — the parser ingests the file as the ONLY input source. The positional (if any) is used only for emit-path resolution (where the resulting `address-reviews-<TS>.json` is written).

**Default — whole-repo scan** (no positional, no `--from-file`):

```bash
grep -rn --include='*' "@review:" . \
  | grep -v '/.git/' \
  | grep -v '/node_modules/' \
  | grep -v '/dist/' \
  | grep -v '/build/'
```

Safe exclusions: `.git/`, `node_modules/`, `dist/`, `build/`. Other common build dirs may be added per project (`.next/`, `target/`, etc.). Auto-discovery does NOT apply for bare invocation — no change-directory anchor for `.orbit-runs/` lookup.

**Positional `<scope>` argument** — resolve in this order:

1. **Active change**: `openspec/changes/<scope>/` exists → treat `<scope>` as a change name; grep restricted to that directory; auto-discovery available (Step (c) below).
2. **Archived change**: no active match, but `openspec/changes/archive/` contains one or more entries matching the regex `^\d{4}-\d{2}-\d{2}-<scope>$` → treat as a change name resolving to the archive directory. If multiple archive entries match the same `<scope>` suffix (rare; same change name archived on different dates), the lexicographically-latest date wins (most-recent archive).
3. **Path/pattern fallback**: no active or archive match → treat as a path/pattern scope; grep restricted to that path; auto-discovery does NOT apply (no `.orbit-runs/` anchor).

**Auto-discovery fallback** (change-name positional, no markers, no `--from-file`):

(a) Run grep for `@review:` markers in the resolved change directory. If any found → walk them (current behavior; auto-discovery does NOT fire because markers already provide work).

(b) If grep returns zero markers AND no `--from-file` flag was specified: look in the change's `.orbit-runs/` directory for files matching `review-<mode>-*.json` OR `audit-drift-*.json`. Pick the single most-recent file by filename `<TS>` token across all candidate types — review and audit-drift JSON compete on the same recency axis with NO class preference (per `D-recency-1`; audit-drift CAN win over review if its `<TS>` token is later). If two candidate files share an identical `<TS>` token (rare; possible only within the same ISO-second), tie-break by stable lexicographic sort of the full filename (ASCII order: `a` < `r`; within `review-`, `proposal` < `system` → so `audit-drift-<TS>.json` sorts before `review-proposal-<TS>.json` sorts before `review-system-<TS>.json`); alphabetically-earliest wins.

(c) If a candidate JSON is found: feed it to the `--from-file` ingest path described below (content-sniff routes to JSON parser per `references/internal-findings-format.md`; identical lifecycle and virtual-marker construction to explicit `--from-file` invocation; the only difference is the resolution log's `source` field — see Step 5).

(d) If neither markers nor JSON candidates exist: emit `No @review: markers in scope and no internal review/audit-drift JSON in .orbit-runs/. Nothing to walk.` and exit cleanly.

**`--mark` is optional, not prerequisite**: historically, `/opsx:review --as proposal --mark` was the way to produce markers that address-reviews would later walk. Auto-discovery makes `--mark` purely a stylistic choice (do you want findings annotated in source for diff-readability?), NOT a structural requirement. The canonical `/opsx:review <name>` → `/opsx:address-reviews <name>` workflow works without `--mark` because address-reviews discovers the just-written `review-<mode>-*.json`.

**`--from-file <path>` ingest** (review findings file — external markdown OR internal JSON):

The `--from-file` flag accepts TWO format families: (a) **external-review markdown** (produced by external AIs per `/opsx:review-external`'s prompt template), (b) **internal findings JSON** (produced by `/opsx:review` to `.orbit-runs/review-<mode>-*.json` OR by `/opsx:audit-drift` to its summary JSON paths). The parser auto-detects format via content sniff and routes to the appropriate parser:

- **Leading `{`** (first non-whitespace character) → JSON parser, per the contract at `references/internal-findings-format.md`. V1 accepts `command: "review"` OR `command: "audit-drift"` JSON; other internal JSON commands (`address-reviews`, `apply`, `archive`, `propose`, etc.) are rejected with a clean unsupported-command error. `command: "address-reviews"` is rejected on purpose to prevent recursive ingest cycles.
- **Leading `# External Review:`** → markdown parser, per the contract at `references/external-findings-format.md`. This is the only markdown format orbit produces — written by external AIs after pulling the repo + reading the `/opsx:review-external` prompt.
- **Neither leading pattern matches** → emit a clean format-mismatch error naming both supported formats; refuse to act on partial parse.

The sniff inspects content only — it does NOT depend on file extension or pathname. Each finding (from either format) becomes a virtual marker with the same shape: `severity` / `title` / `file:line` / `description` / `source` (tagged `external` for markdown, `internal-review` for `command: "review"` JSON, `audit-drift` for `command: "audit-drift"` JSON). Virtual markers walk the same lifecycle as inline markers, with one exception: **the marker-removal step (Step 3e below) is a no-op for virtual markers regardless of provenance** — there's no source-file marker text to delete (the JSON file or markdown file is an audit artifact that travels with the change into archive; removing it on resolution would be wrong).

**Fresh pushback applies to JSON virtual markers** (both review and audit-drift): the source JSON's own `stale_suppressed[]` array already filtered stale findings at source-run time, but state may have changed between the source run and the resolve. The Step 3a pushback verification (per the primary discipline above) runs for every virtual marker — JSON-sourced or markdown-sourced — to catch staleness introduced since the source pass.

### 2. Triage

Present discovered markers (or parsed virtual markers) as a numbered list:

```
Found 12 markers in scope:

  1. openspec/changes/foo/proposal.md:41 — "@review: should this be five or four?"
  2. openspec/changes/foo/design.md:159  — "@review: 9 vs 8 after merge"
  3. src/api/auth.ts:88                   — "// @review: token TTL configurable per env?"
  ...
 12. (external) proposal.md:33            — [CRITICAL] orbit-conventions bullet omits three disciplines
```

Mark external-sourced entries explicitly (`(external)` prefix). Use **AskUserQuestion** to let the user scope:

- Walk all (default)
- Walk a subset (e.g., "1-3, 7, 9-12")
- Walk only critical (when `--from-file` source provides severities)
- Cancel

### 3. Walk each marker

**Walk-mode is the default** (per #11): each marker receives its own complete inner cycle — pushback → classify → fix → ripple-cascade → remove-marker — before the next marker's pushback begins. The user can interrupt between markers; the resolution log captures partial completion if interrupted.

**Batch-mode is opt-in** via one of three triggers (mutually-exclusive — first match wins; resolution log records which fired in `walk_mode_source`):

| Trigger | Detection | `walk_mode_source` |
|---|---|---|
| **`--batch` flag** | Present in the invocation argv. Canonical signal. | `"flag"` |
| **Verbal `--batch` in invocation message** | The user's invocation message (the message that triggered the command) contains one of these batch-intent phrases: `"fix them all"`, `"batch them"`, `"go ahead with all"`, `"address all at once"`, or another clearly equivalent phrase conveying batch-resolution intent. The phrase MUST be in the invocation message — phrases inside subsequent walk-step responses do NOT shift mode (see next row). | `"verbal"` |
| **Mid-walk command-shape interruption** | During an active walk, the user sends a bare, unambiguous mode-switch message: `"go batch"`, `"switch to batch"`, `"batch the rest"`. The message must be command-shaped (typically 2-4 words, no conversational filler) — ambiguous phrases like `"yeah let's just keep going"` or `"fine, fix them all"` mid-walk do NOT shift mode. Record the 1-indexed finding number at which the shift occurred in the resolution log's `walk_mode_shifted_at_finding` field. | `"command-shape-interruption"` |

In batch-mode, all markers complete `pushback + classify + fix` in one pass before any ripple-cascade fires; ripples are applied once at the end as a single aggregated step. **Decision-fork prompts still fire** (per Step 3c.5) even in batch-mode — batch only suppresses per-finding completion checkpoints for unambiguous fixes.

For every marker in the chosen scope, execute the lifecycle below sequentially.

#### 3a. Apply pushback (primary discipline above)

Run the verification procedure. Determine: still applies / already fixed / partially applies.

#### 3b. Classify

Apply heuristics in order:

1. **stale** — pushback determined the issue is already resolved at HEAD. Skip directly to 3d (remove + log as ⚠ Stale).
2. **trivial fix** — single-line edit or a few-line localized edit with one obvious correct answer (no design implication, no scope question, no ambiguity in intent). Proceed to 3c without `AskUserQuestion`.
3. **decision required** — resolution requires ambiguity resolution, a design choice between defensible alternatives, a scope decision, or has implications beyond the immediate location. Surface 2–4 concrete options via **AskUserQuestion** (not open-ended).
4. **unresolvable** — resolution needs information not currently available (deferred decision, future capability, blocked on external input). Default: file as a task in `tasks.md` (proceed to 3d with action "filed as task"). Alternatives via **AskUserQuestion**: convert to `@todo: <content>`, escalate to `@review(escalated): <content with explanation>`.

#### 3b.5. Decision-fork detection (gated on classify == "decision required")

**Fires only when classify == "decision required"** (per #18). Skip this sub-step entirely for `stale` / `trivial fix` / `unresolvable` classifications — they short-circuit straight to 3c.

**Goal**: when a finding's recommendation is disjunctive (multiple defensible paths the user must choose between), surface as an `AskUserQuestion` fork rather than the generic 2–4 option prompt — and capture the chosen option in the resolution log for downstream audit.

**Hybrid detection** (try structured first; fall back to heuristic):

1. **Structured path** — if the finding came from a JSON-virtual marker (internal-review or audit-drift JSON via `--from-file` or auto-discovery) AND the source JSON's finding entry includes a `recommendation_options: [{label, body}]` array with ≥ 2 entries AND every entry has non-empty `label` AND non-empty `body`: use that array DIRECTLY as the fork options. Skip heuristic. Record `recommendation_fork.source: "structured"` in the resolution log.

2. **Heuristic fallback** — if structured path was unavailable (missing field, malformed entry — see malformed-handling below) OR the marker is markdown-sourced (external markdown, inline markers): scan the finding's recommendation text for STRICT disjunctive signals. Triggers (all are non-fuzzy):
   - **Numbered alternatives**: `(A) … (B)`, `1. … 2.`, `[A] … [B]`
   - **"Either … or …"** with clause-level branches (each clause is a recognizable recommendation)
   - **"Options:" prefix** (accepting both bold variants: `**Options**:` colon-outside-bold, `**Options:**` colon-inside-bold; normalize markdown emphasis before pattern-matching) followed by a list or numbered enumeration

   **NOT triggers** — do NOT fire on loose "or" in prose: `"fix it now or later"`, `"X or Y could happen"`, `"the issue is A or B and we should address one of them"`. False-negative bias intentional.

   On heuristic match, parse the recommendation text into options (each option becomes `{label, body}`; labels typically `"A"`, `"B"`, …). Record `recommendation_fork.source: "heuristic"` in the resolution log.

3. **No detection** — neither structured nor heuristic triggered: classify falls through to the generic decision-required path (Step 3b's normal `AskUserQuestion` with 2–4 concrete options the AI surfaces from context). No `recommendation_fork` field is recorded.

**Malformed structured input** (zero entries, single entry, missing `label` or empty `label`, missing `body` or empty `body`): emit a stderr warning naming the finding + reason (`"only 1 entry in array"`, `"missing label on entry index 2"`, `"empty body on entry index 0"`); FALL BACK to heuristic detection over the prose `recommendation`. Record `recommendation_fork.source: "heuristic"` AND `recommendation_fork.structured_path_skipped_reason: "<reason>"` in the resolution log. Malformed input never aborts the walk.

**Fork prompt UX**: surface options via `AskUserQuestion` with the finding title as the question and each option as a button. ALWAYS include a `[discuss]` option as the per-prompt escape hatch — selecting `[discuss]` makes the AI surface tradeoff analysis on the options (pros/cons; relevant context; any related findings) and re-prompt with the same fork (or a refined fork if discussion clarifies a third path). Record `recommendation_fork.discuss_invoked: true` when `[discuss]` was used; `false` otherwise.

**Capture in resolution log** — `recommendation_fork` object on the per-resolution entry (omitted entirely when no fork fired):

```json
"recommendation_fork": {
  "detected": true,
  "source": "structured" | "heuristic",
  "options_presented": [{ "label": "A", "body": "..." }, { "label": "B", "body": "..." }],
  "chosen": "A",
  "discuss_invoked": false,
  "structured_path_skipped_reason": "<optional; only when structured was attempted then skipped>"
}
```

**Fork-prompt firing in batch-mode**: forks STILL fire in batch-mode (per Step 3 intro). Batch only suppresses per-finding completion checkpoints for unambiguous fixes; user decisions on disjunctive recommendations remain user-driven.

**Mini fork-prompt trace**:

```
Walking finding W1 (WARNING, classified "decision required"):
  Title: Test suite covers ~30 of ~45 spec scenarios
  Recommendation: "Either (A) file a follow-up issue tracking the v2 polish, or
                  (B) add Group 19 to tasks.md extending this change's scope."

Decision-fork detection fired via heuristic ("Either ... or" with clause-level branches).
Options parsed:
  (A) file a follow-up issue tracking the v2 polish
  (B) add Group 19 to tasks.md extending this change's scope

→ AskUserQuestion: "W1 — which path?"
   [A] file a follow-up issue
   [B] extend scope to tasks.md
   [discuss] surface tradeoffs

User picks [A]. Resolution-log entry records:
  recommendation_fork: {
    detected: true, source: "heuristic",
    options_presented: [{label: "A", body: "..."}, {label: "B", body: "..."}],
    chosen: "A", discuss_invoked: false
  }
```

#### 3c. Apply the fix

- **trivial fix**: apply the edit. Re-read the file after to confirm.
- **decision required**: apply the user's chosen option (from Step 3b.5 fork prompt if fired, or from the generic prompt otherwise).
- **unresolvable (default)**: append a task to `openspec/changes/<change>/tasks.md` (or repo-level `TODO.md` if no change context); task text is the marker's content + ripple-flag context.
- **unresolvable (`@todo:` or `@review(escalated):`)**: replace the marker text in place (do NOT remove — the new form persists as future signal).

#### 3d. Ripple cascade (auto-apply by default; `--no-cascade` opts out)

If the resolution edited a normative artifact, compute the ripple-flag set — files that need parallel edits to stay consistent with the primary fix. Then split each ripple into **IN** (auto-applied) and **OUT** (recorded in `flagged_not_applied[]`).

**OUT — the four lifecycle-invariant exclusion categories** (these are the ONLY structural exclusions; any other file is IN):

| OUT category | Path prefix | Reason recorded in `flagged_not_applied` |
|---|---|---|
| Audit trail | `openspec/changes/<name>/.orbit-runs/*` AND `openspec/.orbit-runs/*` | `audit-trail file; cascade skipped by policy` |
| Baseline specs | `openspec/specs/<capability>/spec.md` | `baseline spec; add a delta to your current change's specs/<capability>/spec.md to capture this ripple` |
| Cross-change (active + archived) | `openspec/changes/<other-name>/*` AND `openspec/changes/archive/*` | `cross-change ripple; cascade scope is current change only` |
| Safe-exclusions (exact set) | `.git/`, `node_modules/`, `dist/`, `build/` | `safe-exclusion path; never edited` |

**The safe-exclusion prefix set is EXACT** — do NOT extend ad-hoc to other paths during application. Expanding the set requires a spec change.

**IN — everything else.** File extension is NOT a discriminator: `.py`, `.swift`, `.c`, `.sh`, `.md`, dotfiles, configs — all eligible if ripple-flagged and not matching an OUT prefix. For each IN file, apply the parallel edit consistent with the primary fix (run pushback against the file's current state first; classify as decision-required if ambiguous and surface a fork prompt). Record the path in `ripple_cascade.applied[]`.

**`--no-cascade` flag** (opt-out): when set, NO ripples are edited regardless of IN/OUT classification. ALL ripple-flagged files are recorded under `ripple_cascade.flagged_not_applied[]` with reason `--no-cascade suppressed`. Use case: user wants to inspect the ripple set before allowing edits.

**Cascade trusts the ripple-flag analysis**: cascade does NOT make independent "should this file be touched?" judgments beyond the four-category OUT check. The contextual scope judgment ("is this file relevant to my finding?") lives in the earlier ripple-flag-derivation step — if ripple-flag analysis didn't surface a file, cascade doesn't touch it.

**Reason-code source distinction** (per spec's mode invariants):

- **Structural OUT-category reasons** (the four codes in the table above): fire when cascade was ON but the file matched an OUT prefix.
- **Mode-suppression reason** (`--no-cascade suppressed`): fires uniformly for ALL ripple-flagged files when `--no-cascade` is set, regardless of IN/OUT classification.

The two source categories are structurally distinct — `--no-cascade suppressed` is NOT a fifth OUT category.

#### 3e. Remove the marker (invariant)

Unless `--keep-resolved-markers` is set, delete the original `@review: <text>` from its source file:

- **Markdown**: remove the marker text. If it was the only content on a line, remove the line.
- **Source code (C-style)**: remove just the marker text. If the comment now contains only whitespace or the comment delimiters (`//`, `/* */`), remove the whole comment.
- **Source code (hash)**: same as C-style for `# @review:` comments.
- **Virtual marker (external markdown, internal-review JSON, or audit-drift JSON)**: no-op — there's no source text to remove (the source file is an audit artifact). Log as resolved with the appropriate `source` tag (`external`, `internal-review`, or `audit-drift`).

For `unresolvable` conversions (`@todo:` / `@review(escalated):`), the marker is **transformed** in place rather than removed; this is still considered "resolution" for log purposes.

`--keep-resolved-markers` flag: skip the removal step entirely. Debug use only — markers persist after resolution.

### 4. (Internal — Step 3 runs inside this loop)

Iteration. After all markers in scope have walked Step 3, proceed to reporting.

### 5. Report — emit the resolution log

Resolution log is NOT a 3-dimension scorecard. Output structure:

```
## Address-reviews report

Source: <whole-repo | scope <path> | --from-file <path> | auto-discovered <path>>
Markers found: <N>
Markers walked: <M> (subset specified: <yes/no>)

### Summary

| Status        | Count |
|---------------|-------|
| ✓ Resolved    | 5     |
| ⚠ Stale       | 2     |
| ⏸ Deferred    | 1     |
| ✗ Escalated   | 1     |

### ✓ Resolved
- **openspec/changes/foo/proposal.md:41** — `@review: five or four?`
  Action: applied trivial fix (`five` → `four` + added explicit list).
  Ripple: design.md, README.md flagged for sibling consistency.

- **src/api/auth.ts:88** — `// @review: token TTL configurable per env?`
  Action: applied user-selected option B (env-var override with default).
  Ripple: src/api/auth.test.ts flagged.

### ⚠ Stale
- **openspec/changes/foo/design.md:159** — `@review: 9 vs 8 after merge`
  Evidence: already corrected in commit abc1234. Marker removed without edit.

### ⏸ Deferred (filed as tasks / converted)
- **openspec/changes/foo/proposal.md:88** — `@review: caching scope`
  Action: filed as task `tasks.md` 10.5; marker removed.

### ✗ Escalated
- **openspec/changes/foo/design.md:204** — `@review: blocking on legal decision re: data retention`
  Action: converted to `@review(escalated): blocking on legal decision re: data retention (escalated 2026-05-18 — see explore.md)`.

### Final assessment

0 unresolved inline markers remaining in scope.
1 escalated marker deliberately persisted.
Suggested next: re-run /opsx:review --as proposal to confirm clean baseline.
```

The final-assessment line summarizes remaining-in-scope markers (0 if clean) plus any deliberately persisted escalations, and suggests the next command.

### 6. Persist the run summary

Write JSON to:

- **Change-scoped** (when scope is a single change directory or `--from-file` points into a change's `.orbit-runs/`): `openspec/changes/<change-name>/.orbit-runs/address-reviews-<TS>.json`
- **Whole-repo / cross-change**: `openspec/.orbit-runs/address-reviews-<TS>.json`

Full schema (fields, types, semantics) lives at `references/run-summary-schema.md` — read that file when composing the summary.

## Marker syntax across file types

The command recognizes these marker forms uniformly:

| Context | Form |
|---|---|
| Markdown | `@review: <text>` (bare) |
| C-style code (TS/JS/Go/Rust/C/C++/Java) | `// @review: <text>` or `/* @review: <text> */` |
| Hash-comment files (Python/Ruby/shell/YAML/TOML) | `# @review: <text>` |

Adjacent forms (same discovery grep, different semantics):

| Marker | Meaning | Handled by this command? |
|---|---|---|
| `@review: <text>` | Needs review/decision | Yes — full lifecycle |
| `@review(escalated): <text>` | Escalated, awaiting human | Yes — listed in resolution log; NOT auto-walked unless explicitly scoped |
| `@todo: <text>` | Known follow-up work, NOT a review item | No — out of scope |

## Constraints

- **Never write new `@review:` markers.** Only `/opsx:review --as proposal --mark` does that. address-reviews can transform markers (e.g., to `@todo:` or `@review(escalated):`) but never creates fresh `@review:` markers.
- **Cascade by default; `--no-cascade` opts out.** Ripple-flagged files in the IN set (everything outside the four lifecycle-invariant OUT categories — audit trail, baseline specs, cross-change/archive dirs, safe-exclusions) are auto-edited per Step 3d. File extension is NOT a discriminator.
- **Walk-mode by default; `--batch` opts in.** Each marker gets its own complete pushback → classify → fix → cascade → remove cycle. Batch-mode triggers via `--batch` flag, verbal phrase in the invocation message, or a bare command-shape interruption mid-walk (per Step 3 trigger table).

## Worked example (`--from-file` markdown ingest, iter 5 of bootstrap-openspec-orbit)

```
## Address-reviews report

Source: --from-file openspec/changes/bootstrap-openspec-orbit/.orbit-runs/external-proposal-2026-05-18T14-35-23Z.md
External reviewer: Claude Opus 4.7 (fresh-context in-session subagent — iter 5)
Input findings: 0 CRITICAL / 4 WARNING / 5 SUGGESTION
Markers walked: 9 (all)
Walk mode: per_finding (default)
Pushback verification: all 9 verified against current state; 0 stale suppressions.
Cascade summary: 14 IN files cascaded across 9 resolutions; 3 OUT files flagged_not_applied (2 baseline specs, 1 cross-change archive ref).

### Summary

| Status        | Count |
|---------------|-------|
| ✓ Resolved    | 9     |
| ⚠ Stale       | 0     |
| ⏸ Deferred    | 0     |
| ✗ Escalated   | 0     |

### ✓ Resolved
- **proposal.md:41** — [WARNING] "five new SKILL.md + command body pairs" should be four.
  Action: trivial fix — changed "five" to "four" + added explicit list.
  Ripple: design.md, README.md flagged.

- **design.md:159** — [WARNING] "9 capability specs" should be 8 after merge.
  Action: trivial fix — changed 9 to 8 with parenthetical merge context.

- **sketches/review.md:3** — [WARNING] corrupted filenames (sed residue from rename).
  Action: trivial fix — restored proper filenames.

- (6 more, all trivial fix or applied decision)

### Final assessment

0 unresolved external findings.
Suggested next: re-run /opsx:review --as proposal to confirm convergence.

Run summary: openspec/changes/bootstrap-openspec-orbit/.orbit-runs/address-reviews-2026-05-18T14-43-41Z.json
```

## Worked example (`--from-file` JSON ingest, single SUGGESTION from a system-mode review)

```
## Address-reviews report

Source: --from-file openspec/changes/harden-review-mode-recommendations/.orbit-runs/review-system-2026-05-26T20-01-08Z.json (illustrative; the actual change has since been archived — real path is under openspec/changes/archive/2026-05-26-harden-review-mode-recommendations/)
Source format: internal review JSON (command: "review", mode: system, iteration: 2)
Input findings: 0 CRITICAL / 0 WARNING / 1 SUGGESTION
Markers walked: 1 (all)
Pushback verification: 1 verified against current state; 0 stale suppressions.

### Summary

| Status        | Count |
|---------------|-------|
| ✓ Resolved    | 1     |
| ⚠ Stale       | 0     |
| ⏸ Deferred    | 0     |
| ✗ Escalated   | 0     |

### ✓ Resolved
- **.claude/skills/openspec-review/references/run-summary-schema.md:22** — [SUGGESTION] Schema reference does not document new convergence fields emitted by iteration-aware logic.
  Source: internal-review
  Action: classified as `decision required` (out-of-scope deferral vs include-in-current-change). User chose to defer to a future change; filed as task in `tasks.md` referencing the schema-doc gap; original JSON unchanged (marker-removal no-op for virtual markers).
  Ripple: openspec-review/SKILL.md flagged (consumer of the schema doc).

### Final assessment

0 unresolved findings.
Suggested next: continue with current change's archive flow; the schema-doc gap is filed as a follow-up task and will be addressed in a future change.

Run summary: openspec/changes/harden-review-mode-recommendations/.orbit-runs/address-reviews-2026-05-26T20-XX-XXZ.json
```

This example demonstrates the JSON path: the input is an internal `review-system-*.json` produced by `/opsx:review --as system`; auto-detect routes to the JSON parser (leading `{`); the single `findings[]` entry becomes a virtual marker; the lifecycle walks it (pushback verifies the schema gap still exists → classify as `decision required` → user defers → file as task → marker-removal no-op since virtual). Identical lifecycle to the markdown example above, different parser routing.

## Graceful degradation

- **No markers found in bare or path/pattern scope** → emit `No @review: markers in scope. Nothing to do.` and exit clean.
- **No markers found in change-name scope, no candidate JSON in `.orbit-runs/`** → emit `No @review: markers in scope and no internal review/audit-drift JSON in .orbit-runs/. Nothing to walk.` and exit clean. Note: this case ONLY applies after Step 1's auto-discovery fallback has been checked; bare/path scope never reaches auto-discovery (no `.orbit-runs/` anchor).
- **No markers found in change-name scope, candidate JSON in `.orbit-runs/` (auto-discovery fires)** → NOT a graceful-degradation case; proceed with the discovered JSON as if it had been passed via `--from-file` per Step 1's auto-discovery fallback.
- **`--from-file` path missing** → fail clearly with usage; don't fall back to repo scan.
- **`--from-file` neither-format-detected** → emit a clean format-mismatch error naming both supported formats (referencing `references/external-findings-format.md` and `references/internal-findings-format.md`) plus the observed leading-content snippet; refuse to act on partial parse. Concrete error shape:

  ```
  `--from-file <path>`: unrecognized format.

  Supported formats:
  - External-review markdown — see references/external-findings-format.md
  - Internal review JSON (review-<mode>-*.json) — see references/internal-findings-format.md

  Detected: <observed first-line snippet>
  ```
- **`--from-file` JSON parse failure** (leading `{` but invalid JSON) → emit a clean parse-error message naming the file and the JSON parse-error position (best-effort); exit without acting. User fixes the file and re-runs.
- **`--from-file` unsupported JSON `command` field** (valid JSON but `command` is neither `"review"` nor `"audit-drift"`) → emit a clean error naming the supported values (`"review"`, `"audit-drift"`) and the unsupported value detected; reference both format docs for self-diagnosis; exit without acting. V1 supports `review-<mode>-*.json` and `audit-drift-*.json` only. `command: "address-reviews"` is rejected on purpose (cycle prevention).
- **`--from-file` markdown malformed** (leading `# External Review:` but missing required sections / broken field labels) → emit a markdown-parse-error message with format guidance from `references/external-findings-format.md`; exit without acting.
- **`--from-file` JSON missing `findings[]`** → treat as malformed input; emit a clean error naming the missing/malformed field; reference `internal-findings-format.md`; exit.
- **`--from-file` JSON with empty `findings: []`** → NOT an error. Succeed with zero virtual markers; resolution log reports a clean empty walk with note `Source JSON had no findings to walk; resolution log is informational.` A clean review with `findings: []` is the expected state for an all-clear pass.
- **No `tasks.md` in change context** → unresolvable-default-file-as-task falls back to creating a root-level `TODO.md` entry with a note.
- **Marker found in a baseline spec** (`openspec/specs/<capability>/`) → warn that this is unusual; baseline specs should not carry markers; still walk it per user choice but flag in the log.
