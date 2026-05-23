## Why

Orbit has diverged enough from upstream `@fission-ai/openspec` that the original "overlay that augments cleanly" framing has become a source of confusion. Most recently, the just-archived `emit-run-summary-jsons-from-workflow-commands` change added `# Orbit additions` sections to 7 of the 9 skills that issue #6 was originally filed to delete — invalidating #6's premise mid-flight.

The strategic call: peg orbit to a specific upstream version (`@fission-ai/openspec@1.3.1`), own the `.claude/` surface explicitly, and stop pretending the codebase needs to be flexible about future upstream ingestion. Orbit becomes a workflow tool using upstream's CLI as a pinned engine — not a delta-only overlay aspiring to upstream compatibility.

This change implements the immediate phase of that strategy (Option 1: declare + pin + reframe + delete dead weight + add orbit-onboard) and closes cluster 2 issues #6 and #23 under the new framing.

## What Changes

**Pegging strategy**:
- Declare `@fission-ai/openspec@1.3.1` as orbit's pinned upstream dependency in CLAUDE.md + README
- Enforce via documentation only (install-script work tracked separately as #26)
- Update orbit-conventions' `Distribution model` requirement to reflect pegging + the new "orbit-owned surface using upstream as engine" framing
- Add convention requirements: upstream version pinning, overlay file disposition (which files orbit ships vs which it omits)

**Narrative reframe** (forward-looking, per explore decision D6):
- Rewrite CLAUDE.md opening to position orbit as a workflow tool using openspec's CLI as a pinned engine
- Rewrite README opening + install section with the same reframe
- Audit and update references to "overlay" terminology where the new framing fits better
- Correct install instructions per #6 (interactive `openspec config profile`, accurate skill counts, remove obsolete `openspec-sync-specs` direct-use mention)

**Overlay cleanup**:
- DELETE `feedback` skill from `.claude/skills/` (truly upstream, no orbit value, no orbit dependency)
- KEEP `openspec-sync-specs` SKILL (still a needed primitive used by orbit's archive flow)
- KEEP `/opsx:sync` command (per address-reviews iter-4 EW1-reversal: upstream lists it as "Optional command", NOT deprecated; under project memory `orbit-supports-full-openspec-1-3-1` we retain optional upstream commands unless real reason to drop)

**Orbit-onboard skill** (closes #23):
- Replace upstream `openspec-onboard` skill body with 100% orbit-authored content (same directory, same name)
- Reference-leaning hybrid style: setup verification + identity statement + canonical-flow walkthrough + quick-reference + try-it nudge
- Lenses introduced contextually during the explore + review phases of the walkthrough
- External-review loop referenced abstractly (points to `openspec-review-external/SKILL.md` for details)
- Update `.claude/commands/opsx/onboard.md` to invoke the orbit-authored skill

**Issue closure**:
- Close #6 under the pegging-strategy framing (original "delete 9 unmodified files" framing superseded)
- Close #23 (orbit-onboard delivered)

**BREAKING**: orbit users no longer expected to run `openspec update` for evolving upstream content. Pinned to v1.3.1.

## Capabilities

### New Capabilities

- `orbit-onboard`: defines the orbit-authored onboarding skill — setup verification, identity statement, canonical-flow walkthrough, quick-reference, try-it nudge. Reference-leaning hybrid style. Slash command surface `/opsx:onboard`.

### Modified Capabilities

- `orbit-conventions`: MODIFIES the `Distribution model` requirement to reflect pegging + orbit-owned-surface framing. MODIFIES the `Internal-run JSON summary format` requirement to remove the obsolete "openspec-orbit#6 will deprecate/remove `/opsx:sync-specs`" framing (per address-reviews iter-2 EW1: sync-specs is retained as a primitive under pegging). ADDS requirements for upstream version pinning and overlay file disposition (which upstream skills + commands orbit ships vs which it omits, under what rationale).
- `orbit-run-summary-emit`: MODIFIES the `Emit scope` requirement to remove the obsolete "openspec-orbit#6 will deprecate/remove `/opsx:sync-specs`" framing (per address-reviews iter-2 EW1: primitive-only retention, user-callable surface pruned).

## Impact

- **Documentation**: CLAUDE.md, README, and any orbit narrative docs require coordinated rewriting to reflect the pegging strategy and identity reframe. Specifically: README opening + "What this is" section, install instructions + verification table, troubleshooting section (around L969), and any prose referencing pre-pegging upstream/orbit accounting (e.g., "11 upstream skills + feedback", "the other 8 upstream openspec-* skills + feedback untouched").
- **Skills**: `feedback` skill deleted; `openspec-onboard` skill body rewritten 100% orbit-authored.
- **Commands**: `.claude/commands/opsx/onboard.md` updated to invoke the new skill body.
- **Specs baseline**: `orbit-conventions` baseline updated; new `orbit-onboard` baseline created on archive sync.
- **Install model**: documented requirement for `@fission-ai/openspec@1.3.1`; manual `cp -r` overlay install unchanged for now (install-script future work as #26).
- **No CLI changes**: orbit continues to use upstream CLI commands as-is; no schema or behavior changes to `openspec status`, `openspec instructions`, `openspec archive`, etc.
- **No runtime changes**: this is a docs + skill-content + cleanup change. Existing workflow command behavior unchanged.
- **Upstream coupling**: future upstream releases (v1.3.2+, v1.4+, etc.) are explicitly out-of-scope for orbit until a deliberate upgrade-and-port change. Bug fixes and improvements in upstream do not propagate to orbit users automatically.
