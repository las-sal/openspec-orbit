<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):   Write the new orbit-authored SKILL.md body (5 sections + frontmatter)
                       The substantive content work. Largest chunk.
  Chunk 2 (group 2):   Mirror SKILL.md body to .claude/commands/opsx/onboard.md (per D-onboard-5
                       duplicate-pattern discipline)
  Chunk 3 (group 3):   Validation + user-validation handoff

Total: 3 implementation chunks.

The 3 baseline-drift items per D-conventions-1 are addressed entirely in the spec delta
at specs/orbit-conventions/spec.md (already written during propose). Sync at archive time
applies them to baseline — no per-task work needed at apply time.
-->

## 1. Write the new orbit-authored SKILL.md body

- [ ] 1.1 Delete the existing upstream-bodied content from `.claude/skills/openspec-onboard/SKILL.md` (562 lines including the trailing `# Orbit additions` section). Preserve only the frontmatter block at the top of the file (lines 1-9 area: `name`, `description`, `license`, `compatibility`, `metadata`). The frontmatter's `description` field SHOULD update to reflect the new orbit-authored content (away from "guided onboarding" pedagogical-tour framing; toward "orbit workflow reference walkthrough" framing). The `metadata.author` field MAY change from `openspec` to `orbit` (or similar) to reflect the new ownership; verify the change does not break any orbit/upstream tooling that reads this field.

- [ ] 1.2 Write Section 1 (Setup verification) per orbit-onboard spec `Setup verification section`. Implementation:
  - Checks (in order): (a) `openspec --version` matches the pinned upstream version per `orbit-conventions` `Upstream version pinning`; (b) presence of stable orbit-authored skill artifacts at minimum `.claude/skills/openspec-review/SKILL.md`, `.claude/skills/openspec-audit-drift/SKILL.md`, `.claude/skills/openspec-address-reviews/SKILL.md`, `.claude/skills/openspec-review-external/SKILL.md`, `.claude/skills/openspec-onboard/SKILL.md`; (c) presence of `.claude/skills/openspec-sync-specs/` (upstream-required primitive); (d) count of `^# Orbit additions` matches across `.claude/skills/openspec-*/SKILL.md` (should equal 9 post-this-change — the 10 minus onboard which is now orbit-authored); (e) absence of `.claude/skills/feedback/`.
  - Hard-stop emits lumped "overlay incomplete; see README install section #2" with the exact README path (`README.md` `## Installation` → `### 2. Overlay orbit`).
  - Warn-and-continue emits `⚠` for the feedback/ case with `rm -rf .claude/skills/feedback` remediation.
  - Pass emits the 4-category layered ✓ output.
  - Include a brief gotcha-line about stale AI client cache (verification cannot detect; restart client after install).

- [ ] 1.3 Write Section 2 (Identity statement) per orbit-onboard spec `Identity statement section` + `Identity section MUST NOT contain augmentation language` requirement. Implementation:
  - Lead with: "orbit is a workflow tool that owns the `.claude/` surface (skills, commands, supporting docs) and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine."
  - Enumerate orbit-distinctive layers: editorial review (`/opsx:review` 9-pass proposal mode + 7-pass system mode; `/opsx:review-external` for cross-AI second-opinion; `/opsx:address-reviews` for resolution); drift audit (`/opsx:audit-drift` for 4-category drift scan); capture (`openspec/lenses/perspectives.md` for named callers; `openspec/lenses/critical-paths.md` for flows); JSON run-summary emission (universal-spine schema across all editorial + lifecycle + workflow commands); three execution disciplines (read-before-reference / change completeness / pushback) codified in `orbit-conventions`.
  - Avoid augmentation language per the negative-test scenario — DO NOT say `augments cleanly`, `overlay on top of`, `layered on top of`, `extends upstream`, or similar.

