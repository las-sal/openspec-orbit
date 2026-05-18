# External Review: bootstrap-openspec-orbit (iteration 2)

**Reviewer**: Claude Opus 4.7
**Date**: 2026-05-17

## CRITICAL

None.

## WARNING

### design.md still claims the exploration record lives under the pre-propose `openspec/explore/` path
**File**: openspec/changes/bootstrap-openspec-orbit/design.md:11
**Description**: design.md says "The full exploration record that produced this change lives at `openspec/explore/bootstrap-openspec-orbit/explore.md` and `openspec/explore/bootstrap-openspec-orbit/sketches/`." Per this change's own rule (D8 just below at line 114–118), `/opsx:propose` MOVES the staging dir to `openspec/changes/<name>/`, and the dogfood demonstrates that: the actual file lives at `openspec/changes/bootstrap-openspec-orbit/explore.md`. This is exactly the same class of finding GPT-5 Codex (iteration 1) caught in README.md:55 — the README was fixed, but the same broken path survives in design.md. Update both paths on line 11 to `openspec/changes/bootstrap-openspec-orbit/...`, optionally noting that `openspec/explore/<name>/` is the *pre-propose* staging location.

### `sketches/review-external.md` composition diagram still reflects the obsolete chat-paste handoff model
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/review-external.md:305
**Description**: The composition diagram (lines 295–324) flows `/opsx:review-external → prompt in chat → user copies → pastes into codex`. That is the v1-superseded "chat-paste" contract Codex iteration 1 already flagged (and which was reconciled in the prose body of this sketch, in `design.md` D12, and in the spec). The composition diagram at the foot of the sketch was missed during that fix and now contradicts the rest of the file (lines 44–69 correctly describe "prompt file in `.orbit-runs/` + short invocation snippet to chat"). An implementer reading the visual flow at the bottom of the sketch will internalize the wrong handoff surface. Update lines 303–308 so the diagram reads something like "prompt file written to `.orbit-runs/external-prompt-...md`" → "tiny invocation snippet to chat" → "user pushes + pastes snippet into codex" → "codex pulls repo + reads prompt file".

