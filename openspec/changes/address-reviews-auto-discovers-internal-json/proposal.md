## Why

`/opsx:address-reviews <change-name>` after a `/opsx:review` invocation should "just work" — the review just wrote findings to `.orbit-runs/review-<mode>-*.json`, and the natural follow-up is to walk them. But v1 (just-archived #4) requires explicit `--from-file <path>` for the internal-JSON case, OR pre-running `/opsx:review --as proposal --mark` to drop inline `@review:` markers. Without either, address-reviews silently finds zero markers and the JSON findings become read-only.

Issue [#10](https://github.com/las-sal/openspec-orbit/issues/10) (filed 2026-05-19 during `bootstrap-orbit-status-cli` dogfooding) explicitly noted: after `/opsx:review` emitted 10 findings to `.orbit-runs/review-proposal-*.json`, the natural next-step `/opsx:address-reviews <name>` would have found 0 markers — even though the JSON was right there. The just-archived #4 added the parser (`--from-file` accepts the JSON); this change closes the convenience loop by making the parser auto-trigger.

Cost-up-front principle: orbit's canonical "review → address-reviews" workflow should be 2 commands, not 3 (no implicit `--mark` pre-step) and not have a hidden `--from-file <path>` parameter.

## What Changes

- **Auto-discovery**: `/opsx:address-reviews <change-name>` (positional scope) resolves its input source in priority order:
  1. `@review:` markers in scope (current behavior; unchanged)
  2. If no markers found AND no `--from-file` specified: discover the most-recent `.orbit-runs/review-<mode>-*.json` OR `audit-drift-*.json` in the change's directory, use as input
  3. If neither: report "no findings to walk" and exit cleanly
- **Recency rule**: most recent by filename `<TS>` token (lexical sort works because tokens are `YYYY-MM-DDTHH-MM-SSZ`), regardless of `command` field. Single global "most recent" wins — review and audit-drift compete on the same recency axis.
- **`--from-file` always overrides auto-discovery**: explicit user intent wins; the existing cross-AI loop (external markdown via `--from-file`) is unchanged.
- **`--mark` becomes optional, not prerequisite**: the JSON IS the source of truth for internal flows. Source-level markers remain useful for diff-readability use cases.
- **No auto-discovery for whole-repo invocation OR path/pattern scope**: discovery only applies when `<scope>` is a change name (anchors on `.orbit-runs/`). Bare invocation (`/opsx:address-reviews` no positional) and path/pattern scope keep current grep-only behavior — there's no `.orbit-runs/` to discover from.
- **Marker-removal stays no-op for auto-discovered JSON virtual markers** (same as `--from-file` virtual markers; same lifecycle).
- **Resolution log fully captures the auto-discovery decision**: source path, source command, recency-comparison result included in the `address-reviews-<TS>.json` emit.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `orbit-address-reviews`: MODIFIED `Address-reviews command available` requirement — extends the discovery step with auto-discovery as a fallback when no markers found. ADDED scenarios for: auto-discovery happens / multiple candidates resolved by recency / discovery finds nothing / explicit `--from-file` overrides discovery / whole-repo invocation skips discovery / path/pattern scope skips discovery.
- `orbit-run-summary-emit`: MODIFIED `Audit-drift standalone recommendations` requirement — change-scoped audit-drift findings recommendation drops the `--from-file <this-json>` argument (now `/opsx:address-reviews <name>`) since auto-discovery handles the JSON lookup. Project-wide recommendation unchanged (no change-directory anchor; `--from-file` still required). Codifies the change-trigger that baseline L412 already names — "the `--from-file` flag becomes optional once openspec-orbit#10 lands". Caught by external GPT-5 Codex system-mode iter-1 SUGG 2.

## Impact

- **Skill body**: `.claude/skills/openspec-address-reviews/SKILL.md` Step 1 (Discover) gets a new fallback sub-step for change-name-scoped invocations with no markers.
- **Command body**: `.claude/commands/opsx/address-reviews.md` mirror updated; "What it does" Step 1 description broadens.
- **Reference docs**: `references/internal-findings-format.md` lightly updated to note that this parser contract is also consumed by auto-discovery, not just explicit `--from-file`.
- **README**: existing `/opsx:address-reviews` description (~L417 area) gains a note that `<change-name>` invocation auto-discovers internal JSON.
- **No CLI flag changes**, no new arguments. `--mark` stays defined but no longer-prerequisite framing is documented.
- **Strict superset of #4**: #4 added the parser; #10 adds the discovery. Both are now archived/landing in this change's wake.
- **Issue closure**: closes [#10](https://github.com/las-sal/openspec-orbit/issues/10).
- **No runtime changes** outside the address-reviews skill prose.
