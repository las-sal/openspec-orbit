# Explore: align-readme-install-with-v1-3-1

## Premise

The just-archived `lean-overlay-and-add-orbit-onboard` change cut to chunks 1+2 (pegging declaration + narrative reframe + feedback skill deletion + stale-prose sweep). Chunks 3-5 — README install-section rewrite and orbit-authored `openspec-onboard` SKILL body — were deferred to follow-up changes after a 2026-05-23 fresh-sandbox install verification surfaced that v1.3.1's actual install surface differs materially from issue #6's body claims and the README's existing accounting.

This is the first of two follow-up changes: **rewrite the README Installation, Update, and Uninstall sections** so they describe v1.3.1's actual install surface and include the missing `rm -rf .claude/skills/feedback` prune step. The orbit-onboard SKILL body comes in a separate follow-up.

Memory artifacts that ground this explore:
- [[openspec-1-3-1-actual-install-surface]] — verified sandbox findings on `openspec init --tools claude`
- [[readme-install-section-staleness]] — line-by-line list of stale claims in README L898-1020 captured 2026-05-24

## Decisions

### D1: Sequence the deferred work — this change first, orbit-onboard follow-up second

**Decision**: Split the deferred chunks 3-5 work into two independent changes. This change is doc-only (README Install/Update/Uninstall). orbit-onboard SKILL body is its own change after this one archives.

**Why**: The previous bundled change got cut when one half of the scope (install-surface premise) invalidated the other half (orbit-onboard Setup verification scenarios). Sequencing eliminates the coupling — once this change lands, "correct install state" becomes documented ground truth that orbit-onboard's Setup verification can reference as authority.

**Why not bundled again**: Same premise-invalidation risk that killed the previous attempt. Two smaller changes are cheaper to apply individually than one big change with two failure modes.

### D2: Scope γ — full Install + Update + Uninstall pass

**Decision**: This change rewrites README.md from L898 (`## Installation`) through L1020 (end of `### Uninstalling`). Roughly 120 lines, single contiguous section, no spec deltas, no skill rewrites.

**Why γ over α (just counts) or β (counts + partial-adoption)**:
- α/β fix the words inside install steps but leave the **update** section telling users to `cp -r` without pruning `feedback/` — which is the specific user-visible bug from this whole cluster. The prune step lives in the update + uninstall sections, so cutting them out leaves the bug.
- γ is still bounded — one README section, no spec or skill scope. The "premise invalidation" failure mode doesn't apply because the install surface is already verified and captured in memory.
- Done-once-doesn't-pull-more-work-later property: γ resolves the entire post-pegging install-doc surface in one pass.

### D3: Tight sandbox verification — verify only the documented CLI commands

**Decision**: At apply time, run a fresh sandbox (`mktemp -d` + same Node version) and verify the specific CLI commands the rewritten README tells users to run: `openspec init --tools claude`, the documented `cp -r` overlay sequence, `rm -rf .claude/skills/feedback`, the documented update flow, and the documented uninstall sequence (including `init --force` behavior).

**Why tight, not broad**:
- The broader install-surface state (10 skills + 10 commands, no feedback, no expanded preset, etc.) was sandbox-verified less than 48 hours ago (2026-05-23) and captured in memory. Re-verifying that adds ceremony without added confidence.
- The new risk is specifically in the CLI commands the rewrite would document — that's what tight verification covers.
- Broad verification can be invoked on-demand if the tight verification surfaces something unexpected.

