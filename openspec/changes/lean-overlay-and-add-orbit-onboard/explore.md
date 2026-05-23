# Explore: lean-overlay-and-add-orbit-onboard

## Premise

**Cluster 2 of v0.1.0**: make orbit's `.claude/` overlay a true delta-only structure (closes #6) and add an orbit-specific onboarding skill that walks users through orbit's extended workflow (closes #23).

**Bundling rationale**: A (#6) removes upstream `openspec-onboard` from the overlay. B (#23) replaces it with an orbit-authored version. Shipping together avoids an awkward intermediate state where users see only upstream's incomplete-for-orbit onboarding.

## Decisions

### D1: Pegging strategy — Option 1 immediate, Option 2 next cycle, 3/4 deferred

Orbit pegs to a specific upstream `@fission-ai/openspec` version (current: v1.3.1). Stops trying to be flexible about ingesting future upstream versions. Orbit owns the `.claude/` overlay surface; upstream supplies the CLI tool as a pinned engine.

**Why**: User's strategic call (2026-05-22). Orbit has already diverged enough from upstream that maintaining ingest-flexibility is paying ongoing complexity cost for a property the user doesn't value. The real engineering project is homeENV, not orbit — orbit is infrastructure. Get the dev framework solid, move on.

**Sequencing**:
- **Now (Option 1)**: declare the pegging strategy in README + CLAUDE.md; pin the upstream version; stop worrying about overlay-overwrite concerns; bundle whatever orbit needs.
- **Next explore-cycle (Option 2)**: drop the `# Orbit additions` pattern as a discipline; orbit's skills become just orbit's skills; CHANGELOG/git history replaces file-level annotation.
- **Deferred (Option 3/4)**: hard-fork or replace the upstream CLI binary. Only when the CLI contract becomes painful (it isn't yet — `openspec status --json` etc. are stable enough).

### D2: #6 closes under the pegging strategy, not its original framing

Issue #6 was filed pre-pegging-decision with the framing "delete 9 unmodified files to preserve upstream-update flow". Under D1, that framing is moot — there's no longer an update flow to preserve. Close #6 with a note that the underlying concern (overlay-overwrite risk) is addressed structurally by pegging.

### D3: Shipping Model A (bundle freely) wins by default

The earlier Q1 (A bundled / B sidecar / C composer) collapses under D1. With pegging, there's no "fresh upstream copy to preserve" — Model A's downside (freezes upstream at orbit-author time) becomes a non-issue because we're explicitly pegging upstream anyway.

### D4: Pin to `@fission-ai/openspec@1.3.1`

This is both the latest npm release and what's been tested against (#6 verified against this version on 2026-05-19). Release cadence is roughly one minor per month-or-two (1.0.0 → 1.3.1 spans 2026-01-26 to 2026-04-21), so pinning here doesn't leave us catastrophically behind.

### D5: Doc-only pin enforcement for this change

README + CLAUDE.md state the required upstream version. Nothing prevents users from running a different one. Matches orbit's current manual-install reality (`cp -r` per README; no formal installer).

Install-script-with-version-check is filed as a separate future enhancement: [#26](https://github.com/las-sal/openspec-orbit/issues/26). Depends on this change landing first (so the script has a pinned version to check against).

### D6: Forward-looking narrative reframe (Option β)

Rewrite the opening of CLAUDE.md and README to position orbit as "a workflow tool with its own surface, using `@fission-ai/openspec@1.3.1` as a pinned CLI engine" — dropping or heavily qualifying the "overlay" framing.

**Why β over α (minimal edit) or γ (defer)**: The "overlay" framing has been the source of confusion (literally caused #6's premise to be wrong). Better to fix the framing once than re-fix it during the Option 2 cycle. Cost is ~half a day of writing; payoff is no re-work later.

**Implementation scope** (for this change, captured here for proposal time):
- CLAUDE.md opening paragraphs: reframe orbit's identity
- README opening + "What is orbit" section: same reframe
- Add a "Pegging strategy" section to both, naming the pinned version + the philosophy
- Audit other references to "overlay" terminology and update where the new framing better fits

### D7: Q-peg-4 — delete `feedback`, keep `openspec-sync-specs`

**`feedback`** (truly unmodified upstream): DELETE from overlay. Description: "Collect and submit user feedback about OpenSpec with context enrichment and anonymization." Sends feedback upstream to Fission-AI. Orbit only references it from the soon-to-be-replaced upstream `openspec-onboard` skill. For pegged orbit users, sending feedback upstream serves orbit's mission near-zero.

**`openspec-sync-specs`** (truly unmodified upstream): KEEP in overlay. It's deprecated from *direct user invocation* in 1.3.1, NOT deprecated as a capability — it's a primitive that upstream's `openspec archive` CLI orchestrates internally. Orbit's archive flow calls it via subagent during the sync step. We're using the primitive the same way upstream uses it. No deviation.

**Composition nuance**: orbit's archive currently invokes sync-specs + `mv` as separate steps rather than calling `openspec archive` CLI which does both. End state is identical; orbit's style is to weave its own pre-archive sweep + archive-run-summary around the pieces. Not a meaningful deviation; no follow-up refactor needed.

### D8: Orbit-onboard skill named `openspec-onboard` (option α)

Replace upstream's `openspec-onboard` skill in-place with 100% orbit-authored content. Same directory, same name, same `/opsx:onboard` slash command. No suffix awkwardness, no transitional rename inconsistency.

**Why α over β/γ**: User's reasoning — if a global `openspec-*` → `orbit-*` rename happens later (potentially as part of Option 2 or its own change), do it all at once rather than partially now. The new skill should honor current sibling convention.

Pegging makes the "what if openspec update brings back upstream's onboard?" worry moot — same dynamic as how `openspec-explore` already gets overwritten by orbit's version.

### D9: Walkthrough style — reference-leaning hybrid

The orbit-onboard skill body provides:

1. **Setup verification** — quick check orbit is installed and `openspec --version` matches the pinned 1.3.1
2. **Identity statement** — what orbit IS, post-D6 framing (workflow tool using pinned openspec CLI engine)
3. **Canonical flow diagram + 1-paragraph per phase** — explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive
4. **Quick-reference table** of `/opsx:*` commands
5. **Try-it nudge** — "to actually use orbit, run /opsx:explore on a real idea"

No demo change creation. No automated multi-phase orchestration. AI sessions get a parseable workflow doc; humans get a quick refresher + entry hook.

**Why this style**: Primary audience is AI sessions (load context fast), future-self (refresh after break), and eventual collaborators. User notes this may be passed to other people, so quality matters — but interactive guided tour is over-built for the audience. Reference-leaning hybrid is the right complexity tier. Effort: ~2-3 days writing.

**Future enhancement option**: an interactive guided tour can be filed as a separate issue if the reference-only approach proves insufficient in practice.

### D10: Lenses introduced contextually during the canonical-flow walkthrough

Introduce `perspectives.md` and `critical-paths.md`:
- **In the explore-phase section**: where lenses are captured (per `openspec-explore`'s capture triggers)
- **In the review-phase section**: briefly note that `/opsx:review --as system` consults them (Passes 4–5)

Two touches, both contextual to where the user encounters them in the flow. Rejected alternative: opt-in epilogue section (feels disconnected).

### D11: External-review loop demoed abstractly

Show the `/opsx:review-external` command, one-paragraph what-it-does ("packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`"), and point to `openspec-review-external/SKILL.md` for details. No simulation, no pre-generated prompt sample.

**Why abstract**: bundling a sample prompt file as an artifact adds maintenance burden — the underlying SKILL.md evolves and the sample would drift. Reference-leaning style means we trust readers to follow the link for depth.

## Open questions

*(All resolved. See Decisions D1–D11 above.)*

### Q5: Orbit-onboard skill — naming

Options on the table:
- `openspec-onboard-orbit` — mirrors `openspec-*` pattern; slightly awkward suffix
- `opsx-onboard` — cleaner; breaks `openspec-*` skill naming convention
- `orbit-onboard` — matches `orbit-*` capability convention; deviates from `openspec-*` skill convention

The skill name affects the `/opsx:*` slash-command surface and how upstream/orbit skills are visually distinguished.

### Q6: Orbit-onboard skill — walkthrough style

Issue #23 didn't specify:
- **Interactive guided tour** — creates a real demo change, walks user through all phases hands-on
- **Reference walkthrough** — reads through the canonical cycle with examples but doesn't create artifacts
- **Hybrid** — narrated tour with optional "try it" prompts

Tradeoffs: interactive is more useful but introduces demo-change cleanup concerns; reference is simpler but more passive.

### Q7: Capture-layer (lenses) introduction point

The orbit walkthrough should introduce `perspectives.md` and `critical-paths.md` somewhere. Natural fit points:
- During explore demo ("here's where you'd capture a perspective")
- After review ("review surfaced X — let's add a lens for it")
- As an opt-in epilogue

### Q8: External-review demo concreteness

The walkthrough should demonstrate the external-review loop. Options:
- Abstract ("at this point you'd run /opsx:review-external")
- Generated-prompt-only (actually generates the prompt file but doesn't push)
- Full loop (generate + push + simulate external AI return)

Latter is most pedagogically valuable but requires deciding on a simulated-return convention.

## Considered & out

(items considered and explicitly scoped out — captured here as we go)

## References

- GH issue [#6](https://github.com/las-sal/openspec-orbit/issues/6) — Overlay ships unmodified upstream files
- GH issue [#23](https://github.com/las-sal/openspec-orbit/issues/23) — /opsx:onboard for orbit workflow
- Just-archived change `2026-05-21-emit-run-summary-jsons-from-workflow-commands` — caused the premise-invalidation finding by adding `# Orbit additions` to 7 of the 9 skills #6 targets
- GH issue [#25](https://github.com/las-sal/openspec-orbit/issues/25) — Task-gate pushback in /opsx:archive (filed during cluster 1 archive flow; not in cluster 2)
