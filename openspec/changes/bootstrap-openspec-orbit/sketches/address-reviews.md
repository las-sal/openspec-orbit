# Sketch: `/opsx:address-reviews`

> **Status**: design sketch — **lean v1**. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence), principle 2 (cost up front).

## Purpose

The **resolution counterpart** to the generative review commands. Walks `@review:` markers anywhere in the repo and resolves each with pushback discipline.

Where `/opsx:review-proposal` and `/opsx:review-system` *generate* findings, `/opsx:address-reviews` *acts on* `@review:` markers the user (or another tool) has left.

## What it delivers over "just ask the AI"

Four enforcement wins. Each maps to a documented failure mode in the home-control transcripts:

| Discipline | Without the skill |
|---|---|
| **Convention durability** — `@review:` is the same marker every session | AI uses different syntax (`TODO:`, `FIXME:`, `XXX:`); markers don't get found later |
| **Pushback discipline** — verify against current state before fixing | AI re-fixes already-fixed state, churns the diff (OPENSPEC_LESSONS.md lesson #2) |
| **Marker removal invariant** — resolved markers get deleted | Markers leak into canonical specs (your codex feedback flagged this) |
| **Multi-file-type uniformity** — same marker in markdown, code, configs | Per-file-type marker conventions diverge, lose searchability |

Lean v1 is scoped to these four enforcement wins **plus `--from-file`** (needed to close the cross-AI loop without per-finding copy-paste). `--from-paste` (stdin), automatic ripple cascade, severity-tracked output, and other comprehensive features remain deferred to v2 (issue tracked).

## Marker convention

`@review: <content>` — single convention across all file types. Inside the file type's own comment syntax when applicable:

```
# spec.md (markdown — bare, sits in prose)
Each device exposes a state attribute. @review: should we
explicitly forbid sensitive devices from this?

# foo.ts (source — inside file-type comment)
function authorize(token: string) {
  // @review: what if token has expired but is still well-formed?
  return decode(token);
}

# config.yaml (config — inside file-type comment)
auth:
  token_ttl: 3600  # @review: should this be configurable per env?
```

Grep is just `@review:` regardless of file type. address-reviews handles them uniformly.

## Inputs

- `<scope>` (optional) — path or pattern restricting scan. Default: whole repo with safe exclusions (`.git`, `node_modules`, `dist`, `build`, etc.).
- Flags:
  - `--from-file <path>` — ingest findings from an external-review output file (typically `openspec/changes/<name>/.orbit-runs/external-<as>-<TS>.md` produced by `/opsx:review-external`). Treats each finding in the file as a virtual marker — same lifecycle as inline markers (pushback → classify → apply → log), except there's no actual marker text in source files to remove.
  - `--list` — preview only; enumerate markers (or virtual markers from a file) without acting.
  - `--only <pattern>` — narrow the scan (e.g., `--only openspec/changes/` to scope to in-flight changes, or `--only src/` to scope to source). Ignored when `--from-file` is used.
  - `--keep-resolved-markers` — debug flag; don't remove markers after resolution. No effect on `--from-file` virtual markers.

No depth modes (no `--fast`/`--full`/`--thorough`) — discrete-item processing, no scan depth dimension. No `--parallel` — markers may have inter-dependencies; sequential is safer.

## What it reads

- **Markers source**: any file in scope (default: whole repo with exclusions).
- **Resolution context**: `explore.md` (decisions), `CLAUDE.md`, `openspec/project.md`, `openspec/specs/<capability>/spec.md` (for spec resolution), `openspec/lenses/`, `*_convention.md` — read as needed when resolving a specific marker.
- **Pushback context**: current git state (`git log`, `git diff`, file contents at HEAD).

## Lifecycle

```
┌────────────────────────────────────────────────────┐
│  1. DISCOVER                                        │
│     grep -rn "@review:" scope (respecting          │
│     exclusions). For each: file:line + content.    │
└─────────────────────────┬──────────────────────────┘
                          ▼
┌────────────────────────────────────────────────────┐
│  2. TRIAGE                                          │
│     Show numbered list; user can scope             │
│     ("just 1-3, skip the rest").                   │
└─────────────────────────┬──────────────────────────┘
                          ▼
┌────────────────────────────────────────────────────┐
│  3. WALK EACH (sequential)                          │
│     For each marker:                                │
│       a. VERIFY against current state (pushback)    │
│       b. CLASSIFY: trivial / decision / unresolvable│
│       c. APPLY (or defer per user choice)           │
│       d. REMOVE the marker                          │
└─────────────────────────┬──────────────────────────┘
                          ▼
┌────────────────────────────────────────────────────┐
│  4. RIPPLE FLAG (no auto-cascade in lean v1)        │
│     For each resolved change touching normative     │
│     content, list affected related files. User      │
│     can re-run /opsx:address-reviews after fixing   │
│     them, or address inline.                        │
└─────────────────────────┬──────────────────────────┘
                          ▼
┌────────────────────────────────────────────────────┐
│  5. REPORT                                          │
│     Resolution log: ✓ resolved / ⚠ stale /          │
│     ⏸ deferred / ✗ escalated.                       │
└────────────────────────────────────────────────────┘
```

### Step 3a — Pushback (verify against current state)

Before fixing, `grep` / `git log` / read the file to check if the claim is still true:

- **Already fixed** (state changed between marker creation and now): mark as "stale, suppressed"; report current state + commit hash; **remove the marker without further edit**.
- **Still applies**: proceed to classify.
- **Partially applies** (mixed): break into pieces; handle each separately.

This is the discipline from OPENSPEC_LESSONS.md lesson #2. Easy to forget without the skill enforcing it.

### Step 3b — Classification

Each marker resolves as one of:

| shape | what happens |
|---|---|
| **Trivial fix** | Propose edit + apply + remove marker. No user input needed. |
| **Decision required** | Surface options via `AskUserQuestion`; apply user's choice; remove marker. |
| **Stale** (from pushback) | Report current state + evidence; remove marker. No edit. |
| **Unresolvable now** | Per-marker user choice with default: (a) file as a follow-up task in `tasks.md` and remove the marker (default), (b) convert to permanent `@todo: ...` marker, (c) leave with `@review(escalated): ...` and explanation. |

### Step 4 — Ripple flag (no auto-cascade in lean v1)

When a resolution touches normative content, the skill **lists** affected related files (sibling specs, CLAUDE.md, project.md, conventions, lenses) but **doesn't automatically edit them**.

Why this is lean: auto-cascade is convenient but introduces edit bursts the user may not want. Flagging is honest about the dependency without making implicit decisions. User decides what to fix and when.

v2 (issue tracked) can add `--cascade` for batch ripple resolution once the lean version proves the workflow.

## Output format

A simple resolution log (not the 3-dimension scorecard used by review/audit commands — this is action, not scan):

```markdown
## Resolution Log: 12 markers processed

### Summary
| Outcome    | Count |
|------------|-------|
| ✓ Resolved |     8 |
| ⚠ Stale    |     2 |
| ⏸ Deferred |     1 |
| ✗ Escalated|     1 |

### Resolved (8)
1. openspec/changes/foo/specs/host-lifecycle/spec.md:42
   Added explicit "obstruction clears" scenario per user decision.
   Related files to consider: design.md:78, CLAUDE.md:104.
2. openspec/changes/foo/design.md:147
   Clarified thermostat temperatureUnit semantics (display-only).
…

### Stale, suppressed (2)
1. spec.md:284 — claim "scene CRUD allows action attributes" — verified
   at HEAD (commit a1b2c3d): rule already explicit. Marker removed.
…

### Deferred to tasks.md (1)
1. design.md:529 — "fixture format extensions need YAML examples" —
   filed as task "Document fixture-format YAML for v0.5 device types".

### Escalated (1)
1. specs/mcp-server/spec.md:97 — JSON coercion schema lookup — needs
   design decision. Converted to @review(escalated): ... with note.
   Recommend: revisit before /opsx:apply.

### Final Assessment
0 unaddressed markers in scope (1 escalated marker is deliberately
persisted).
Suggested next: re-run /opsx:review-proposal to confirm clean baseline.
```

## Heuristics & graceful degradation

- **Always remove markers on resolution** — primary invariant.
- **Pushback before fix** — every marker passes through Step 3a.
- **Surface options, don't pre-commit** — ask the user when a marker has design implications.
- **Never create new markers without explicit user consent** — only `/opsx:review-proposal --mark` writes new markers, and only with user opt-in.
- **Graceful degradation**:
  - No markers found → report "no `@review:` markers in scope" and exit clean.
  - Pushback can't verify (no git history, no current file) → escalate to user rather than guess.

## `--from-file` parsing (v1 inclusion)

When `--from-file <path>` is supplied, address-reviews parses the file as markdown with severity sections. Expected structure (matches what `/opsx:review-external` instructs the external AI to produce):

```markdown
# External Review: <change> (iteration N)

**Reviewer**: <model>
**Date**: <date>

## CRITICAL

### <Title>
**File**: <path>:<line>
**Description**: <text>

## WARNING

### ...

## SUGGESTION

### ...
```

Parser extracts per finding: severity (from section), title, file:line, description. Each becomes a virtual `@review:` marker — severity tag carried through. No marker exists in the actual source file, so the "remove on resolution" step is a no-op for virtual markers. The source file (`external-<as>-<TS>.md`) persists in `.orbit-runs/` as historical record.

If parse fails (malformed file), report the parse error and exit; don't act on partial input.

## What's NOT in lean v1 (tracked in v2 issue)

- `--from-paste` (stdin / chat paste — different ergonomics from file)
- Automatic ripple cascade (`--cascade` to batch-fix related files)
- Severity-aware filtering in the resolution log (e.g., `--severity CRITICAL` to work criticals first; virtual markers carry severity but the report doesn't yet prioritize on it)
- `--strict` (fail-fast on user-input-required markers)
- `--parallel` (independent-marker concurrent resolution)
- Categorized markers (`@review(blocker):`, etc.)
- Auto re-run of `/opsx:review-proposal` after batch resolution

Each of these has real value when the workflow demands it. Lean v1 ships the four enforcement wins; v2 adds the polish.

## Composition with related commands

```
   user reads / external review
   ───────────────────────────
   manual @review: markers
   in spec.md / design.md / source code
                              │
                              │
   /opsx:review-proposal      │
   ─────────────────────       │
   --mark flag drops          │
   @review: markers           │
   based on findings          │
                              │
                              ▼
                  /opsx:address-reviews [<scope>]
                              │
                              ▼
                       resolution log
                              │
                              ▼
                  (cycle: re-run review-proposal /
                   review-system to confirm clean)
                              │
                              ▼
                       /opsx:apply or /opsx:archive
```

External review findings (codex, fresh-claude) in lean v1 are ingested directly via `--from-file <path>`: the external AI writes its findings markdown to a file (per the `/opsx:review-external` handoff format), and `/opsx:address-reviews --from-file` parses each finding as a virtual marker and walks it through the standard lifecycle (pushback → classify → apply → log). No per-finding copy-paste. v2's `--from-paste` adds an stdin-style entry point for ad-hoc findings.

## Parallels with review / audit commands

| | review-proposal / review-system / audit-drift | address-reviews (lean v1) |
|---|---|---|
| Mode | scan + report | act + log |
| Input | a scope (change name) | `@review:` markers in scope |
| Output | 3-dimension scorecard, severity | resolution log, per-item outcome |
| Flags | depth + execution + secondary | minimal (`--list`, `--only`, `--keep-resolved-markers`) |
| State after | findings reported; user reads | items resolved; markers removed |
| Idempotent? | yes (scanning doesn't change state) | no (deliberately mutates artifacts) |

The pairing **scan → act** is what closes the review loop.
