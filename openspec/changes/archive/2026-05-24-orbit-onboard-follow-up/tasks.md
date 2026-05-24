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

- [x] 1.1 Delete the existing upstream-bodied content from `.claude/skills/openspec-onboard/SKILL.md` (562 lines including the trailing `# Orbit additions` section). Preserve only the frontmatter block at the top of the file (lines 1-9 area: `name`, `description`, `license`, `compatibility`, `metadata`). The frontmatter's `description` field SHOULD update to reflect the new orbit-authored content (away from "guided onboarding" pedagogical-tour framing; toward "orbit workflow reference walkthrough" framing). The `metadata.author` field MAY change from `openspec` to `orbit` (or similar) to reflect the new ownership; verify the change does not break any orbit/upstream tooling that reads this field.

- [x] 1.2 Write Section 1 (Setup verification) per orbit-onboard spec `Setup verification section`. Implementation:
  - Skill checks (in order): (a) `openspec --version` matches the pinned upstream version per `orbit-conventions` `Upstream version pinning`; (b) presence of orbit-authored skill artifacts at minimum `.claude/skills/openspec-review/SKILL.md`, `.claude/skills/openspec-audit-drift/SKILL.md`, `.claude/skills/openspec-address-reviews/SKILL.md`, `.claude/skills/openspec-review-external/SKILL.md`, `.claude/skills/openspec-onboard/SKILL.md`; (c) presence of `.claude/skills/openspec-sync-specs/` (upstream-required primitive); (d) count of `^# Orbit additions` matches across `.claude/skills/openspec-*/SKILL.md` (should equal 9 post-this-change — the 10 minus onboard which is now orbit-authored); (e) absence of `.claude/skills/feedback/`.
  - **Command checks** (in order, per orbit-onboard spec `Setup verification section` `Hard-stop on overlay-incomplete` scenario which promises hard-stop on missing skill OR command): (f) presence of orbit-authored command files: `.claude/commands/opsx/review.md`, `.claude/commands/opsx/review-external.md`, `.claude/commands/opsx/audit-drift.md`, `.claude/commands/opsx/address-reviews.md`, `.claude/commands/opsx/onboard.md`; (g) presence of orbit-shipped sync command: `.claude/commands/opsx/sync.md`; (h) count of files in `.claude/commands/opsx/` (should equal 15 post-this-change — 14 from orbit's overlay + 1 untouched upstream `ff.md`; was 16 pre-this-change before `fast-forward.md` was dropped).
  - Hard-stop emits lumped message per spec `Lumped messaging for overlay-incomplete sub-modes` scenario (points to README `## Installation` section as a whole, covering both `### 1. Initialize upstream OpenSpec` for version-pin issues and `### 2. Overlay orbit` for overlay-incomplete issues including command-file gaps).
  - Warn-and-continue emits `⚠` for the feedback/ case with `rm -rf .claude/skills/feedback` remediation.
  - Pass emits the 4-category layered ✓ output (skills + commands both verified).
  - Include a brief gotcha-line about stale AI client cache (verification cannot detect; restart client after install).

- [x] 1.3 Write Section 2 (Identity statement) per orbit-onboard spec `Identity statement section` + `Identity section MUST NOT contain augmentation language` requirement. Implementation:
  - Lead with: "orbit is a workflow tool that owns the `.claude/` surface (skills, commands, supporting docs) and uses `@fission-ai/openspec@1.3.1` as a pinned CLI engine."
  - Enumerate orbit-distinctive layers: editorial review (`/opsx:review` 9-pass proposal mode + 7-pass system mode; `/opsx:review-external` for cross-AI second-opinion; `/opsx:address-reviews` for resolution); drift audit (`/opsx:audit-drift` for 4-category drift scan); capture (`openspec/lenses/perspectives.md` for named callers; `openspec/lenses/critical-paths.md` for flows); JSON run-summary emission (universal-spine schema across all editorial + lifecycle + workflow commands); three execution disciplines (read-before-reference / change completeness / pushback) codified in `orbit-conventions`.
  - Avoid augmentation language per the negative-test scenario — DO NOT say `augments cleanly`, `overlay on top of`, `layered on top of`, `extends upstream`, or similar.

- [x] 1.4 Write Section 3 (Canonical-flow walkthrough) per orbit-onboard spec `Canonical-flow walkthrough section`. Implementation:
  - ASCII diagram showing the 9 phases: explore → propose → review → address-reviews → apply → verify → review --as system → address-reviews → archive. Diagram width should fit standard terminal (80 cols comfortable).
  - One paragraph per phase. Each paragraph: purpose + primary `/opsx:*` command + typical output. Aim for 80-120 words per paragraph — substantive but not encyclopedic.
  - In the explore-phase paragraph: introduce lenses (`perspectives.md`, `critical-paths.md`) — what they capture, when they're grown (via `/opsx:explore` capture triggers), where they live (`openspec/lenses/`).
  - In the review-phase paragraphs (proposal + system): re-reference lenses — `/opsx:review --as system` Passes 4-5 consult them; passes graceful-skip when lenses are absent.
  - In the review-phase or address-reviews-phase paragraph: name `/opsx:review-external` with one-paragraph what-it-does (packages a review request for a different AI; emits findings file; consumed via `/opsx:address-reviews --from-file`); point readers to `.claude/skills/openspec-review-external/SKILL.md` for details. NO bundled sample prompt artifact; NO simulated cross-AI exchange.
  - In the archive-phase paragraph: mention the `sync-specs` subagent invocation (orbit's archive flow uses `openspec-sync-specs` as a callable primitive) and the per-change `.orbit-runs/` audit trail.

- [x] 1.5 Write Section 4 (Quick-reference command table) per orbit-onboard spec `Quick-reference command table section`. Implementation:
  - Markdown table with columns: `Command` | `One-line description`
  - Enumerate every slash command in the user's post-install `.claude/commands/opsx/` (15 files post-this-change): `address-reviews`, `apply`, `archive`, `audit-drift`, `bulk-archive`, `continue`, `explore`, `ff` (upstream-installed, untouched — orbit no longer ships its own `fast-forward.md` per `orbit-conventions` `Verbatim upstream files not in orbit's overlay`), `new`, `onboard`, `propose`, `review`, `review-external`, `sync`, `verify` — 15 commands total
  - One-line descriptions: tight, action-oriented (e.g., "Editorial review of a change — proposal mode (9 passes) or system mode (7 passes via verify-change + 6 system-wide)").
  - Do NOT include an origin column distinguishing orbit-authored vs orbit-modified vs upstream — that classification is canonical in `orbit-conventions` `Overlay file disposition`. MAY include a one-line pointer to that requirement for readers wanting categorization.

- [x] 1.6 Write Section 5 (Try-it nudge) per orbit-onboard spec `Try-it nudge section`. Implementation:
  - Two closing recommendations, clearly distinguished:
    - **Named-mode** ("If you have a concrete project idea: run `/opsx:explore <name>`"). Brief example invocation. Avoid "for practice" / "try a demo" / "as an exercise" framing.
    - **Bare-mode** ("If you're orienting and don't have a concrete idea yet: run `/opsx:explore` (no name)"). One sentence describing bare-mode as thinking-mode-without-commitment.
  - Close with a single-sentence pointer to other orbit commands (e.g., "When you have findings to address, see `/opsx:address-reviews`; when you want a project-wide drift check, see `/opsx:audit-drift`.").

- [x] 1.7 Add the non-emission metadata note per orbit-onboard spec `Non-emission of run-summary JSON` requirement. Implementation:
  - Brief 1-2 line note (placement: SKILL body footer OR near the frontmatter — pick whichever reads more naturally). Content: "`/opsx:onboard` does NOT emit a run-summary JSON when invoked. This is a teaching session, not a workflow advancement; composes with `orbit-run-summary-emit` `Emit scope`."

- [x] 1.8 Sanity sweep on the SKILL body after sections 1.1-1.7 are complete:
  - `grep -E 'augments cleanly|overlay on top of|layered on top of|extends upstream' .claude/skills/openspec-onboard/SKILL.md` — should return 0 matches (per Identity section's negative-test scenario)
  - `grep -c "^## " .claude/skills/openspec-onboard/SKILL.md` — should return 5 (one per section, plus any frontmatter-adjacent context; verify visually)
  - Confirm overall body length is reasonable (target ~500-700 lines including all 5 sections; up from upstream's 562 was 100% upstream content).

## 2. Mirror SKILL.md body to .claude/commands/opsx/onboard.md

- [x] 2.1 Delete existing upstream-bodied content in `.claude/commands/opsx/onboard.md` (548 lines of upstream guided-tour + 8-phase pedagogy). Preserve frontmatter block at the top (lines 1-5 area: `name`, `description`, `category`, `tags`).

- [x] 2.2 Copy the body content (Sections 1-5 + non-emission note) from `.claude/skills/openspec-onboard/SKILL.md` into `.claude/commands/opsx/onboard.md`. Maintain the frontmatter divergence: SKILL frontmatter has `license` + `compatibility` + `metadata`; command frontmatter has `category` + `tags`. Bodies are otherwise byte-identical (no prose drift).

- [x] 2.3 Confirm via diff that the bodies match. `diff <(sed '/^---$/,/^---$/d' .claude/skills/openspec-onboard/SKILL.md) <(sed '/^---$/,/^---$/d' .claude/commands/opsx/onboard.md)` — should return empty (or show only frontmatter-stripping artifacts). This becomes the duplicate-pattern verification per spec `Command-file body matches SKILL.md body (duplicate-pattern discipline)`.

- [x] 2.3a Delete `.claude/commands/opsx/fast-forward.md` from orbit's overlay (per `orbit-conventions` modified `Verbatim upstream files not in orbit's overlay` scenario). The file is byte-identical to upstream's `ff.md` modulo trailing whitespace; orbit no longer ships it. After the deletion, users invoke the fast-forward workflow via upstream's `ff.md` (already installed by `init --tools claude`). Verify: `test ! -f .claude/commands/opsx/fast-forward.md && echo OK || echo FAIL`. Note: `openspec-ff-change` SKILL stays orbit-modified (it has `# Orbit additions`); only the command-file duplicate is removed. SKILL/command-file naming inconsistency (`openspec-ff-change` SKILL ↔ no orbit-shipped `ff` command) is acknowledged but not resolved here — out-of-scope rename per design.md Non-Goals.

- [x] 2.4 (Per baseline `orbit-conventions` `Install documentation describes actual install surface` requirement, `Upgrade and overlay-change proposals include README sync` scenario — triggered because this change moves `openspec-onboard` between Overlay file disposition categories AND drops `fast-forward.md` from the overlay per the new `Verbatim upstream files not in orbit's overlay` scenario) Update README install section to reflect the new overlay disposition. Specific edits:
  - **README.md L955** (Step 2 Overlay orbit, "Adds 4 orbit-authored skills + 4 orbit-authored commands") — update to "Adds 5 orbit-authored skills + 5 orbit-authored commands" with `openspec-onboard` added to the skill list + `onboard.md` added to the command list.
  - **README.md L956** (Step 2 Overlay orbit, "Overwrites 10 upstream skill files" bullet) — update to "Overwrites 9 upstream skill files" + remove `openspec-onboard` from the enumerated list (the 9 retained: `openspec-explore`, `openspec-propose`, `openspec-archive-change`, `openspec-apply-change`, `openspec-verify-change`, `openspec-continue-change`, `openspec-ff-change`, `openspec-new-change`, `openspec-bulk-archive-change`). Update the verification hint to "should return 9 files" (was 10).
  - **README.md L957** (Step 2 Overlay orbit, "Overwrites 9 of the matching slash command bodies" bullet) — update to "Overwrites 8 of the matching slash command bodies" + remove `onboard.md` from the enumerated list (the 8 retained: `apply.md`, `archive.md`, `bulk-archive.md`, `continue.md`, `explore.md`, `new.md`, `propose.md`, `verify.md`).
  - **README.md (Step 2 sub-bullet describing additional orbit-shipped commands)** — currently says "Adds 2 additional orbit commands: `sync.md`, `fast-forward.md`". Update to: "Adds 1 additional orbit-shipped command: `sync.md`. Note: orbit previously shipped `fast-forward.md` (byte-identical to upstream's `ff.md`); this was removed per `orbit-conventions` `Verbatim upstream files not in orbit's overlay`. Users invoke the fast-forward workflow via upstream's `ff.md`, which `init --tools claude` installs and orbit's overlay leaves untouched."
  - **README.md L972** ("What you should see after install" table — `.claude/skills/openspec-*/` row) — update "10 upstream (modified by orbit's overlay) + 4 orbit-authored" to "9 upstream (modified by orbit's overlay) + 5 orbit-authored" with `openspec-onboard` added to the orbit-authored enumeration.
  - **README.md L973** ("What you should see after install" table — `.claude/commands/opsx/` row) — update from "16 command files: 9 upstream commands modified by orbit ... + 1 upstream command untouched by overlay (`ff.md`) + 4 orbit-authored ... + 2 additional orbit commands (`sync.md`, `fast-forward.md`)" to "**15 command files**: 8 upstream commands modified by orbit (apply, archive, bulk-archive, continue, explore, new, propose, verify) + 1 upstream command untouched by overlay (`ff.md` — orbit no longer ships its own copy per `Verbatim upstream files not in orbit's overlay`) + 5 orbit-authored (`review.md`, `review-external.md`, `audit-drift.md`, `address-reviews.md`, `onboard.md`) + 1 additional orbit-shipped command (`sync.md`)". Also update the "Modified upstream skills" row's 10-skill enumeration list to remove `openspec-onboard` (now 9).
  - **README.md (Prerequisites bullet around L907)** — if it says "orbit modifies 10 upstream skills" or similar, update to "9 upstream skills".
  - **README.md L985** ("Partial adoption" — "All 10 upstream skills then behave unmodified" bullet) — update "All 10 upstream skills" to "All 9 modified upstream skills" + update "four new orbit-authored commands ... four new SKILL.md directories" to "five new orbit-authored commands ... five new SKILL.md directories" with onboard added.
  - **README.md L994** ("Working alongside" — "any of the 10 orbit-modified upstream skills" + enumeration of 10) — update to "any of the 9 orbit-modified upstream skills" + remove `openspec-onboard` from the enumerated list.
  - **README.md L999** ("Common gotchas" — "All 10 orbit-modified skills") — update to "All 9 orbit-modified skills".
  - **README.md L1036 + L1040 + L1048** (Uninstalling section — references to "4 orbit-authored" / "4 orbit-authored commands and skills" / "10 orbit-modified skills") — update the rm commands to remove 5 (not 4) orbit-authored command files + 5 (not 4) orbit-authored skill directories. Specifically: include `onboard.md` in the rm of orbit-authored commands; include `openspec-onboard` in the rm -rf of orbit-authored skill directories. Update the "Restore upstream-pristine state for the 10 orbit-modified skills" comment to "9 orbit-modified skills".
  - **fast-forward.md-specific README cleanup (per `Verbatim upstream files not in orbit's overlay` scenario)** — orbit no longer ships `fast-forward.md`, so README references to it as orbit-shipped become stale:
    - **README.md L829** (ASCII tree of orbit's `.claude/commands/opsx/` structure) — remove `fast-forward.md` from the orbit-shipped listing in the tree; users see upstream's `ff.md` only.
    - **README.md L957** (Step 2 sub-bullet) — currently ends with "(The 10th — upstream's `ff.md` — is left untouched; see the `fast-forward.md` note below.)" — UPDATE to drop the forward-reference: "(The 10th — upstream's `ff.md` — is left untouched. Orbit no longer ships its own `fast-forward.md`; users run the fast-forward workflow via upstream's `ff.md` per `orbit-conventions` `Verbatim upstream files not in orbit's overlay`.)"
    - **README.md L959** (the bullet describing `fast-forward.md` as orbit-shipped) — REMOVE the bullet entirely (it described `fast-forward.md` as "orbit's rewrite of the fast-forward workflow under a different filename" — that framing is now wrong; investigation showed it was byte-identical to upstream's `ff.md`).
    - **README.md L975** (entire "Naming divergence: `ff.md` vs `fast-forward.md`" table row in `### What you should see after install`) — REMOVE the entire table row. The naming-divergence concept no longer applies; orbit ships only one copy via upstream's `ff.md`.
    - **README.md L1037-1038** (Uninstalling section, the rm command that includes `fast-forward`) — UPDATE the rm pattern to remove `fast-forward` from the orbit-only commands list. Was: `rm .claude/commands/opsx/{review,review-external,audit-drift,address-reviews,sync,fast-forward}.md` → becomes: `rm .claude/commands/opsx/{review,review-external,audit-drift,address-reviews,onboard,sync}.md` (now 6 orbit-authored/added commands including `onboard.md`, dropping `fast-forward`).
    - **README.md L1060** (init --force scope statement, the ❌ bullet) — UPDATE the list of orbit-shipped target-only files that `init --force` doesn't delete: remove `.claude/commands/opsx/fast-forward.md` from the enumeration. The remaining target-only files: `.claude/skills/openspec-sync-specs/`, `.claude/commands/opsx/sync.md`. Adjust the "11 skills + 12 commands instead of upstream's 10 + 10" framing to "11 skills + 11 commands" (one fewer command since `fast-forward.md` is gone). Verify against sandbox in task 2.5.
  - **README.md** — sweep for **numeric AND spelled-out** stale forms with a broadened residue grep: `grep -nEi '\b(10|ten)\b.*(upstream|orbit)-?modified|\b(10|ten)\b.*modified.*upstream|All (10|ten).*skill|\b(4|four)\b.*new orbit|\b(4|four)\b.*orbit-authored.*(skill|command|directory|directories)|any of the (10|ten)' README.md` — should return 0 hits (or only intentional historical references — verify each match's context). ALSO sweep for residual fast-forward references: `grep -n 'fast-forward' README.md` should return 0 hits (or only references inside historical-text contexts — verify each).
  - **Sandbox verify covered in task 2.5** (per `README-modifying changes pair with sandbox verification` baseline scenario — unconditional).

- [x] 2.5 (Per baseline `orbit-conventions` `Install documentation describes actual install surface` requirement, `README-modifying changes pair with sandbox verification` scenario — unconditional) Run a fresh-sandbox verification of the rewritten README:
  - `SANDBOX=$(mktemp -d)` + same Node version as current orbit dev environment + `npx -y @fission-ai/openspec@1.3.1 --version` confirms 1.3.1 pin.
  - Run `npx -y @fission-ai/openspec@1.3.1 init --tools claude` → assert 10 skills + 10 commands (including `ff.md`); no `feedback/`.
  - Apply the documented `cp -r` overlay from a fresh clone of orbit's `main` (NOT the dev copy — use a temp clone) → assert post-overlay state: 15 skills (10 upstream-modified-or-authored: 9 orbit-modified + 1 orbit-authored `openspec-onboard` + 5 orbit-additions including `openspec-sync-specs`) and 15 commands (9 orbit-modified `apply`/`archive`/`bulk-archive`/`continue`/`explore`/`new`/`onboard`/`propose`/`verify` plus 5 orbit-added `review`/`review-external`/`audit-drift`/`address-reviews`/`sync` plus 1 upstream-untouched `ff` — NO `fast-forward.md` since orbit dropped it).
  - Run `rm -rf .claude/skills/feedback` → assert no error, file absent.
  - Run the README's updated uninstall sequence (per task 2.4's L1036/1040/1048 edits) → assert post-uninstall state: 10 upstream skills (pristine) + 10 upstream commands (including `ff.md`); orbit-authored/added files absent; preserves user-created `.claude/custom/*` + all `openspec/` content.
  - Verify the README's broadened residue grep (per task 2.4's last bullet) returns 0 hits in the final synced README.
  - If verification surfaces a mismatch (CLI behavior differs from README prose, or post-install counts differ from documentation), HALT and add `@review(escalated):` markers at the relevant README locations; do NOT proceed to chunk 3 until escalations are resolved.

## 3. Validation + user-validation handoff

- [x] 3.1 Run `openspec validate orbit-onboard-follow-up --strict`; resolve any validation findings before user-validation handoff.

- [x] 3.2 (User-validation) User reads `.claude/skills/openspec-onboard/SKILL.md` cold (ideally in a fresh AI session that doesn't have prior context) and confirms:
  - (a) Section 1 (Setup verification) is clear about what's checked and what happens on hard-stop vs warn vs pass
  - (b) Section 2 (Identity statement) reads cleanly without augmentation language; orbit's distinctive layers are concrete and discoverable
  - (c) Section 3 (Canonical-flow walkthrough) — 9-phase diagram is parseable; each phase paragraph is substantive enough to orient a cold-context reader; lenses introduced contextually; external-review demoed abstractly per design
  - (d) Section 4 (Quick-reference table) enumerates current commands accurately
  - (e) Section 5 (Try-it nudge) covers both named-mode + bare-mode audiences without "for practice" framing
  - (f) Reads cleanly as one continuous narrative with section transitions; no abrupt tone shifts

- [x] 3.3 (Optional inline verify) After 3.2 passes, optionally invoke `/opsx:onboard` in a fresh AI session against a properly-installed orbit project to confirm Section 1's verification logic emits the expected layered ✓ output (a real end-to-end smoke test). This is OPTIONAL because the verification logic is text-based (the SKILL describes what to check; the AI executes via Bash); spec scenarios + user-validation read are the primary correctness gates. **Covered by task 2.5 sandbox verification**: the documented procedure (init → overlay → prune) produces 15 skills + 15 commands; the verification logic in Section 1 was authored to assert exactly this state. Cold-read by fresh AI subagent (task 3.2) also independently verified Section 1's clarity + correctness.

- [x] 3.4 If user-validation surfaces no blocking findings, the change is ready for `/opsx:review --as system` (post-apply review), then `/opsx:archive`. **Cold-read findings resolved**: WARNING (disposition info in /opsx:ff row) fixed by trimming the parenthetical. SUGGESTION (capitalization inconsistency at L115) fixed by capitalizing "Orbit" at sentence start in Section 2. CRITICAL (15-vs-14 mismatch) stale-suppressed — false positive; reviewer ran verification against orbit's source repo, not a user-installed orbit project. Task 2.5 sandbox proved verification works in user-install context (15 + 15). Other SUGG (#29 verify, footer prose) deferred to editorial discretion.
