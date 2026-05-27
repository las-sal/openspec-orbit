## Why

`/opsx:address-reviews --from-file <path>` v1 parses external-review markdown only. Internal `/opsx:review` findings — emitted to `openspec/changes/<name>/.orbit-runs/review-<mode>-*.json` — can't be ingested through `--from-file`. After an internal `/opsx:review` with non-trivial findings, the user has no clean lifecycle path: either resolve inline (no pushback discipline, no audit log) or hand-copy findings into the external markdown format. The cost-up-front principle says: the same lifecycle (pushback → classify → fix → ripple-flag → log) that handles external markdown should handle internal JSON.

Issue [#4](https://github.com/las-sal/openspec-orbit/issues/4) has tracked this since 2026-05-18 (bootstrap-openspec-orbit iter-2). Catalyzing case today: `harden-review-mode-recommendations` system-mode iter-2 `--fresh` emitted a SUGGESTION about a run-summary-schema gap that lived in the JSON but couldn't be `--from-file`-ed without manual reformatting.

## What Changes

- **`--from-file` auto-detects format** via content sniff: markdown (existing path, unchanged) OR JSON (new path).
- **JSON ingest path**: parse `findings[]` array from `review-<mode>-*.json` AND `audit-drift-*.json` into virtual markers; identical lifecycle to markdown virtual markers (pushback → classify → fix → ripple-flag → log); marker-removal step is no-op for all virtual-marker types.
- **V1 accepts `command: "review"` OR `command: "audit-drift"`**; other internal JSONs (`address-reviews`, `apply`, `archive`, `propose`, etc.) rejected with a clean error message naming supported shapes. `address-reviews` rejection prevents recursive cycles. Audit-drift inclusion honors the existing baseline contract at `orbit-run-summary-emit` `Audit-drift standalone recommendations` requirement, which mandates audit-drift findings to emit `next_recommended: "/opsx:address-reviews --from-file <this-json>"` — closing the bridge for BOTH JSON families this change targets.
- **Pushback discipline still applies** to JSON findings: the JSON's own `stale_suppressed[]` array filtered stale at review time, but time-since-review staleness still needs fresh verification.
- **New reference doc**: `references/internal-findings-format.md` documenting the parser contract (symmetric structure with `external-findings-format.md`).
- **Failure modes**: clean refusal when neither format matches — refuse to act on partial parse.
- **No break to v1**: existing markdown ingest path is byte-identical in behavior. Closes [#4](https://github.com/las-sal/openspec-orbit/issues/4).

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `orbit-address-reviews`: MODIFIED `--from-file ingest of external-review findings` requirement — extended to cover both markdown AND internal JSON shapes (rename of requirement title to reflect the broader scope). ADDED scenarios for format auto-detect, JSON ingest, unsupported-JSON-command rejection, and shared marker-removal no-op semantics.

## Impact

- **Skill body**: `.claude/skills/openspec-address-reviews/SKILL.md` `--from-file ingest` section extended with auto-detect + JSON path prose + worked example.
- **Command body**: `.claude/commands/opsx/address-reviews.md` mirror updated per orbit-modified-command convention.
- **Reference docs**: NEW `references/internal-findings-format.md`. Light cross-reference update to `references/external-findings-format.md` clarifying it documents ONE of two supported formats.
- **README**: `README.md` updated in two sites — (a) L422 area `--from-file` flag description broadens beyond external-review markdown; (b) L769 area "External-review markdown findings format" sub-header gains a sibling note for internal review JSON pointing at `internal-findings-format.md`.
- **No CLI flag changes**, no new arguments — `--from-file` invocation is unchanged from the user's perspective; the parser does more work internally.
- **No runtime changes** outside the address-reviews skill itself.
- **Issue closure**: closes [#4](https://github.com/las-sal/openspec-orbit/issues/4); side-effect on [#3](https://github.com/las-sal/openspec-orbit/issues/3) (general address-reviews feature roadmap) — one feature struck off the list.
- **Forward-compatible**: extending to additional JSON shapes (audit-drift, etc.) later is straightforward — add another `command` field check + reference doc. Tracked separately if/when needed.
