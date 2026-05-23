## Context

Cluster 2 of v0.1.0 grouped two related issues: #6 (overlay ships unmodified upstream files — should be delta-only) and #23 (`/opsx:onboard` should walk users through orbit's extended workflow). Both target the boundary between orbit's overlay surface and upstream OpenSpec.

A premise-invalidation finding emerged mid-explore (2026-05-21): the just-archived `emit-run-summary-jsons-from-workflow-commands` change added `# Orbit additions` sections to 7 of the 9 skills #6 was filed to delete. Only `feedback` and `openspec-sync-specs` remain truly unmodified upstream copies. This made #6's original framing ("delete 9 unmodified files to preserve upstream-update flow") incoherent — the very change that just shipped *broke* the "unmodified" predicate for 7 of the targets.

The user's strategic call (2026-05-22): peg orbit to a specific upstream version, accept that orbit has become a different tool sharing concepts with upstream, optimize for the user's actual workflow rather than upstream-ingest flexibility. This collapses cluster 2's complexity significantly — many of the original questions (shipping model, file-list verification, install instructions) either dissolve or become trivial under pegging.

Orbit is infrastructure for the user's real engineering project (homeENV); ship cluster 2 good-enough and move on. Don't perfectionate.

## Goals / Non-Goals

**Goals:**

- Declare orbit pegged to `@fission-ai/openspec@1.3.1` in CLAUDE.md + README
- Reframe orbit's identity narrative (forward-looking, Option β) to survive the eventual Option 2 cleanup without re-work
- Close #6 under the pegging-strategy framing (not the original delete-9-files framing)
- Delete the `feedback` skill from the overlay (no orbit value)
- Replace upstream `openspec-onboard` skill body with orbit-authored reference-leaning hybrid content
- Close #23

**Non-Goals:**