### `sketches/review-proposal.md` composition diagram still shows `--from-paste` as the external-feedback path
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/review-proposal.md:215
**Description**: In the composition diagram, the `/opsx:address-reviews` node lists "consumes external-feedback pastes (--from-paste)" alongside marker consumption. Per `design.md` D6 and the address-reviews spec, `--from-paste` is explicitly deferred to v2 and `--from-file` is the v1 external-feedback ingest path. The same diagram was the focus of Codex iteration 1's third finding (which was fixed in `sketches/address-reviews.md`); the parallel fix in this sketch's composition diagram was missed. The same regression is present at line 184 of this file (Open Question #2: "Pastes route through address-reviews with `--from-paste`, not through review-proposal."). Update both call-outs to reference `--from-file` for v1, leaving `--from-paste` only in v2 deferral notes.

### `sketches/review-system.md` composition diagram has the same stale `--from-paste` reference
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/review-system.md:232
**Description**: Mirror of the previous finding in the post-apply sketch: the composition diagram lists "consumes external-feedback pastes (--from-paste)" under address-reviews. v1 ingests external findings via `--from-file`; `--from-paste` is v2. Replace `--from-paste` with `--from-file` in this diagram. (This is the same blind spot the previous two findings flag — a fix that touched the primary `address-reviews` sketch but didn't grep the other sketches' composition diagrams for the same stale token.)

### Spec Requirement "Nine review passes executed" does not define what Pass 8 actually does
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-proposal/spec.md:19
**Description**: The requirement enumerates the nine passes by name only; only Pass 6 (Gap Hunt) gets a "operational heuristic" scenario describing the work. Pass 8 (Inline Review Marker Residue) is named but has no scenario specifying (a) what file scope it greps (the change dir under `openspec/changes/<name>/`), (b) the carve-out for the orbit-conventions and orbit-address-reviews specs that document the `@review:` syntax as content (the sketch's prompt at `sketches/review-external.md:218–221` and this very review-external prompt's pass list have the carve-out, but no spec scenario records it), and (c) what severity Pass 8 findings get (sketch line 100 says CRITICAL, but spec doesn't say). Without those, an implementer who only reads the spec will either skip the carve-out and report CRITICAL findings on the documentation specs themselves, or invent the carve-out scope arbitrarily. Add a `#### Scenario: Pass 8 marker-residue scope and exclusions` with explicit file scope, exclusion rule, and severity.

### Spec doesn't define what counts as "Pass 8 marker residue" vs. the `@review:` documentation appearances throughout the change
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-proposal/spec.md:19
**Description**: Related to the above but distinct: the change's own design.md (lines 19, 82, 87–89), tasks.md (multiple lines), proposal.md (lines 5, 15, 19, 35), and several specs contain literal `@review:` strings as part of describing the convention. A Pass 8 implementation that greps `@review:` over the change dir without per-file context will produce a flood of false positives. The sketch carve-out only mentions the conventions / address-reviews specs, but the convention is also documented in proposal.md, design.md (D5), tasks.md item 10.1, and (transitively) in this very review prompt file. The cleanest spec-side answer is to define "marker residue" as `@review:` appearances NOT inside a fenced code block, NOT in a markdown-table cell quoting marker syntax, and NOT in a section explicitly describing the convention. Either tighten the scenario language, or accept that Pass 8 needs an AI-judged distinction between "documentation of the marker convention" and "an actual unresolved review marker" and say so explicitly.

### explore.md Decisions section still describes `/opsx:audit-residue` as Pass 6 / a sibling command in two normative-looking bullets
**File**: openspec/changes/bootstrap-openspec-orbit/explore.md:93
**Description**: Two bullets under the "Review-command design" Decisions section still reference `/opsx:audit-residue` rather than `/opsx:audit-drift`: line 93 ("Drift / residue check — call into `/opsx:audit-residue`...") and line 95 ("This shared convention applies to `/opsx:review-proposal`, `/opsx:review-system`, `/opsx:audit-residue`, and `/opsx:distill-specs`."). A later Decisions entry on line 149 explicitly supersedes `audit-residue` with `audit-drift` ("Unified audit command: `/opsx:audit-drift` (2026-05-17, supersedes `audit-residue` — one command, not many)"), but the two earlier bullets were not updated when the rename landed. They sit inside the Decisions section, dated 2026-05-17, with no rejection or supersession marker on them. Codex's iteration-1 review noted in Notes that historical rejected-context references to old vocabulary were left alone — these two cases are not rejected context, they're current-looking Decision bullets. Either rewrite them to `audit-drift`, or tag them with `(superseded same day by audit-drift)` so a fresh reader can tell why they read as out-of-date. Also at line 192 in the References section a bullet still references `/opsx:audit-residue`.

## SUGGESTION

### Mode-C crystallization threshold drifts between artifacts: `2 or more` vs `2+` vs `~2-3` vs `~2`
**File**: openspec/changes/bootstrap-openspec-orbit/sketches/explore.md:48
**Description**: The normative spec scenario (`specs/orbit-explore-modifications/spec.md:38`) defines the threshold as "2 or more substantive decisions". The same sketch says `~2-3` at line 48 of its mode diagram and `2+` at line 186 of its heuristics. The README has both `~2 substantive decisions` (line 203) and `2+ substantive decisions` (line 212). Pick one phrasing (the spec's "2 or more" / "2+") and use it consistently across spec, sketch, and README. The `~2-3` phrasing in the mode diagram is the most likely to mislead an implementer because it's at the most-visible-quick-skim location in the sketch.

### explore.md Decisions section dated 2026-05-17 still uses pre-rename verb in two bullets that aren't tagged superseded
**File**: openspec/changes/bootstrap-openspec-orbit/explore.md:95
**Description**: Same shape as the earlier warning about `audit-residue`. Listed separately as a suggestion because the entry mentioning `audit-residue` in the "All orbit review/audit commands inherit `verify-change`'s reporting convention" decision (line 95) lists it alongside `distill-specs`, which is also still v2-deferred. A reader can guess the intent, but a fresh implementer auditing the explore record for "what does orbit do today" will see `audit-residue` listed as if it were a current command. Replace with `audit-drift` (already done in the explicitly-superseding bullet at line 149) and consider whether `distill-specs` should be marked v2 in this bullet for symmetry.

### Repo URL in `proposal.md` does not match the actual checkout / dogfood path used throughout this review
**File**: openspec/changes/bootstrap-openspec-orbit/proposal.md:49
**Description**: proposal.md line 49 says "Repo: `github.com/las-sal/openspec-orbit` (currently private during v1 development)". The README is consistent. Just flagging: design.md doesn't mention the URL, and the local checkout path is `openspec-review/` (renamed-but-not-renamed). This is harmless when read by someone who knows the rename story, but if the dogfood checkout is intentionally still named `openspec-review`, a one-line note in `design.md` Context or under D1 saying "local sandbox path may still read `openspec-review` until the repo is renamed; intentional during v1" would close the cognitive gap. (This is the only place where the originally-rejected name might confuse a fresh reader.)

### Task 6.10 description doesn't make the v1 status of `--from-file` explicit
**File**: openspec/changes/bootstrap-openspec-orbit/tasks.md:67
**Description**: Task 6.10 reads "Implement `--from-file <path>` parsing of external-review markdown into virtual markers". design.md D6 explicitly notes the v2-to-v1 promotion ("revising earlier v2 deferral"), but the task list reads as routine work. Given codex iteration 1's experience that `--from-file` v1-status was mis-recorded in the sketches, a `[R2 in orbit-address-reviews — v1, promoted from v2 deferral per D6]` annotation on this task would lower the risk of an implementer accidentally re-deferring it. Optional.

### orbit-review-system spec roll-up text references "all three dimensions per verify-change's own mapping" without pinning the mapping
**File**: openspec/changes/bootstrap-openspec-orbit/specs/orbit-review-system/spec.md:126
**Description**: The Roll-up Mapping scenario says "Pass 0 contributes to all three dimensions per verify-change's own mapping". This delegates to upstream `verify-change`'s mapping without specifying what happens if upstream changes that mapping after orbit ships. Codegen would naturally need a fixed mapping at the time of implementation. Either pin the mapping in the spec ("Pass 0 contributes to Completeness, Correctness, and Coherence following verify-change v1.3.1's mapping") or accept the dependency explicitly ("If upstream `verify-change` changes its mapping, this spec inherits the change; no orbit-side delta needed."). Low priority; matters only if a future upstream verify-change shifts its dimensions.

## Notes

`openspec validate bootstrap-openspec-orbit` passes (confirmed via direct invocation).

**Comparison with iteration-1 (GPT-5 Codex)**: codex caught seven items and all were resolved cleanly EXCEPT the address-reviews sketch class of fix, which was applied to the address-reviews sketch but missed the parallel composition diagrams in `sketches/review-proposal.md:215`, `sketches/review-system.md:232`, and `sketches/review-external.md:305-308`. Codex didn't see those because it found the canonical occurrence and the prior fix narrowed scope rather than greping the whole `sketches/` tree for the same stale tokens. That pattern — "fix the loudest place; sibling sketches drift" — is the dominant flavor of this iteration's findings, and it's the exact "renamed in some places but not others" blind spot the handoff prompt warned about.

I also found two new classes codex didn't flag: (1) the `design.md:11` stale-path mirror of codex's README.md:55 finding — codex's fix scoped to README; (2) Pass 8's under-specification, which both internal and external prior passes missed because the documented carve-out lives only in sketches/prompts, not in any spec scenario. The Pass 8 issue is the most likely to cause real implementation churn (false-positive findings on the change's own meta-documentation of the marker convention) and could plausibly be elevated to WARNING from a stricter-than-mine reviewer; I left it at WARNING because the orbit-review-proposal spec at least names Pass 8 and points implementers to the orbit-address-reviews / orbit-conventions specs for the marker shape.

The explore.md inconsistencies on `audit-residue` are interior to a long historical record. They didn't merit a CRITICAL, but they're the kind of inconsistency that, in a renames-heavy exploration, accumulates and erodes trust in the record. Worth scrubbing once for completeness.

Nothing I observed contradicted codex's prior findings in retrospect — codex's calls were sound and the resolutions on the surfaces it pointed at look complete. The leakage was structural: same-shape problems in sibling files the fix-pass didn't visit.

Overall: the change is in good shape for `/opsx:apply`. Specs are coherent, validate passes, capability count and task count match (9 capabilities / 105 tasks / 102 spec requirements counted), graceful-degradation scenarios are present where expected, the cross-AI loop contract is now consistent in spec + design + README + the primary sketch, and the dogfood demonstrates the workflow end-to-end. The remaining drift is in sketches (which an implementer reads as design context, not normative) plus one design.md path slip and one spec gap on Pass 8.
