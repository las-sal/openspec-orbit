<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):   README install-section rewrite (L898-L1020): all subsections, single
                       contiguous edit + Pegging strategy anchor target
  Chunk 2 (group 2):   Apply-time sandbox verification (tight per D-verification-2)
                       + project memory cleanup (per D-verification-3)
  Chunk 3 (group 3):   Validation + user-validation handoff

Total: 3 implementation chunks. All sandbox-resolvable open questions from
the explore were resolved during propose-time (D5/D6/D7); no @review: markers
carry forward.
-->

## 1. README install-section rewrite (L898-L1020)

- [x] 1.1 Add new `### Pegging strategy` subsection inside `## Installation` (after `### Prerequisites`, before `### 1. Initialize upstream OpenSpec` — or wherever it reads most naturally). Content mirrors `orbit-conventions` baseline `Upstream version pinning` requirement: pinned version `@fission-ai/openspec@1.3.1`, upgrade-is-a-deliberate-change-proposal discipline, why orbit doesn't auto-ingest upstream improvements (per design D-arch-1 of `lean-overlay-and-add-orbit-onboard` archive), doc-only enforcement, install-script future-work pointer ([#26](https://github.com/las-sal/openspec-orbit/issues/26)). Section MUST have HTML anchor `pegging-strategy` (markdown heading auto-anchor; verify by grep that 3 existing forward-refs at L54, L902, L937 link correctly after edit).
- [x] 1.2 Rewrite `### 1. Initialize upstream OpenSpec` step. Per design D-verification-1 D5: change the `npx @fission-ai/openspec@latest init` invocation at L916 to `npx @fission-ai/openspec@latest init --tools claude`. Update the post-init claim at L919 from "11 upstream `openspec-*` skills (plus the `feedback` skill)" to "10 upstream `openspec-*` skills" (no `feedback` — never installed by upstream init per [[openspec-1-3-1-actual-install-surface]]). Verification line: replace "Verify with `openspec list` — you should see no active changes yet and no errors" with the same plus a count check (e.g., `ls .claude/skills/ | wc -l` should equal 10).
- [x] 1.3 Rewrite `### 2. Overlay orbit` step. Update the post-overlay bullet list:
  - "Adds four new orbit slash commands" — unchanged
  - "Overwrites three upstream skill files" — CHANGE to enumerate the actual 10 orbit-modified skills. Origin breakdown (verified via git pickaxe + grep): originally 3 in bootstrap (`explore`, `propose`, `archive-change`); emit-change added Orbit additions to 6 more (`apply-change`, `bulk-archive-change`, `continue-change`, `ff-change`, `new-change`, `verify-change`) and extended `archive-change`'s existing additions; emit-change also added Orbit additions to `openspec-onboard` (the orbit-onboard follow-up will then move openspec-onboard from "orbit-modified" to "orbit-authored" category — for this change, list onboard as orbit-modified per current state). Total currently with `# Orbit additions`: 10. Grep authoritative list at edit time (`grep -l "^# Orbit additions" .claude/skills/openspec-*/SKILL.md`); don't trust this task's listing as primary source.
  - "Overwrites the three corresponding slash command bodies" — same expansion.
  - "Leaves the other 8 upstream `openspec-*` skills + `feedback` untouched" — REMOVE entirely (incorrect on both counts).
  - ADD: `rm -rf .claude/skills/feedback` step immediately after the `cp -r` block, with prose explaining that `cp -r` overlay install does NOT delete files in the user's target project that don't exist in orbit's source, so `feedback/` from a prior orbit overlay persists unless explicitly removed (per `orbit-conventions` baseline `Overlay file disposition` "Not shipped — pruned from overlay" scenario).
- [x] 1.4 Rewrite `### What you should see after install` table (currently L943-L953). Correct counts:
  - `.claude/skills/openspec-*/` row: "14 directories total: 10 upstream + 4 new orbit" (was "15 total: 11 + 4")
  - `.claude/commands/opsx/` row: "14 command files: 10 upstream + 4 new orbit" (was "15 cmds: 11 + 4")
  - Drop "Plus `feedback/` separately" — `feedback` is not installed.
  - "Modified upstream skills" row: update enumeration to match the 10 orbit-modified skills from 1.3 (single source of truth at edit time).
  - Other rows (`openspec/lenses/`, `openspec/.orbit-runs/`) are correct as-is; leave unchanged.
- [x] 1.5 Rewrite `### Partial adoption is fine` (L955-L961). Update the "Install only the four new commands and skip the upstream-skill modifications" bullet: skipping orbit-modified skills means skipping 10 modifications (not 3); the partial-adoption surface is larger. Sweep prose for stale "3 modified" assumptions and update.
- [x] 1.6 Rewrite `### Working alongside existing .claude/ customizations` (L963-L969). The bullet listing "`openspec-explore`, `openspec-propose`, or `openspec-archive-change`" as the orbit-modified skills MUST update to the full 10 modified list (single source of truth at edit time, same as 1.3 and 1.4). User customizations on any of those 10 skills get clobbered by orbit's overlay; the warning needs to cover all of them.
- [x] 1.7 Sweep `### Common gotchas` (L971-L978) for staleness. The 5 bullets are mostly procedural and likely still accurate. Verify each against current state:
  - "Skipped step 1" gotcha — still accurate
  - "Stale AI client cache" — still accurate
  - "`/opsx:review-external` without git" — still accurate
  - "`openspec` CLI not on PATH" — still accurate
  - "`openspec/lenses/` looking empty" — still accurate
  - "First `/opsx:review` finding has no baseline" — still accurate
  - If any reference v0.0.x or pre-pegging concepts, update; otherwise leave.
