<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):     Overlay cleanup — delete feedback skill + sweep stale spec language
  Chunk 2 (group 2):     Narrative reframe (CLAUDE.md + README opening, audit "overlay" terminology)

Originally planned chunks 3-5 (pegging declaration in README install, orbit-onboard skill body,
issue closures + validation handoff) were CUT mid-implementation on 2026-05-23 after a fresh-sandbox
install verification surfaced that v1.3.1's actual install surface differs materially from the
assumptions baked into the original spec content. The deferred work (orbit-onboard skill rewrite
+ README install-section corrections + #23 closure) returns as a follow-up change once a
re-explore cycle re-grounds orbit-onboard against accurate v1.3.1 install state.

Total in this change: 2 implementation chunks. The orbit-onboard new-capability delta spec
was also removed at cut time.
-->

## 1. Overlay cleanup — delete feedback skill + sweep stale spec language

- [x] 1.1 Delete `.claude/skills/feedback/` directory entirely (the SKILL.md and any other files in it).
- [x] 1.2 Sanity sweep: `grep -r 'feedback' .claude/ --include="*.md"` after deletion; confirm remaining "feedback" matches are prose-only (the `approval/feedback` wording in onboard files — NOT references to the deleted skill). No orbit-shipped file should reference the deleted `feedback` SKILL.
- [x] 1.3 (Per address-reviews iter-3 FF1 resolution + iter-4 EW1-reversal) Rewrite `.claude/skills/openspec-archive-change/references/archive-summary-schema.md:18` to remove the obsolete `openspec-orbit#6 will deprecate /opsx:sync-specs entirely` framing. Replace with framing that simply describes the field (e.g., `sync_specs reports the result of orbit's archive flow invoking the openspec-sync-specs upstream primitive — per orbit-conventions Distribution model — pegged engine + orbit-owned surface`). Drop "transitional" / "removal" / "follow-up change" framing entirely. Then `grep -rn 'openspec-orbit#6\|slated for removal' .claude/` to confirm no further residue remains.
- [x] 1.4 (Per address-reviews iter-7 FF4-4 resolution, parallel to task 1.3 pattern) Update `openspec/specs/orbit-conventions/spec.md:5` Purpose summary to replace the old `Distribution model` framing "(overlay, not CLI fork)" with the post-rename framing "(pegged engine + orbit-owned surface)". OpenSpec archive-sync only updates Requirements blocks, not Purpose prose, so this hand-edit is required for baseline coherence after RENAMED is applied. Verify the Purpose summary's enumeration of orbit-conventions content matches the post-rename requirement title. (Also added forward-references to the two new ADDED requirements — `upstream version pinning` and `overlay file disposition` — so the Purpose summary fully reflects post-archive baseline.)

## 2. Narrative reframe (CLAUDE.md + README opening)

- [x] 2.1 Rewrite CLAUDE.md opening paragraphs (~lines 1-15 currently) to post-pegging framing per design D-arch-3 / orbit-conventions MODIFIED `Distribution model`. Position orbit as a workflow tool that owns the `.claude/` surface and uses `@fission-ai/openspec` at its pinned version as a CLI engine. Drop or heavily qualify "overlay that augments cleanly" language. Mention the pinned version contextually.
- [x] 2.2 Rewrite README opening (`# openspec-orbit` intro + any "What is openspec-orbit" section) with the same reframe. Tone-match CLAUDE.md but README is user-facing (slightly more welcoming, less terse). (Also added forward-ref to a coming "Pegging strategy" section anchor — that section gets added in chunk 3 per task 3.1, now deferred to follow-up.)
- [x] 2.3 Sweep CLAUDE.md and README for other instances of "overlay" terminology; update each where the new framing better fits. Updated 3 pre-pegging-framed uses (README:225 CLAUDE.md snippet for adopters, README:900 install section opening, README:935 idempotent-overlay note). Preserved technical references (e.g., "orbit's overlay overwrites these files" describing `cp -r` mechanism) and the new "NOT an automatic-update overlay" framing language added in 2.1/2.2. Install-section references to `feedback` skill at L917/L933 preserved (deferred to follow-up change — README install-section rewrite was originally chunks 3+ of this change, now part of the follow-up that re-explores orbit-onboard).

## 3. Issue tracking (carried over from original chunk 5)

- [x] 3.1 (Per address-reviews iter-2 ES1) File 2 orbit GH issues to give durable tracking handles to the deferred Option 2 + global-rename work mentioned in design.md Non-Goals. **DONE during address-reviews iter-2 walk**: filed [#27](https://github.com/las-sal/openspec-orbit/issues/27) (Option 2 — drop `# Orbit additions` pattern) and [#28](https://github.com/las-sal/openspec-orbit/issues/28) (Global rename `openspec-*` → `orbit-*`). design.md Non-Goals updated with issue cross-references.