**Escalation procedure**: if tight sandbox verification surfaces a new premise problem (e.g., `init --force` doesn't behave as L1013 describes), the change halts and a `@review(escalated):` marker captures the finding for user resolution. Don't ship documentation based on unverified CLI behavior.

### D4: Fix the dead `#pegging-strategy` anchor by adding a subsection — not by removing references

**Decision**: This change adds a `### Pegging strategy` subsection inside the Installation section, providing the anchor target that 3 live forward-references at L54, L902, L937 currently point to nothing.

**Why add the section, not remove the refs**:
- The 3 references are well-placed — they appear at the intro (L54), the install opening (L902), and the idempotent-overlay note (L937), each pointing readers to the rationale for pinning. Removing them would leave each location with no rationale link.
- The original chunks 1+2 explicitly added these forward-refs anticipating that chunk 3 (now this change) would create the target. The cut left the target unbuilt; building it now finishes the work cleanly.

**Section content**: pinned version (`@fission-ai/openspec@1.3.1`), upgrade discipline (a deliberate change proposal, not silent maintenance), why orbit doesn't auto-ingest upstream improvements, doc-only enforcement plus the install-script future-work pointer (#26).

### D5: README install command at L916 documents `npx @fission-ai/openspec@latest init --tools claude` (resolved Q1 via inline sandbox)

**Decision**: The init command documented in the README MUST include the `--tools claude` flag. Bare `openspec init` (without `--tools claude`) is non-functional for the README's documented non-interactive flow.

**Sandbox verification (2026-05-24)**:
- `npx -y @fission-ai/openspec@1.3.1 init </dev/null` (non-interactive) returns `✖ Error: No tools detected and no --tools flag provided.` followed by an enumeration of 26 valid tools. No `.claude/` directory is created. Exit code is reported as 0 in the test wrapper but the error is real and `.claude/` is missing.
- `npx -y @fission-ai/openspec@1.3.1 init --tools claude` (the form already used in the 2026-05-23 verification) produces 10 skills + 10 commands successfully.

**Implication**: the README L916 form `npx @fission-ai/openspec@latest init` is incorrect. The fix is straightforward: add `--tools claude`.

### D6: L995's `v0.1.0` git tag reference is valid as-is (resolved Q2 via local check)

**Decision**: L995 (`git clone --branch v0.1.0 --depth 1 https://github.com/las-sal/openspec-orbit /tmp/orbit`) requires no change. The tag exists.

**Verification (2026-05-24)**: `git tag -l 'v0.1*'` returns `v0.1.0`. The version-pin pattern in the README's "Updating orbit" section works as documented.

### D7: `init --force` behavior is safer than the current README warning implies (resolved Q4 via inline sandbox)

**Decision**: The README's L1013-1016 uninstall step using `npx @fission-ai/openspec@latest init --force` is safe for the documented use case. The current "may overwrite other .claude/ content too" warning is overly cautious for v1.3.1. The rewrite SHALL keep `init --force --tools claude` as the primary uninstall mechanism for restoring upstream-pristine skills, with a more accurate scope statement of what it does and does not touch.

**Sandbox verification (2026-05-24)** — sequence: fresh `init --tools claude` → captured md5 of `openspec-explore/SKILL.md` (`d160105373f4fe54bfbe1790b343c51b`, 288 lines) → modified that skill (`7673d9d7a563f2399926f338c789cd01`, 290 lines) → added `.claude/custom/user-notes.md` + `openspec/changes/test-user-change/proposal.md` → ran `init --force --tools claude` → re-captured state:

- ✅ Modified upstream skill restored to pristine: md5 back to `d160105373f4fe54bfbe1790b343c51b`, 288 lines.
- ✅ User-created `.claude/custom/user-notes.md` preserved intact with original content.
- ✅ User-created `openspec/changes/test-user-change/proposal.md` preserved intact.
- ✅ 10 skills before, 10 skills after — no spurious additions or removals.

**Implication for the README rewrite**:
- Keep `init --force --tools claude` as the primary uninstall mechanism for restoring upstream-pristine skills.
- Replace the "may overwrite other .claude/ content too" warning with an accurate scope statement: "restores upstream-installed skills + commands to pristine state; does not touch user-created files outside upstream paths or `openspec/` content."
- The "safer pattern" advice (keep upstream files in version control for `git checkout` revert) is still good belt-and-suspenders guidance and stays in the doc — but as a complement to `init --force`, not a replacement.

## Open questions

(All open questions resolved during propose-time sandbox verification — see D5, D6, D7. Q3 resolved during propose-time prompt — see Considered & out.)

## Considered & out

- **Bundled change (Option A) — rewrite README install + orbit-onboard SKILL body together.** Rejected. Same coupling shape that killed the previous attempt; no upside that sequencing doesn't preserve.
- **Scope α (just counts) and Scope β (counts + partial-adoption)**. Rejected. Both leave the update + uninstall sections without the `rm -rf feedback` prune step — which is the specific user-visible bug this whole cluster was supposed to fix.
- **Broad sandbox verification — re-run full install-surface check at apply time.** Rejected as ceremony without added confidence; the 2026-05-23 verification is fresh and memory-captured. Tight sandbox verification during propose (D5/D7) was sufficient.
- **Remove the 3 dead `#pegging-strategy` references instead of adding the section.** Rejected. The references are well-placed; the target was the missing piece. Adding the section finishes the work cleanly.
- **Re-grounding orbit-onboard Setup verification as part of this change.** Rejected — that's the follow-up change's scope. This change establishes "correct install state" as documented authority; the follow-up references it.
- **Cross-reference the `Overlay file disposition` baseline requirement from the README install section** (originally Q3). Rejected during propose-time prompt. The 4-category framework lives in `orbit-conventions` baseline; the README install section's job is user-facing install steps. Meta-explanation belongs in orbit-onboard's quick-reference section (next follow-up change), not here. Keeps this change tight.

## References

### Closes (intended)

- (no new GH issue tracking this change explicitly — captures the cluster-2 install-doc tail. If a tracking issue is wanted, file at change-creation time.)

### Related (open)

- **#23** — orbit-authored `openspec-onboard` SKILL body. The next follow-up change after this one.
- **#26** — install-script with version-check enforcement. Still doc-only enforcement post-this-change; #26 closes when the install script lands.

### Archived precedents

- `openspec/changes/archive/2026-05-24-lean-overlay-and-add-orbit-onboard/` — the cut change whose chunks 3-5 deferral led to this explore. Its `proposal.md`, `design.md`, and `tasks.md` describe the original bundled scope.
- `openspec/changes/archive/2026-05-21-emit-run-summary-jsons-from-workflow-commands/` — the change that added `# Orbit additions` sections to 7 skills, making the README's "3 modified upstream skills" claim stale.

### Memory

- [[openspec-1-3-1-actual-install-surface]] — verified sandbox findings (2026-05-23) on `openspec init --tools claude`
- [[readme-install-section-staleness]] — line-by-line list of stale claims in current README L898-1020 (captured 2026-05-24)
- [[orbit-pegged-to-openspec-1-3-1]] — strategic pegging decision driving this work
- [[orbit-supports-full-openspec-1-3-1]] — default scope: keep upstream features unless real reason to drop

### Baseline specs (relevant)

- `openspec/specs/orbit-conventions/spec.md` — `Distribution model — pegged engine + orbit-owned surface` + `Upstream version pinning` + `Overlay file disposition` (synced in commit `6f79f74`). The pegging-strategy subsection's content should mirror this requirement's framing.