- [ ] 1.4 Write Section 3 (Canonical-flow walkthrough) per orbit-onboard spec `Canonical-flow walkthrough section`. Implementation:
  - ASCII diagram showing the 9 phases: explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive. Diagram width should fit standard terminal (80 cols comfortable).
  - One paragraph per phase. Each paragraph: purpose + primary `/opsx:*` command + typical output. Aim for 80-120 words per paragraph — substantive but not encyclopedic.
  - In the explore-phase paragraph: introduce lenses (`perspectives.md`, `critical-paths.md`) — what they capture, when they're grown (via `/opsx:explore` capture triggers), where they live (`openspec/lenses/`).
  - In the review-phase paragraphs (proposal + system): re-reference lenses — `/opsx:review --as system` Passes 4-5 consult them; passes graceful-skip when lenses are absent.
  - In the review-phase or address-reviews-phase paragraph: name `/opsx:review-external` with one-paragraph what-it-does (packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`); point readers to `.claude/skills/openspec-review-external/SKILL.md` for details. NO bundled sample prompt artifact; NO simulated cross-AI exchange.
  - In the archive-phase paragraph: mention the `sync-specs` subagent invocation (orbit's archive flow uses `openspec-sync-specs` as a callable primitive) and the per-change `.orbit-runs/` audit trail.

- [ ] 1.5 Write Section 4 (Quick-reference command table) per orbit-onboard spec `Quick-reference command table section`. Implementation:
  - Markdown table with columns: `Command` | `One-line description`
  - Enumerate every `/opsx:*` command currently in `.claude/commands/opsx/` (per the documented post-install state: `address-reviews`, `apply`, `archive`, `audit-drift`, `bulk-archive`, `continue`, `explore`, `fast-forward`, `new`, `onboard`, `propose`, `review`, `review-external`, `sync`, `verify` — 15 commands total)
  - One-line descriptions: tight, action-oriented (e.g., "Editorial review of a change — proposal mode (9 passes) or system mode (7 passes via verify-change + 6 system-wide)").
  - Do NOT include an origin column distinguishing orbit-authored vs orbit-modified vs upstream — that classification is canonical in `orbit-conventions` `Overlay file disposition`. MAY include a one-line pointer to that requirement for readers wanting categorization.

- [ ] 1.6 Write Section 5 (Try-it nudge) per orbit-onboard spec `Try-it nudge section`. Implementation:
  - Two closing recommendations, clearly distinguished:
    - **Named-mode** ("If you have a concrete project idea: run `/opsx:explore <name>`"). Brief example invocation. Avoid "for practice" / "try a demo" / "as an exercise" framing.
    - **Bare-mode** ("If you're orienting and don't have a concrete idea yet: run `/opsx:explore` (no name)"). One sentence describing bare-mode as thinking-mode-without-commitment.
  - Close with a single-sentence pointer to other orbit commands (e.g., "When you have findings to address, see `/opsx:address-reviews`; when you want a project-wide drift check, see `/opsx:audit-drift`.").

- [ ] 1.7 Add the non-emission metadata note per orbit-onboard spec `Non-emission of run-summary JSON` requirement. Implementation:
  - Brief 1-2 line note (placement: SKILL body footer OR near the frontmatter — pick whichever reads more naturally). Content: "`/opsx:onboard` does NOT emit a run-summary JSON when invoked. This is a teaching session, not a workflow advancement; composes with `orbit-run-summary-emit` `Emit scope`."

- [ ] 1.8 Sanity sweep on the SKILL body after sections 1.1-1.7 are complete:
  - `grep -E 'augments cleanly|overlay on top of|layered on top of|extends upstream' .claude/skills/openspec-onboard/SKILL.md` — should return 0 matches (per Identity section's negative-test scenario)
  - `grep -c "^## " .claude/skills/openspec-onboard/SKILL.md` — should return 5 (one per section, plus any frontmatter-adjacent context; verify visually)
  - Confirm overall body length is reasonable (target ~500-700 lines including all 5 sections; up from upstream's 562 was 100% upstream content).

## 2. Mirror SKILL.md body to .claude/commands/opsx/onboard.md

- [ ] 2.1 Delete existing upstream-bodied content in `.claude/commands/opsx/onboard.md` (548 lines of upstream guided-tour + 8-phase pedagogy). Preserve frontmatter block at the top (lines 1-5 area: `name`, `description`, `category`, `tags`).

- [ ] 2.2 Copy the body content (Sections 1-5 + non-emission note) from `.claude/skills/openspec-onboard/SKILL.md` into `.claude/commands/opsx/onboard.md`. Maintain the frontmatter divergence: SKILL frontmatter has `license` + `compatibility` + `metadata`; command frontmatter has `category` + `tags`. Bodies are otherwise byte-identical (no prose drift).

- [ ] 2.3 Confirm via diff that the bodies match. `diff <(sed '/^---$/,/^---$/d' .claude/skills/openspec-onboard/SKILL.md) <(sed '/^---$/,/^---$/d' .claude/commands/opsx/onboard.md)` — should return empty (or show only frontmatter-stripping artifacts). This becomes the duplicate-pattern verification per spec `Command-file body matches SKILL.md body (duplicate-pattern discipline)`.

- [ ] 2.4 (Per baseline `orbit-conventions` `Install documentation describes actual install surface` requirement, `Upgrade and overlay-change proposals include README sync` scenario — triggered because this change moves `openspec-onboard` between Overlay file disposition categories) Update README install section to reflect the new overlay disposition. Specific edits:
  - **README.md L956** (Step 2 Overlay orbit, "Overwrites 10 upstream skill files" bullet) — update to "Overwrites 9 upstream skill files" + remove `openspec-onboard` from the enumerated list (the 9 retained: `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`). Update the verification hint to "should return 9 files" (was 10).
  - **README.md L957** (Step 2 Overlay orbit, "Overwrites 9 of the matching slash command bodies" bullet) — update to "Overwrites 8 of the matching slash command bodies" + remove `onboard.md` from the enumerated list (the 8 retained: `apply.md`, `archive.md`, `bulk-archive.md`, `continue.md`, `explore.md`, `new.md`, `propose.md`, `verify.md`).
  - **README.md L955** (Step 2 Overlay orbit, "Adds 4 orbit-authored skills + 4 orbit-authored commands") — update to "Adds 5 orbit-authored skills + 5 orbit-authored commands" with `openspec-onboard` added to the skill list + `onboard.md` added to the command list.
  - **README.md L972** ("What you should see after install" table — `.claude/skills/openspec-*/` row) — update "10 upstream (modified by orbit's overlay) + 4 orbit-authored" to "9 upstream (modified by orbit's overlay) + 5 orbit-authored" with `openspec-onboard` added to the orbit-authored enumeration.
  - **README.md L973** ("What you should see after install" table — `.claude/commands/opsx/` row) — update "9 upstream commands modified by orbit ... + 4 orbit-authored" to "8 upstream commands modified by orbit ... + 5 orbit-authored" with `onboard.md` added to the orbit-authored enumeration. Also update the "Modified upstream skills" row's 10-skill enumeration list to remove `openspec-onboard`.
  - **README.md (Prerequisites bullet around L907)** — if it says "orbit modifies 10 upstream skills", update to "orbit modifies 9 upstream skills".
  - **README.md (anywhere else)** — sweep for any other `10 upstream-modified` / `10 orbit-modified` / `4 orbit-authored` claims; replace per the new disposition.
  - **No fresh-sandbox verify** required for this update — the post-state counts (9 upstream-modified + 5 orbit-authored) are derivable from the `Overlay file disposition` baseline scenarios this change MODIFIES; no new CLI behavior is introduced.
  - After edits, re-grep README: `grep -nE '10 (upstream|orbit)-modified|10 modified upstream|4 orbit-authored' README.md` should return 0 hits (or only intentional historical references — verify each match's context).

## 3. Validation + user-validation handoff

- [ ] 3.1 Run `openspec validate orbit-onboard-follow-up --strict`; resolve any validation findings before user-validation handoff.

- [ ] 3.2 (User-validation) User reads `.claude/skills/openspec-onboard/SKILL.md` cold (ideally in a fresh AI session that doesn't have prior context) and confirms:
  - (a) Section 1 (Setup verification) is clear about what's checked and what happens on hard-stop vs warn vs pass
  - (b) Section 2 (Identity statement) reads cleanly without augmentation language; orbit's distinctive layers are concrete and discoverable
  - (c) Section 3 (Canonical-flow walkthrough) — 9-phase diagram is parseable; each phase paragraph is substantive enough to orient a cold-context reader; lenses introduced contextually; external-review demoed abstractly per design
  - (d) Section 4 (Quick-reference table) enumerates current commands accurately
  - (e) Section 5 (Try-it nudge) covers both named-mode + bare-mode audiences without "for practice" framing
  - (f) Reads cleanly as one continuous narrative with section transitions; no abrupt tone shifts

- [ ] 3.3 (Optional inline verify) After 3.2 passes, optionally invoke `/opsx:onboard` in a fresh AI session against a properly-installed orbit project to confirm Section 1's verification logic emits the expected layered ✓ output (a real end-to-end smoke test). This is OPTIONAL because the verification logic is text-based (the SKILL describes what to check; the AI executes via Bash); spec scenarios + user-validation read are the primary correctness gates.

- [ ] 3.4 If user-validation surfaces no blocking findings, the change is ready for `/opsx:review --as system` (post-apply review), then `/opsx:archive`.
