## Context

Cluster 2 of v0.1.0 grouped two related issues: #6 (overlay ships unmodified upstream files — should be delta-only) and #23 (`/opsx:onboard` should walk users through orbit's extended workflow). Both target the boundary between orbit's overlay surface and upstream OpenSpec.

A premise-invalidation finding emerged mid-explore (2026-05-21): the just-archived `emit-run-summary-jsons-from-workflow-commands` change added `# Orbit additions` sections to 7 of the 9 skills #6 was filed to delete. Only `feedback` and `openspec-sync-specs` remain truly unmodified upstream copies. This made #6's original framing ("delete 9 unmodified files to preserve upstream-update flow") incoherent — the very change that just shipped *broke* the "unmodified" predicate for 7 of the targets.

The user's strategic call (2026-05-22): peg orbit to a specific upstream version, accept that orbit has become a different tool sharing concepts with upstream, optimize for the user's actual workflow rather than upstream-ingest flexibility. This collapses cluster 2's complexity significantly — many of the original questions (shipping model, file-list verification, install instructions) either dissolve or become trivial under pegging.

Orbit is infrastructure for the user's real engineering project (homeENV); ship cluster 2 good-enough and move on. Don't perfectionate.

**Mid-implementation scope cut (2026-05-23):** A second premise-invalidation surfaced during task 3.4's mandatory fresh-sandbox install verification. The actual v1.3.1 install surface diverged from what spec content assumed (10 skills + 10 commands installed by `openspec init --tools claude`, NOT the "11 upstream + feedback" #6 body claimed; the expanded-profile CLI path documented in #6 does not exist; `feedback` and `opsx/sync.md` are never installed by upstream init at all). Rather than silently re-edit reviewed spec language to match the corrected facts, the change was cut to its already-coherent chunks 1+2 (pegging declaration + narrative reframe + feedback deletion + stale-prose sweep) and the orbit-onboard work (originally chunks 3-5) was deferred to a follow-up change that re-explores from accurate v1.3.1 install-surface knowledge. #6 closes here; #23 stays open and returns in the follow-up.

## Goals / Non-Goals

**Goals:**

- Declare orbit pegged to `@fission-ai/openspec@1.3.1` in CLAUDE.md + README
- Reframe orbit's identity narrative (forward-looking, Option β) to survive the eventual Option 2 cleanup without re-work
- Close #6 under the pegging-strategy framing (not the original delete-9-files framing)
- Delete the `feedback` skill from the overlay (no orbit value)

**Non-Goals:**