- [x] 1.8 Rewrite `### Updating orbit` (L980-L998). The `cp -r` re-overlay block at L984-L990 MUST add `rm -rf .claude/skills/feedback` step after the cp commands (same reason as 1.3 — `cp -r` doesn't auto-prune). The version-pin example at L995 (`git clone --branch v0.1.0 --depth 1`) is valid as-is per propose-time D6 — leave unchanged. The "Long-term: package as a proper Claude Code plugin" note is forward-looking, leave unchanged.
- [x] 1.9 Rewrite `### Uninstalling / reverting to upstream-only` (L1000-L1018). Per design D-verification-1 D7:
  - Keep the `npx @fission-ai/openspec@latest init --force` command BUT change it to `init --force --tools claude` for consistency with 1.2 and the actual non-interactive behavior.
  - Replace L1016 "Note: `openspec init --force` may overwrite other .claude/ content too — check the upstream init's behavior before running" with the propose-time-verified scope statement: `init --force --tools claude` restores upstream-installed skills + commands to pristine state but does NOT touch user-created files outside upstream paths (e.g., `.claude/custom/`, custom non-upstream skills) and does NOT touch any `openspec/` content (`changes/`, `specs/`, `archive/`).
  - Keep "safer pattern" advice about keeping upstream files in version control as a complementary belt-and-suspenders option (not a replacement for `init --force`).
  - Update the bullet list of orbit files to remove: must now include `rm -rf .claude/skills/feedback` for users who explicitly want pure-upstream state (since feedback was orbit-historical, not upstream).
- [x] 1.10 After all rewrites in 1.1-1.9 complete: re-grep README for the residue patterns to confirm all stale references are gone:
  - `grep -n "11 upstream" README.md` — should return 0 results
  - `grep -n "the other 8 upstream" README.md` — should return 0 results
  - `grep -n "(plus the \`feedback\` skill)" README.md` — should return 0 results
  - `grep -n "three upstream skill" README.md` — should return 0 results
  - `grep -n "Plus \`feedback/\` separately" README.md` — should return 0 results
  - All 3 `#pegging-strategy` forward-references should now resolve to a real anchor (grep heading + count vs reference count)
  - If any residue remains, fix before declaring chunk 1 done.

## 2. Apply-time sandbox verification + memory update

- [x] 2.1 (Per design D-verification-2) Run a fresh sandbox verification of the rewritten README:
  - `SANDBOX=$(mktemp -d)` + same Node version as current orbit dev environment + `npx -y @fission-ai/openspec@1.3.1 --version` confirms 1.3.1 pin
  - Run `npx -y @fission-ai/openspec@1.3.1 init --tools claude` → assert 10 skills + 10 commands, no `feedback/`
  - Apply the documented `cp -r` overlay from a fresh clone of orbit (NOT the dev copy — use a temp clone) → assert 14 skills + 14 commands post-overlay
  - Run `rm -rf .claude/skills/feedback` → assert no error, file absent
  - Verify the documented Updating-orbit flow (re-overlay + prune) does not introduce unexpected file changes
  - Verify the documented Uninstall sequence (remove 4 orbit commands + 4 orbit skill dirs + `init --force --tools claude`) restores upstream-pristine state for upstream skills AND preserves user-created `.claude/` content (test by creating a `.claude/custom/test.md` file before uninstall, asserting it survives)
  - Document the exact commands run + observed output in the chunk-end emit.
- [x] 2.2 If 2.1 surfaces any premise problem (CLI behavior differs from README prose), HALT and add `@review(escalated):` markers at the relevant README locations; do NOT proceed to chunk 3 until escalations are resolved per `openspec-address-reviews` discipline. **Triggered + resolved inline.** Initial sandbox in 2.1 surfaced count mismatch: README claimed 14 skills + 14 commands, actual was 15 + 16. Premise problem reported to user with escalation framing; user directed inline fix rather than formal `@review(escalated):` walk. Inline fix added: `openspec-sync-specs` (upstream-required primitive), `sync.md`, `fast-forward.md` accounting + `ff.md`/`fast-forward.md` naming-divergence note in both `### 2. Overlay orbit` bullets and `### What you should see after install` table. Sandbox v2 re-run with corrected README: all 8 verification checks PASS.
- [x] 2.3 (Per design D-verification-3) Trim the `[[readme-install-section-staleness]]` project memory. The memory body explicitly says "Delete it at archive-time of the follow-up" — this change IS the relevant follow-up for the README install-section content. Remove the README install-section staleness section; preserve the "Related skill bodies still upstream-bodied" section (that part is still load-bearing for the next change — the orbit-onboard SKILL body rewrite). Update memory frontmatter description to reflect the narrower remaining scope.

## 3. Validation + user-validation handoff

- [x] 3.1 Run `openspec validate align-readme-install-with-v1-3-1 --strict`; resolve any validation findings.
- [ ] 3.2 (User-validation) User reads the rewritten README L898-L1020 cold and confirms:
  - (a) Pegging strategy subsection content is clear and the anchor works from all 3 forward-reference sites (L54, L902, L937)
  - (b) Install steps are executable and correct (CLI commands work, counts match, prune step is in place)
  - (c) Update + Uninstall sections include the `rm -rf .claude/skills/feedback` step
  - (d) `init --force` warning prose matches the verified scope (no understating, no overstating)
  - (e) The rewritten section reads cleanly as a single contiguous narrative — no abrupt tone shifts where chunks 1 sub-tasks were stitched together
  - Surface specific findings via `/opsx:review` (or inline `@review:` markers) rather than blocking apply.
- [ ] 3.3 If user-validation surfaces no blocking findings, the change is ready for `/opsx:archive`.
