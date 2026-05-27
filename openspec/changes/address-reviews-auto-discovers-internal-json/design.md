## Context

The just-archived `address-reviews-accepts-internal-json` (closes #4) added the parser machinery for `--from-file` to accept internal review JSON + audit-drift JSON via content-sniff auto-detect. With the parser in place, the user-facing convenience question is: does the user need to type `--from-file <path>` for the internal-JSON case, or can address-reviews discover the JSON itself?

Issue #10 framed the trap: `/opsx:address-reviews <change-name>` after `/opsx:review <change-name>` is the canonical 2-step workflow, but v1 silently fails (0 markers found, JSON ignored) unless the user knows to type `--from-file <path>` OR pre-runs review with `--mark`. This change closes that gap by adding auto-discovery as a fallback when no markers are found.

The change is strictly additive: existing flows (markers, `--from-file`, `--mark`, whole-repo scan) are byte-identical from the user's perspective. Auto-discovery only triggers in a specific niche: change-name positional scope + zero markers found + no explicit `--from-file`.

## Goals / Non-Goals

**Goals:**
- `/opsx:address-reviews <change-name>` "just works" after `/opsx:review <change-name>`.
- No hidden `--from-file <path>` parameter; no `--mark` prerequisite.
- Resolution log captures the auto-discovery decision (source path, source command, recency-comparison evidence).
- Discovery is deterministic: filename `<TS>` token, lexical sort, single global "most recent" wins.

**Non-Goals:**
- **Auto-discovery for whole-repo invocation** (`/opsx:address-reviews` bare). No anchor — no clear single `.orbit-runs/` to discover from. Whole-repo scan stays grep-only.
- **Auto-discovery for path/pattern scope** (`/opsx:address-reviews src/`). Path arguments anchor on a directory but not necessarily a change directory; conflating these would mis-route invocations. Path/pattern scope stays grep-only.
- **Prompt user when multiple candidates exist**. Option C from pre-decision was rejected — adds friction without value when recency is unambiguous. If user wants a non-recent JSON, they pass it explicitly via `--from-file`.
- **Cross-change discovery**. Discovery is scoped to the named change's own `.orbit-runs/` directory — never reaches across change directories.
- **Auto-trigger on `--mark` runs**. `--mark` runs add inline markers; address-reviews finds those markers via the existing grep path. Auto-discovery doesn't compete with `--mark` — it just ensures the no-`--mark` case also works.
- **Stale-JSON warning**. If the most-recent JSON is older than the most-recent `apply-*.json` (artifacts changed since), discovery still picks it. The resolution log records the relative timestamps; the user reads and decides. Adding stale-detection at the discovery layer would duplicate the apply-token logic from the just-archived `harden-review-mode-recommendations`.

## Decisions

### D-discovery-1: Auto-discovery triggers only on change-name positional scope

**Decision**: Auto-discovery runs ONLY when the user invokes `/opsx:address-reviews <change-name>` (positional argument resolves to an existing change directory at `openspec/changes/<name>/` or `openspec/changes/archive/<date>-<name>/`). Bare invocation OR path/pattern scope: no auto-discovery; grep-only.

**Why**:
- Change-name positional is the unambiguous case: there's exactly one `.orbit-runs/` to discover from; intent is clear.
- Bare invocation is a whole-repo marker scan; injecting auto-discovery would conflate scopes.
- Path/pattern scope is an ad-hoc grep with no `.orbit-runs/` anchor.

**Alternative considered**:
- Allow path/pattern scope to opportunistically auto-discover when the path happens to point inside a change directory. Rejected — too magic; user intent is ambiguous; path-vs-name disambiguation gets brittle.

### D-priority-1: Discovery order — markers, then JSON, then empty

**Decision**: When `/opsx:address-reviews <change-name>` runs, resolve input in this order:
1. Grep the change directory for `@review:` markers. If any found → walk them (current behavior; auto-discovery does NOT fire).
2. If no markers AND no `--from-file` flag: look in `openspec/changes/<name>/.orbit-runs/` for `review-<mode>-*.json` files AND `audit-drift-*.json` files. Take the single most-recent by filename `<TS>` token. If found → walk it as `--from-file` would.
3. If neither markers nor JSON: report `No @review: markers in scope and no internal review/audit-drift JSON in .orbit-runs/. Nothing to walk.` and exit cleanly.

**Why**:
- Markers-first preserves current behavior — the `--mark` workflow keeps producing source-level markers that grep finds; no regression.
- JSON-fallback closes the no-`--mark` convenience gap.
- Empty-exit is the clean "no work" state.

**Alternative considered**:
- JSON-first, markers-fallback. Rejected — would change current behavior; existing flows expecting markers-priority would surprise users.
- Walk both if both exist. Rejected — over-engineering; if the user wants both, they invoke twice (once for each source).

### D-recency-1: Most recent by filename `<TS>` token, regardless of `command` (Option A)

**Decision**: When discovery considers candidate JSONs in `.orbit-runs/`, pick the single most-recent by filename `<TS>` token (lexical sort works because tokens are `YYYY-MM-DDTHH-MM-SSZ`). Both `review-<mode>-*.json` AND `audit-drift-*.json` compete on the same recency axis; the `command` value of the winner is read from the file content.

**Why**:
- Simple, consistent — one rule fits all candidate types.
- Filename `<TS>` token is the canonical orbit-conventions sorting axis (per `Internal-run JSON summary format` requirement); no new convention introduced.
- If user wants a specific (non-recent) JSON, `--from-file <path>` is the explicit override (existing v1 behavior, unchanged).

**Alternatives considered**:
- **Option B**: prefer review JSON over audit-drift; most recent within preferred class. Rejected — adds a hidden preference rule; surprises users who expect "most recent" to mean "most recent". The audit-drift case is just as worthy a walk target as review.
- **Option C**: prompt user via `AskUserQuestion` when multiple candidates exist. Rejected — adds friction without value when recency is unambiguous; turns a 1-command workflow into 2.

**Trade-off accepted**: a user who has just run both `/opsx:review` AND `/opsx:audit-drift` and wants to walk the review (not the audit-drift) findings will have to pass `--from-file` explicitly if audit-drift is more recent. Edge case; explicit-override is the documented escape hatch.

### D-explicit-override: `--from-file` always overrides auto-discovery

**Decision**: If `--from-file <path>` is specified, address-reviews uses that path verbatim. Auto-discovery does NOT run, regardless of marker presence or JSON availability.

**Why**:
- Explicit user intent wins. Always.
- Preserves the cross-AI loop (external markdown via `--from-file`) without surprise.
- Lets users override the recency-based pick if they want a specific JSON.

### D-no-mark-prerequisite: `--mark` becomes optional, not required

**Decision**: The framing of `--mark` shifts from "prerequisite for the address-reviews workflow" to "optional convenience for diff-readability". Documentation updates accordingly.

**Why**:
- The trap that #10 surfaced was the silent `--mark`-required-but-not-obvious state.
- Auto-discovery makes `--mark` purely a stylistic choice (do you want markers in your diff?), not a structural requirement.
- The cost is documentation work, not behavior change — `--mark` still drops markers; address-reviews still walks them via grep.

### D-no-stale-detection: Don't duplicate the apply-token staleness logic at discovery time

**Decision**: Discovery picks the most-recent JSON regardless of whether the change's `apply-*.json` tokens are newer (which would mean artifacts changed since the JSON was written). The resolution log records the source-JSON timestamp and the most-recent apply timestamp; the user reads and decides.

**Why**:
- The just-archived `harden-review-mode-recommendations` already implements stale-detection for system-mode review's State 5. Duplicating that logic at discovery would couple two skills that should stay independent.
- The intent of auto-discovery is convenience, not gate-keeping. If a JSON is stale, the lifecycle's pushback discipline (already mandatory per orbit-conventions) catches the stale findings during the walk.
- A stale-warning at discovery time would add prose without changing behavior; the walk still happens; the user still sees the findings.

**Trade-off accepted**: a user with both a stale JSON and a recent `apply-*.json` gets the JSON walked, possibly with many findings already-resolved-at-HEAD. Pushback catches these as ✓ Stale entries. Acceptable cost.

## Risks / Trade-offs

- **Surprise when both review JSON and audit-drift JSON exist with similar timestamps**. The user thinks "address-reviews on this change" means review findings, but auto-discovery picked audit-drift because it's 30 seconds newer. Mitigation: resolution log surfaces source-command name + path explicitly; user can re-invoke with `--from-file <review-path>` if they wanted the other. Edge case; documented.

- **The marker-removal-becomes-no-op-when-virtual gets one more provenance variant in code paths**. Now we have inline / external-markdown-virtual / internal-review-JSON-virtual (via `--from-file`) / internal-review-JSON-virtual (via auto-discovery) / audit-drift-JSON-virtual (via `--from-file`) / audit-drift-JSON-virtual (via auto-discovery). Mitigation: the no-op behavior is uniform across all virtual-marker subtypes; the `source` tag distinguishes provenance in the log but the lifecycle logic doesn't branch.

- **Drift between SKILL prose and actual behavior** if auto-discovery surfaces edge cases not captured in spec scenarios. Mitigation: 8 net-new scenarios cover the happy path (1) + 7 edge cases (markers-found-precedence / no-candidates / explicit `--from-file` override / multiple candidates with recency tie-break / standard ingest lifecycle / `--mark` no-longer-prerequisite / archive-location respected).

- **Reduces visibility of where findings come from**. Before this change, a `/opsx:address-reviews <name>` either found markers (visible in git diff) or said "nothing to do" (clear). After this change, it might silently discover a JSON and walk findings the user hasn't seen yet. Mitigation: triage step (Step 2 in the SKILL lifecycle) presents the discovered findings as a numbered list before walking begins; user reads them then; auto-discovery doesn't bypass triage.
