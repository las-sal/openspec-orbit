## Why

The README.md Installation, Updating, and Uninstalling sections (L898-L1020) carry pre-pegging and pre-emit-change accounting that contradicts v1.3.1's actual install surface — skill counts wrong, "modified upstream skills" list stale (says 3, reality is ~10), the missing `rm -rf .claude/skills/feedback` prune step that produces the specific user-visible bug from cluster 2, and three dead `#pegging-strategy` anchors at L54, L902, L937 pointing to a section that doesn't exist.

This was originally chunks 3 of the just-archived `lean-overlay-and-add-orbit-onboard` change. It was cut on 2026-05-23 after sandbox-install verification surfaced that v1.3.1's actual install surface differs materially from the assumptions baked into the original spec content. The pegging-strategy core landed in chunks 1+2; this change closes the install-doc tail with accurate, sandbox-verified content.

## What Changes

**README install-section rewrite (L898-L1020)**:
- Correct skill/command counts: 10 upstream + 4 orbit = 14 directories total (not 11 + 4 = 15); same for commands
- Update modified-skills list: orbit now adds `# Orbit additions` to ~10 skills (not the 3 the README claims)
- Add `rm -rf .claude/skills/feedback` prune step to install, update, and uninstall sections — addresses the `cp -r` overlay's inability to delete target-only files
- Fix L916 install command to include `--tools claude` flag (bare `init` errors out non-interactively per propose-time sandbox D5)
- Replace overly-cautious `init --force` warning with accurate scope statement (init --force restores upstream-installed skills/commands but preserves user-created `.claude/` content and all `openspec/` content per propose-time sandbox D7)
- Add `### Pegging strategy` subsection — anchor target for 3 live forward-references at L54, L902, L937 (currently dead)
- Sweep partial-adoption and coexistence prose for stale "3 modified skills" enumeration

**Apply-time verification**:
- Tight fresh-sandbox verification (`mktemp -d` + same Node version + the exact CLI commands the rewritten README documents)
- No re-verification of the broader install-surface state (sandbox-verified less than 48 hours ago, captured in project memory)

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
- `orbit-conventions`: ADD one requirement codifying "README install/update/uninstall sections SHALL describe the actual install surface produced by the pinned upstream version." Same drift bug has now caused two cluster-2 cuts; the rule belongs in baseline, not as an unwritten discipline. Five scenarios cover: skill/command counts match install output, CLI invocations are non-interactively executable, README-modifying changes pair with sandbox verification, upgrade/overlay-change proposals include README synchronization, and user-facing behavior warnings are accurate (neither understated nor overstated).

## Impact

- **Documentation**: README.md L898-L1020 fully rewritten. ~120 lines touched. Three previously-dead anchors (`#pegging-strategy`) become live.
- **Spec deltas**: one small ADDED requirement to `orbit-conventions` codifying the README-matches-install-surface rule (~20 lines of spec text). Sync applies on archive; baseline gains a new requirement.
- **No code changes, no skill rewrites, no command rewrites.**
- **Apply-time sandbox verification**: required. The specific CLI commands the rewritten README documents (`openspec init --tools claude`, `cp -r` overlay sequence, `rm -rf .claude/skills/feedback`, update flow, `init --force --tools claude` uninstall) must execute as documented in a fresh sandbox before the change is considered apply-complete.
- **Sets up the orbit-onboard follow-up**: the next change (orbit-authored `openspec-onboard` SKILL body) can reference the now-correct README install steps as documented ground truth, eliminating the premise-invalidation risk that cut the previous bundled attempt.
- **Issue tracking**: this change closes cluster-2's residual install-doc tail. #23 (orbit-onboard SKILL body) stays open for the follow-up. #26 (install-script with version-check) remains unchanged — this change keeps doc-only pin enforcement.
- **Upstream coupling**: no behavioral change. Same `@fission-ai/openspec@1.3.1` pin; same CLI invocations; only prose accuracy.