- **Option 2 work** — dropping the `# Orbit additions` pattern as a discipline; orbit's skills becoming purely "orbit's skills" without the upstream-derived layering annotation. Tracked as [openspec-orbit#27](https://github.com/las-sal/openspec-orbit/issues/27) for a future explore/change cycle.
- **Option 3/4** — hard-forking or replacing the upstream CLI binary. Reserved for when the CLI contract becomes painful (it isn't yet).
- **Install script with version-check enforcement** — tracked as [#26](https://github.com/las-sal/openspec-orbit/issues/26). Doc-only pin is sufficient for now.
- **Interactive guided tour for orbit-onboard** — over-built for current audience (primarily AI sessions + future-self). Future enhancement if reference-leaning proves insufficient.
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

### D-arch-5: Replace `openspec-onboard` body in-place (option α)

**Decision**: The new orbit-authored onboarding skill takes the same directory and name as upstream's: `openspec-onboard`. Slash command `/opsx:onboard` stays unchanged. 100% orbit-authored body — no upstream-derived content remains.

**Why α over β (`openspec-onboard-orbit`) or γ (`orbit-onboard`)**:
- **β** introduces an awkward suffix that has no semantic purpose under pegging.
- **γ** preempts a possible future `openspec-*` → `orbit-*` rename for orbit-authored skills. But if such a rename happens, it should happen *all at once* across all orbit-authored skills (`openspec-review`, `openspec-audit-drift`, etc.), not piecemeal here.
- **α** matches the pattern we already use for `openspec-explore`, `openspec-propose`, `openspec-archive-change` (upstream skill names with modified content). We just go further: 100% orbit body, no upstream-bundled portion.

Pegging makes the "what if `openspec update` brings back upstream's onboard?" worry moot — same dynamic as how `openspec-explore` already gets overwritten by orbit's version. Under pegging, the overwrite is *intended*.

### D-onboard-1: Reference-leaning hybrid walkthrough style

**Decision**: The orbit-authored `openspec-onboard` SKILL.md body has 5 sections:

1. **Setup verification** — quick check orbit is installed (overlay applied) and `openspec --version` matches pinned 1.3.1
2. **Identity statement** — what orbit IS, post-D-arch-3 framing (workflow tool using pinned openspec CLI engine, with orbit-authored review/audit/capture layers)
3. **Canonical-flow diagram + 1-paragraph per phase** — explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive
4. **Quick-reference table** of `/opsx:*` commands with one-liner descriptions
5. **Try-it nudge** — "to actually use orbit, run `/opsx:explore` on a real idea you have"

No demo change creation. No automated multi-phase orchestration.

**Why this style**:
- **Primary audience**: AI sessions loading orbit context cold (most leveraged), future-self refresh after break, eventual collaborators
- AI sessions don't benefit from interactive demos — they benefit from parseable workflow docs
- Humans get a quick refresher + try-it entry hook without commitment to a sandbox change
- Effort scale: ~2-3 days writing vs ~1-2 weeks for interactive guided tour

**Alternative considered**: interactive guided tour (creates real demo change end-to-end). Rejected as over-built for current audience. Filed as potential future enhancement if reference-leaning proves insufficient.

**Quality bar note**: user mentioned this may be passed to other people. Reference-leaning is acceptable for "other people" reading it cold — but the content quality must be high (clear examples, accurate command syntax, well-explained rationale).

### D-onboard-2: Lenses introduced contextually (explore + review phases)

**Decision**: Introduce `perspectives.md` and `critical-paths.md` lenses:
- **In the explore-phase walk-through section**: where lenses are captured (per `openspec-explore`'s capture triggers)
- **In the review-phase walk-through section**: brief note that `/opsx:review --as system` consults them (Passes 4–5)

Two touches, both contextual to where the user encounters lenses in the canonical flow.

**Alternative considered**: opt-in epilogue section. Rejected — feels disconnected from the flow; readers may miss it.

### D-onboard-3: External-review loop demoed abstractly

**Decision**: The walkthrough mentions `/opsx:review-external` in the review-phase section with a one-paragraph what-it-does ("packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`") and points to `openspec-review-external/SKILL.md` for details.

No bundled sample prompt artifact. No simulation of the external loop.

**Why abstract**:
- Reference-leaning style means we trust readers to follow the link for depth
- Bundling a sample prompt file as an artifact creates maintenance burden — the underlying SKILL.md evolves and the sample would drift
- The full-loop-simulation alternative is clearly over-built for a reference doc

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

4. ~~**MODIFY** `Internal-run JSON summary format`~~ + ~~**MODIFY** (in `orbit-run-summary-emit` capability delta) `Emit scope`~~ — **REMOVED**: both originally proposed to rewrite parenthetical text in baseline specs that prematurely claimed `/opsx:sync-specs` would be "deprecated/removed by openspec-orbit#6". Under [[orbit-supports-full-openspec-1-3-1]], orbit doesn't author requirements about upstream behavior orbit inherits implicitly. The baseline stale text persists as documentation drift to be addressed in a future doc-hygiene change — NOT load-bearing for this change's cluster-2 goals (pegging declaration + onboard skill).

**Overlay install model implication** (added in address-reviews iter-2 per EC1; refined per iter-4 EW1-reversal):

6. The overlay install mechanism (`cp -r`) does NOT delete files in the user's target project that don't exist in orbit's source. So orbit deleting `.claude/skills/feedback/` from its repo only ensures that file won't be *added* by overlay; it doesn't actively remove pre-existing copies. README install + update + uninstall sections SHALL document an explicit `rm -rf .claude/skills/feedback` command users run after the overlay copy to keep user-project state consistent with overlay intent. Future install-script work (#26) will automate this; until then, doc-only enforcement.

**Address-reviews iter-4 EW1-reversal** (significant correction):

7. The EW1 decision in address-reviews iter-2 to delete `.claude/commands/opsx/sync.md` was REVERSED in iter-4. Original framing claimed sync was "deprecated upstream" — propagated from issue #6's body. **Pushback verification against upstream docs (https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md) at iter-4 showed `/opsx:sync` is listed as "Optional command" — NOT deprecated.** Under the project memory `orbit-supports-full-openspec-1-3-1` ("default scope: fully and cleanly support the openspec@1.3.1 functionality set unless there's a real reason not to"), there is no real reason to drop sync. Reverted EW1's sync-pruning actions: kept `/opsx:sync` command file; restored sync.md to orbit-modified category in `Overlay file disposition`; simplified spec language in `Internal-run JSON summary format` + `Emit scope` to drop "primitive only" + "deprecation" framing. Lesson: pushback discipline must verify claims against current upstream state before propagating; prior "deprecated" claims that pre-date current upstream docs cannot be trusted.

**Address-reviews iter-3 + iter-4 TR-finding resolutions**:

8. **TR1** (task 3.4 sandbox definition): "fresh sandbox" defined inline in task 3.4 — clean temp directory (e.g., `mktemp -d`) + same Node version as project's package.json engines field (or current LTS) + `openspec init --tools claude` from clean state + documented overlay copy + documented prune step + `openspec --version` confirming match. Resolves FF4's "if possible" escape hatch by giving the implementing AI concrete steps.

9. **TR2** (archive→sync-specs dependency anchoring): ACCEPTED AS-IS per user "we don't need to write requirements around it". The dependency lives in `.claude/skills/openspec-archive-change/SKILL.md:65` prose; under project memory `orbit-supports-full-openspec-1-3-1`, orbit's continued use of the upstream primitive is implicit and doesn't warrant a normative spec requirement. If Option 2 work (#27) later rewrites archive flow, the implementing change is expected to read the existing SKILL.md to understand the invocation pattern.

10. **TR3** (subjective Identity statement framing): added negative-test scenario to orbit-onboard `Identity statement section` requirement — Identity section text MUST NOT contain augmentation language ("augments cleanly", "overlay on top of", "layered on top of", etc.). Negative test more useful than positive prescription.

11. **TR4** (Try-it nudge assumes real idea): orbit-onboard `Try-it nudge section` requirement split into 2 closing-recommendation scenarios — named-mode for concrete ideas, bare-mode for orientation-only users (preserves no-demo-change discipline while surfacing bare-mode explore as the no-decision-yet entry point per orbit-explore conventions).

## Risks / Trade-offs

- **Pegging means no upstream improvements** — orbit users freeze at 1.3.1 until a deliberate upgrade-and-port change. Acceptable trade-off per user's explicit framing ("more important to develop correct behavior than ingest new versions").

- **Narrative reframe is forward-looking but Option 2 not yet defined** — D-arch-3 commits to language that anticipates Option 2's cleanup, but Option 2's exact shape isn't specced yet. Risk: the narrative could mention concepts (e.g., "orbit-owned skill set") that don't match Option 2's eventual decisions. Mitigation: keep the reframe high-level enough to survive any reasonable Option 2 shape.

- **Quality bar for orbit-onboard** — user noted it may be passed to other people. Reference-leaning style is acceptable for that audience but requires high content quality (clear examples, well-explained rationale). Review-time scrutiny on the SKILL.md body is warranted.

- **Doc-only pin enforcement is honor-system** — a user running a different upstream version won't be warned. Acceptable for now (small user base); install-script in #26 will close this.

- **Per-skill prune judgment** — `feedback` is the one skill this change prunes, on the criterion "no orbit-mission value". Future prune decisions for upstream skills require similar concrete justification per [[orbit-supports-full-openspec-1-3-1]]. Mitigated by the new `Overlay file disposition` requirement codifying the categories.

- **Interactive guided tour deferred** — if collaborators actually need hands-on guidance to learn orbit, reference-leaning may prove insufficient. Mitigation: the natural extension point ("try-it nudge") could later become an interactive prompt without rewriting the rest of the skill.