- **Orbit-authored `openspec-onboard` skill body** — originally bundled here as the #23-closing deliverable; deferred to a follow-up change after the 2026-05-23 install-surface premise-invalidation. The follow-up re-enters `/opsx:explore` with accurate v1.3.1 install-surface knowledge captured in project memory, rather than silently revising already-reviewed spec content to match corrected facts. #23 stays open.
- **README install-section rewrite + expanded-profile documentation** — originally chunks 3 of this change; also deferred to the follow-up, since the install-section content depends on the same v1.3.1 surface re-grounding as orbit-onboard.
- **Option 2 work** — dropping the `# Orbit additions` pattern as a discipline; orbit's skills becoming purely "orbit's skills" without the upstream-derived layering annotation. Tracked as [openspec-orbit#27](https://github.com/las-sal/openspec-orbit/issues/27) for a future explore/change cycle.
- **Option 3/4** — hard-forking or replacing the upstream CLI binary. Reserved for when the CLI contract becomes painful (it isn't yet).
- **Install script with version-check enforcement** — tracked as [#26](https://github.com/las-sal/openspec-orbit/issues/26). Doc-only pin is sufficient for now.
- **Upstream version upgrade (>1.3.1)** — explicitly deferred. Will be a deliberate, separately-proposed change when upstream evolves in a way that earns the upgrade-and-port cost.
- **Renaming orbit-authored skills from `openspec-*` to `orbit-*`** — if/when it happens, do it all at once in its own change; don't partially rename now. Tracked as [openspec-orbit#28](https://github.com/las-sal/openspec-orbit/issues/28).
- **Refactoring orbit's archive flow to use `openspec archive` CLI sync absorption** — current pattern (sync-specs subagent + manual `mv`) works and is not deviating from upstream behavior; no follow-up issue needed.

## Decisions

### D-arch-1: Pegging strategy — Option 1 now, Option 2 next cycle, Option 3/4 deferred

**Decision**: Orbit pegs to `@fission-ai/openspec@1.3.1`. Stops trying to support flexible upstream-version ingestion. Owns the `.claude/` overlay surface explicitly; upstream supplies the CLI as a pinned engine.

**Why now (Option 1)**: The "overlay" framing has been the source of confusion (literally caused #6's premise to be wrong) and the cost of pretending upstream-flexibility matters has been paid in repeated cluster-2 explore loops. Pegging is what's actually happening in practice — formalize it.

**Why Option 1 (not 2/3/4)**:
- **Option 2** (drop `# Orbit additions` pattern, rename to `orbit-*`): one-time cleanup with real coordination cost. Doable as next cycle after Option 1 lands.
- **Option 3** (hard-fork upstream repo): weeks of engineering for full CLI ownership. Not justified until CLI contract becomes painful.
- **Option 4** (reimplement CLI entirely): months of engineering. Reserved for "orbit's needs diverge from openspec at the schema level."

**Alternative considered**: keep aspiring to upstream-flexibility and try to actually make the overlay delta-only as #6 originally framed. Rejected because the cost is permanently high (every orbit-additions change has to maintain delta-only discipline) for a property the user doesn't value.

### D-arch-2: #6 closes under the pegging-strategy framing

**Decision**: Close #6 with a note that the underlying concern (overlay-overwrite risk for upstream-updated files) is addressed structurally by pegging. The original framing ("delete 9 unmodified files") is moot — under pegging, there's no upstream-update flow to preserve.

**Why**: #6 was filed pre-pegging-decision. Its remediation under the original framing would have been mostly destructive (delete 9 files) with knock-on README work. Under pegging, the remediation is constructive (declare pin + reframe + delete only `feedback`) with smaller knock-on work. Net win.

### D-arch-3: Narrative reframe is forward-looking (Option β)

**Decision**: Rewrite CLAUDE.md and README opening to position orbit as "a workflow tool with its own surface, using `@fission-ai/openspec@1.3.1` as a pinned CLI engine" — dropping or heavily qualifying the "overlay" framing.

**Why β over α (minimal edit) or γ (defer)**:
- **α** (keep "overlay" language, add Pegging section): does some work without the full payoff. Will need re-reframing during Option 2 cycle anyway. Pays the cost twice.
- **γ** (defer narrative work entirely to Option 2 cycle): cheapest now but leaves CLAUDE.md actively contradicting reality (still describes orbit as an "overlay" while we just pinned upstream). Acceptable but ugly.
- **β** (forward-looking rewrite now): half-day of writing across CLAUDE.md + README; survives Option 2 without re-work; fixes the source of confusion that caused #6's bad premise.

### D-arch-4: Delete `feedback` skill from overlay

**Decision**: DELETE `feedback` from `.claude/skills/`. Sends user feedback upstream to Fission-AI; serves orbit's pegged-user mission near-zero. Orbit only references it from the soon-to-be-replaced upstream `openspec-onboard` skill.

(All other upstream skills + commands stay implicitly under project memory [[orbit-supports-full-openspec-1-3-1]] — orbit fully and cleanly supports the openspec@1.3.1 functionality set unless there's a real reason to deviate. `feedback` is the only deviation in scope for this change.)

### D-pin-1: Pin to `@fission-ai/openspec@1.3.1`

**Decision**: Pin orbit to `@fission-ai/openspec@1.3.1` exactly.

**Why this version**: Both the latest npm release (as of 2026-05-21) AND what's been tested against during all recent orbit work (#6's verification on 2026-05-19; the just-archived emit change throughout 2026-05-21). Release cadence is ~one minor per month-or-two, so pinning at 1.3.1 doesn't leave us catastrophically behind.

### D-pin-2: Doc-only enforcement for this change

**Decision**: Pin enforcement is documentation-only. README states the required version. CLAUDE.md states the required version. Nothing in the install flow or runtime detects/warns about version mismatch.

**Why**: Orbit has no formal installer (README walks the user through `cp -r` manually). Building an install-script-with-version-check would be a larger scope expansion. Filed as separate enhancement: [#26](https://github.com/las-sal/openspec-orbit/issues/26).

When #26 lands, it'll have a pinned version to check against (this change provides that).

### D-conventions-1: orbit-conventions changes

**Decision**: This change modifies orbit-conventions' `Distribution model` requirement and adds 2 new requirements:

1. **MODIFY** `Distribution model` — replace "overlay, not CLI fork" framing with pegged-engine framing. Rename to `Distribution model — pegged engine + orbit-owned surface` or similar. Scenarios updated to describe the pegging strategy.

2. **ADD** `Upstream version pinning` — codify that orbit declares + enforces (doc-only for now) a specific pinned upstream version. Scenarios cover: version declared in CLAUDE.md + README; install-time check is future work tracked elsewhere; version-upgrade is a deliberate change-proposal event.

3. **ADD** `Overlay file disposition` — codify the rule for what files orbit ships in `.claude/` (both skills AND commands per address-reviews iter-2 EW3). Four categories:
   - **Orbit-authored** (full ownership): orbit ships the file, no upstream-derived content
   - **Orbit-modified with `# Orbit additions`** (current pattern for some skills): orbit ships upstream body + appended additions (this pattern is transitional under Option 2)
   - **Upstream-required primitive** (kept verbatim because orbit depends on it as a callable primitive)
   - **NOT shipped** (rare — applied per-file when an upstream skill provides no orbit-mission value): e.g., `feedback` — removed in this change

**Baseline-language drift deferred** (per address-reviews iter-5/iter-6 per Codex EW1' principle):

4. ~~**MODIFY** `Internal-run JSON summary format`~~ + ~~**MODIFY** (in `orbit-run-summary-emit` capability delta) `Emit scope`~~ — **REMOVED**: both originally proposed to rewrite parenthetical text in baseline specs that prematurely claimed `/opsx:sync-specs` would be "deprecated/removed by openspec-orbit#6". Under [[orbit-supports-full-openspec-1-3-1]], orbit doesn't author requirements about upstream behavior orbit inherits implicitly. The baseline stale text persists as documentation drift to be addressed in a future doc-hygiene change — NOT load-bearing for this change's cluster-2 goals.

(Numbering note: item 5 was merged into item 4 during iter-6 cleanup; sequence continues at item 6 below.)

**Overlay install model implication** (added in address-reviews iter-2 per EC1; refined per iter-4 EW1-reversal):

6. The overlay install mechanism (`cp -r`) does NOT delete files in the user's target project that don't exist in orbit's source. So orbit deleting `.claude/skills/feedback/` from its repo only ensures that file won't be *added* by overlay; it doesn't actively remove pre-existing copies. README install + update + uninstall sections SHALL document an explicit `rm -rf .claude/skills/feedback` command users run after the overlay copy to keep user-project state consistent with overlay intent. **NOTE (2026-05-23 cut)**: this README documentation work was originally scheduled in chunk-3 task 3.5 and is now deferred to the follow-up change alongside the broader install-section rewrite. The convention requirement codifying the user-side prune obligation stays in scope (codified in the `Overlay file disposition` ADDED requirement); only the README implementation defers.

**Address-reviews iter-4 EW1-reversal** (significant correction):

7. The EW1 decision in address-reviews iter-2 to delete `.claude/commands/opsx/sync.md` was REVERSED in iter-4. Original framing claimed sync was "deprecated upstream" — propagated from issue #6's body. **Pushback verification against upstream docs (https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md) at iter-4 showed `/opsx:sync` is listed as "Optional command" — NOT deprecated.** Under the project memory `orbit-supports-full-openspec-1-3-1` ("default scope: fully and cleanly support the openspec@1.3.1 functionality set unless there's a real reason not to"), there is no real reason to drop sync. Reverted EW1's sync-pruning actions: kept `/opsx:sync` command file; restored sync.md to orbit-modified category in `Overlay file disposition`; simplified spec language in `Internal-run JSON summary format` + `Emit scope` to drop "primitive only" + "deprecation" framing. Lesson: pushback discipline must verify claims against current upstream state before propagating; prior "deprecated" claims that pre-date current upstream docs cannot be trusted.

## Risks / Trade-offs

- **Pegging means no upstream improvements** — orbit users freeze at 1.3.1 until a deliberate upgrade-and-port change. Acceptable trade-off per user's explicit framing ("more important to develop correct behavior than ingest new versions").

- **Narrative reframe is forward-looking but Option 2 not yet defined** — D-arch-3 commits to language that anticipates Option 2's cleanup, but Option 2's exact shape isn't specced yet. Risk: the narrative could mention concepts (e.g., "orbit-owned skill set") that don't match Option 2's eventual decisions. Mitigation: keep the reframe high-level enough to survive any reasonable Option 2 shape.

- **Doc-only pin enforcement is honor-system** — a user running a different upstream version won't be warned. Acceptable for now (small user base); install-script in #26 will close this.

- **Per-skill prune judgment** — `feedback` is the one skill this change prunes, on the criterion "no orbit-mission value". Future prune decisions for upstream skills require similar concrete justification per [[orbit-supports-full-openspec-1-3-1]]. Mitigated by the new `Overlay file disposition` requirement codifying the categories.

- **Orbit-onboard deferred to follow-up** — the original cluster-2 plan would have closed #23 in this change with the orbit-authored `openspec-onboard` skill body. Cutting it means cluster 2's onboarding story stays incomplete pending the follow-up explore cycle. Acceptable trade-off: shipping correct pegging now is more valuable than shipping orbit-onboard content that was written against incorrect v1.3.1 install assumptions. Risk mitigation: install-surface findings captured in project memory (`openspec_1_3_1_actual_install_surface`) so the follow-up explore enters with accurate facts.
