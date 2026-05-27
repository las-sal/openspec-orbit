## Context

`/opsx:address-reviews --from-file <path>` was designed in v1 specifically for the cross-AI review loop: external AI writes a markdown findings file, user ingests via `--from-file`, address-reviews walks each finding with pushback discipline. The parser at `references/external-findings-format.md` is the single source of truth for that format.

Internal `/opsx:review` doesn't write markdown — it writes JSON to `.orbit-runs/review-<mode>-<TS>.json`. The JSON's `findings[]` array carries the same conceptual fields (severity, title, file:line, recommendation) but in a different shape (typed JSON objects vs markdown sections with `### <title>` + `**File**:` + `**Description**:`). Both shapes carry the same lifecycle semantics — pushback → classify → fix → ripple-flag → log applies identically.

The mismatch was tolerable in v1 because the cross-AI loop was the friction blocker (manual copy-paste between AIs). The internal loop's friction (writing a resolution log by hand vs having the SKILL produce it) is less acute but real. After ~6 months of dogfooding, the gap is now well-understood (the iter-2 SUGGESTION pattern in `harden-review-mode-recommendations` is the cleanest demonstration).

## Goals / Non-Goals

**Goals:**
- Allow `--from-file <path>` to ingest internal review JSON (`review-<mode>-*.json`).
- Preserve the markdown ingest path byte-identically — zero regression risk for the cross-AI loop.
- Detect format via content sniff (not file extension).
- Clean failure mode when file matches neither format.
- Add a reference doc symmetric with the existing external one — so the two parser contracts are easy to compare and maintain.

**Non-Goals:**
- **Other internal JSONs (audit-drift, address-reviews)** — different walk semantics; out of scope for v1. Add as separate follow-up if/when needed.
- **A standalone markdown↔JSON converter command** — over-engineering for the use case. The auto-detect path inside `--from-file` is sufficient.
- **Updating any external-review workflows** — those use markdown by definition; nothing changes for them.
- **A `--from-internal <path>` separate flag** — auto-detect via content sniff is cleaner UX (D-detect-1).
- **Cycle prevention for address-reviews-*.json ingest** — out of scope; v1's `command: "review"` filter is enough; ingest of an address-reviews JSON would error cleanly via the unsupported-command path. NOTE: if a future change extends the parser to accept `command: "address-reviews"` JSON, cycle-detection guards become required (a walk that produces its own input would loop) — that future change MUST add explicit cycle awareness.

## Decisions

### D-detect-1: Auto-detect format via content sniff (not extension)

