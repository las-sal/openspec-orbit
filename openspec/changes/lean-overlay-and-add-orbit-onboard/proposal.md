## Why

Orbit has diverged enough from upstream `@fission-ai/openspec` that the original "overlay that augments cleanly" framing has become a source of confusion. Most recently, the just-archived `emit-run-summary-jsons-from-workflow-commands` change added `# Orbit additions` sections to 7 of the 9 skills that issue #6 was originally filed to delete — invalidating #6's premise mid-flight.

The strategic call: peg orbit to a specific upstream version (`@fission-ai/openspec@1.3.1`), own the `.claude/` surface explicitly, and stop pretending the codebase needs to be flexible about future upstream ingestion. Orbit becomes a workflow tool using upstream's CLI as a pinned engine — not a delta-only overlay aspiring to upstream compatibility.

This change implements the pegging-strategy core of cluster 2 and closes #6 under that new framing. Scope cut mid-implementation (2026-05-23): the orbit-onboard skill rewrite (originally bundled to close #23) is deferred to a follow-up change after sandbox-install verification surfaced premise problems requiring a re-explore with accurate v1.3.1 install-surface knowledge.

## What Changes

**Pegging strategy**:
- Declare `@fission-ai/openspec@1.3.1` as orbit's pinned upstream dependency in CLAUDE.md + README
- Enforce via documentation only (install-script work tracked separately as #26)
- Update orbit-conventions' `Distribution model` requirement to reflect pegging + the new "orbit-owned surface using upstream as engine" framing
- Add convention requirements: upstream version pinning, overlay file disposition (which files orbit ships vs which it omits)

**Narrative reframe** (forward-looking, per explore decision D6):
- Rewrite CLAUDE.md opening to position orbit as a workflow tool using openspec's CLI as a pinned engine
- Rewrite README opening with the same reframe
- Audit and update references to "overlay" terminology where the new framing fits better

**Overlay cleanup**:
- DELETE `feedback` skill from `.claude/skills/` (truly upstream, no orbit value, no orbit dependency)
- Sweep stale spec/skill prose that referenced the deleted feedback skill or the old "Distribution model — overlay, not CLI fork" framing

**Issue closure**:
- Close #6 under the pegging-strategy framing (original "delete 9 unmodified files" framing superseded)

**Deferred to follow-up**:
- Orbit-authored `openspec-onboard` skill body (closes #23) — deferred. The 2026-05-23 sandbox-install verification surfaced that v1.3.1's actual install surface differs materially from the assumptions baked into the original orbit-onboard spec (Setup verification scenarios referenced files that aren't installed by upstream; expanded-profile CLI path doesn't exist). The orbit-onboard work needs a fresh `/opsx:explore` cycle with accurate v1.3.1 surface knowledge before re-scoping. #23 stays open.
- README install-section corrections + expanded-profile documentation (originally chunks 3+ of this change) — also deferred to the follow-up cycle alongside orbit-onboard, since both depend on the same v1.3.1 install-surface re-grounding.

**BREAKING**: orbit users no longer expected to run `openspec update` for evolving upstream content. Pinned to v1.3.1.

## Capabilities

### Modified Capabilities

- `orbit-conventions`: RENAMES `Distribution model — overlay, not CLI fork` → `Distribution model — pegged engine + orbit-owned surface` and MODIFIES the requirement body to reflect pegging + orbit-owned-surface framing. ADDS requirements for upstream version pinning and overlay file disposition (which upstream skills orbit ships vs which it omits, under what rationale).

## Impact

- **Documentation**: CLAUDE.md and README opening sections rewritten to reflect the pegging strategy and identity reframe. Stale-prose sweeps applied to several `.claude/` files that referenced the deleted feedback skill or the old "overlay, not CLI fork" framing.
- **Skills**: `feedback` skill deleted from the overlay.
- **Specs baseline**: `orbit-conventions` baseline updated on archive sync.
- **Install model**: documented requirement for `@fission-ai/openspec@1.3.1`; manual `cp -r` overlay install unchanged for now (install-script future work as #26). Detailed install-section rewrite deferred to the follow-up change that re-explores orbit-onboard.
- **No CLI changes**: orbit continues to use upstream CLI commands as-is; no schema or behavior changes to `openspec status`, `openspec instructions`, `openspec archive`, etc.
- **No runtime changes**: this is a docs + cleanup change. Existing workflow command behavior unchanged.
- **Upstream coupling**: future upstream releases (v1.3.2+, v1.4+, etc.) are explicitly out-of-scope for orbit until a deliberate upgrade-and-port change. Bug fixes and improvements in upstream do not propagate to orbit users automatically.