**Decision**: Sniff file content to detect format. If first non-whitespace character is `{`, attempt JSON parse. If first non-whitespace begins with `# External Review:`, attempt markdown parse (this is the only markdown format orbit produces — written by external AIs per `/opsx:review-external`'s prompt template; internal `/opsx:review` does NOT emit markdown). Otherwise emit clean format error.

**Why**:
- File extensions are unreliable. `.md` could be JSON inside a markdown template; `.json` could be misnamed. Content is unambiguous.
- Two cheap leading-character checks separate the cases without invoking expensive parsers prematurely.
- Mirrors the way `git` and other tools auto-detect text vs binary on the same flag.

**Alternative considered**:
- **Separate `--from-internal <path>` flag**. Rejected — adds invocation cost (user must remember which flag to use); auto-detect is strictly better UX when the file content is unambiguous.
- **Extension-only detection**. Rejected — fragile if user renames files or pipes content; doesn't compose with future formats.

### D-shape-1: V1 accepts `command: "review"` AND `command: "audit-drift"` JSON

**Decision**: After successful JSON parse, the parser checks the top-level `command` field. If `command: "review"` or `command: "audit-drift"`, proceed. Anything else: emit a clean error message naming the supported shapes and exit without acting.

**Why**:
- The baseline `orbit-run-summary-emit` `Audit-drift standalone recommendations` requirement (synced earlier this session) already mandates audit-drift findings to emit `next_recommended: "/opsx:address-reviews --from-file <this-json>"` — closing the bridge for BOTH review JSON AND audit-drift JSON is exactly what the baseline references this change to do. The initial v1 scope of review-only would have created an internal product contradiction (audit-drift recommends a command the parser refuses to honor); caught by GPT-5 Codex's iter-1 external CRITICAL.
- The audit-drift `findings[]` shape mirrors review's: `severity` / `file` / `line` / `title` / `recommendation` are identical; only the provenance slot (`pass` vs `category`) differs. Lifecycle is identical regardless of source command.
- `address-reviews` ingest of itself stays REJECTED — preserves cycle prevention (the resolution log would feed itself recursively).
- The universal-spine `command` field is the natural discriminator — every orbit-emitted JSON carries it.

**Alternative considered**:
- **Accept review JSON only** (original v1 scope). Rejected on external iter-1 CRITICAL — created an internal product contradiction with the baseline audit-drift recommendation contract. The "different walk semantics" framing turned out to be wrong; the lifecycle is the same regardless of source command.
- **Accept all JSONs that have a `findings[]` array shape** (including `address-reviews`). Rejected — `address-reviews` ingest of itself would loop; cycle prevention requires explicit rejection.

### D-no-op-1: Marker removal is no-op for both virtual marker types

**Decision**: For virtual markers from EITHER markdown OR JSON source, the marker-removal step (Step 3d in the SKILL.md lifecycle) is a no-op. Documented in both reference files.

**Why**:
- Aligns with existing v1 behavior for markdown virtual markers; no new exception to track.
- Virtual markers don't have inline source text (unlike grep-found `@review:` markers); there's nothing to delete on resolution.
- Removing the original review JSON or external markdown on resolution would be wrong (those are audit artifacts that travel with the change into archive).

### D-pushback-1: Fresh pushback against current state IS applied

**Decision**: When ingesting JSON, the parser does NOT skip the pushback step even though the JSON's own `stale_suppressed[]` array already filtered stale findings at review time.

**Why**:
- The JSON's `stale_suppressed[]` captures what was stale **at review time**. Findings in `findings[]` may have become stale **since** (e.g., the user fixed an issue ad-hoc between review and address-reviews; or another commit landed).
- Pushback is one of the three execution disciplines; the same discipline applies to internal-JSON-source findings as to external-markdown-source findings as to inline-grep-found markers.
- Double-pushback is not harmful: the JSON-side filter is a subset of staleness; the lifecycle-side check is a superset.

**Documented in**: SKILL.md walk section + `internal-findings-format.md` parser contract.

### D-format-error-1: Clean refusal on neither-format match

**Decision**: When content sniff yields neither `{` nor `# External Review:` prefix, OR JSON parse fails, OR `command` field isn't `"review"`: emit a clear error message naming both supported formats + exit without acting. Refuse to act on partial parse.

**Concrete error shape**:
```
`--from-file <path>`: unrecognized format.

Supported formats:
- External-review markdown — see references/external-findings-format.md
- Internal review JSON (review-<mode>-*.json) — see references/internal-findings-format.md

Detected: <observed first-line snippet OR "JSON parse failure" OR "command field: <value>">
```

**Why**:
- Half-parse-then-warn is worse than no-parse-then-fail: users can't tell which findings made it through; the resolution log would be unreliable.
- A clear error message names the format files so the user can self-diagnose.

### D-find-1: New reference doc symmetric with external

**Decision**: Create `references/internal-findings-format.md` mirroring the structure of `external-findings-format.md`: "Expected file format" / "Parser contract" / "Malformed input handling" / "Tolerated variations" / "Worked example". Same five sections; analogous content.

**Why**:
- Code symmetry makes the two parser contracts easy to compare and maintain.
- Future format additions (e.g., audit-drift JSON ingest) can follow the same template.
- Each reference doc remains the single source of truth for its format; SKILL.md prose stays high-level.

### D-source-tag: virtual marker `source` tag distinguishes provenance

**Decision**: For markdown virtual markers, the `source` field on the virtual marker is `external` (existing). For JSON virtual markers, the `source` field is `internal-review`. The resolution log captures this on each entry.

**Why**:
- Provenance is useful for audit-trail analysis (which findings came from where; helps when reviewing convergence across iterations).
- Existing v1 already uses `source` for the `external` vs `inline` distinction; adding `internal-review` is a natural third value.
- No behavior change beyond logging — the lifecycle is identical regardless of source.

## Risks / Trade-offs

- **Markdown that starts with `{`** would mis-route to JSON parser. **Mitigation**: very rare in practice; JSON parser fails fast on invalid JSON; user sees a JSON-parse error and can fix the file. Worst case is a confusing error message; never silent corruption.

- **Cross-format auto-detect heuristic could grow** as more formats add support (audit-drift, etc.). **Mitigation**: v1 accepts 2 formats; extending later means adding another leading-character check OR another `command` field branch. Not a structural risk; the sniff layer stays simple as long as formats are visually distinct at the leading character.

- **JSON consumers double-pushback** — the user runs `/opsx:review` (which already did pushback, filtered `stale_suppressed[]`), then `/opsx:address-reviews --from-file` does pushback again on `findings[]`. **By design** (D-pushback-1) — fresh pushback catches state changes since review. Mitigation: not a real risk; double-pushback is cheap and the second pass catches real cases.

- **Reference doc duplication** — `internal-findings-format.md` repeats structure with `external-findings-format.md` (5 sections, analogous content). **Trade-off accepted**: per orbit's "intentional duplication for reliability" principle (per orbit-conventions). Each format gets a self-contained reference; SKILL.md doesn't grow.

- **Versioning of internal JSON shape** — if the universal spine or per-command extensions change in `orbit-conventions`, the parser contract here would need to follow. **Mitigation**: this risk applies symmetrically to the existing external-findings parser when external reviewers' output formats drift. The mitigation pattern is the same: drift-audit Cat 3 catches schema-vs-parser-contract divergence on next run.
