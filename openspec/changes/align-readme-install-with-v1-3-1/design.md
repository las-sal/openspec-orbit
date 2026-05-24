## Context

Cluster 2 of v0.1.0 (issues #6 + #23) was bundled into the change `lean-overlay-and-add-orbit-onboard`. That change was cut to chunks 1+2 on 2026-05-23 after a fresh-sandbox install verification surfaced that v1.3.1's actual install surface differs materially from issue #6's body claims (10 skills + 10 commands from `init --tools claude`, not the "11 upstream + feedback" the issue body documented; no non-interactive `expanded` profile preset; `feedback` and `opsx/sync.md` are never installed by upstream init). The pegging-strategy core landed (`Distribution model — pegged engine + orbit-owned surface` + `Upstream version pinning` + `Overlay file disposition` baseline requirements; CLAUDE.md/README opening reframed; `feedback` skill deleted from orbit's overlay). Issue #6 closed under the pegging-strategy framing.

What remained deferred: the README Installation/Updating/Uninstalling sections (L898-L1020) that still describe pre-pegging state, the missing `rm -rf .claude/skills/feedback` prune step that produces the specific user-visible bug from the cluster, and the orbit-authored `openspec-onboard` SKILL body (issue #23). The explore for this change (`openspec/explore/align-readme-install-with-v1-3-1/explore.md`) decided to sequence those two deferrals — README rewrite first (this change), orbit-onboard SKILL body as the follow-up — to eliminate the bundling-induced premise-invalidation risk that killed the previous attempt.

The explore also resolved its 4 open questions during propose-time via inline sandbox + local checks (D5/D6/D7) so no `@review:` markers carry forward.

## Goals / Non-Goals

**Goals:**
- Rewrite README L898-L1020 (Installation + Updating + Uninstalling sections) so they describe v1.3.1's actual install surface accurately
- Add `rm -rf .claude/skills/feedback` prune step to install, update, and uninstall sections (closes the user-visible bug from cluster 2)
- Fix the dead `#pegging-strategy` anchor at L54, L902, L937 by adding a `### Pegging strategy` subsection inside the Installation section
- Establish "correct install state" as documented authority that the orbit-onboard follow-up can reference

**Non-Goals:**
- **Orbit-authored `openspec-onboard` SKILL body** — the follow-up change after this one. #23 stays open.
- **Install-script with version-check enforcement** — tracked as #26. This change keeps doc-only pin enforcement; no installer is built or specified.
- **Cross-reference to `Overlay file disposition` baseline requirement from the README install section** — rejected during explore (see Considered & out). Meta-explanation belongs in orbit-onboard's quick-reference (follow-up), not in install steps.
- **Major spec changes** — the existing baseline requirements (`Distribution model — pegged engine + orbit-owned surface`, `Upstream version pinning`, `Overlay file disposition`) were synced in the just-archived `lean-overlay-and-add-orbit-onboard` and are NOT modified by this change. This change adds ONE small new requirement to `orbit-conventions` codifying the README-matches-install-surface rule (see D-conventions-1 below); that's the entire spec scope.
- **Other README sections** — the rewrite is scoped to L898-L1020. Other parts of README (workflow descriptions, command reference, etc.) are out of scope even if they mention install-related concepts in passing.
- **Upstream version upgrade** — orbit stays pinned to `@fission-ai/openspec@1.3.1`. Upgrade is a separately-proposed change per existing baseline `Upstream version pinning` requirement.
- **Re-grounding orbit-onboard Setup verification scenarios as part of this change** — that's the follow-up's job. This change makes "correct install state" the documented authority; the follow-up references it.

## Decisions

### D-arch-1: Sequence the deferred cluster-2 work — README rewrite first, orbit-onboard SKILL body second

**Decision**: Split the deferred chunks 3-5 work from `lean-overlay-and-add-orbit-onboard` into two independent changes. This change is doc-only (README L898-L1020). The orbit-authored `openspec-onboard` SKILL body comes in a follow-up change after this one archives.

**Why**: The previous bundled attempt got cut when one half of the scope (install-surface premise) invalidated the other half (orbit-onboard Setup verification scenarios that referenced files not actually installed by upstream). Sequencing decouples the failure modes. Once this change lands, "correct install state" becomes documented ground truth that orbit-onboard's Setup verification can reference as authority — eliminating the premise-invalidation risk that killed the previous attempt.

**Why not bundled again**: Same coupling shape that killed the previous attempt; no upside that sequencing doesn't preserve. The cost of two smaller changes (slight redundancy in review/archive ceremony) is less than the cost of a second cut.

### D-arch-2: Scope γ — full Install + Update + Uninstall pass

**Decision**: This change rewrites README.md from L898 (`## Installation`) through L1020 (end of `### Uninstalling`). Roughly 120 lines, single contiguous section, no spec deltas, no skill rewrites.

**Why γ over α/β**:
- **α (just install steps + counts)** — leaves Update + Uninstall sections without the `rm -rf .claude/skills/feedback` prune step, which IS the specific user-visible bug from this cluster. The prune step needs to appear in all three of install/update/uninstall because `cp -r` overlay install doesn't auto-prune target-only files in user projects.
- **β (α + partial-adoption)** — better coherence inside the install steps but still leaves uninstall stale.
- **γ** — done-once-doesn't-pull-more-work-later property. Touches one contiguous README section, no spec or skill scope, no escalation surface.

**Alternative considered**: bundled with orbit-onboard SKILL body (the "Option A" from explore). Rejected — same coupling risk that killed the previous attempt.

### D-arch-3: Fix the dead `#pegging-strategy` anchor by adding a subsection — not by removing references

**Decision**: Add a `### Pegging strategy` subsection inside the Installation section. The subsection becomes the anchor target for three live forward-references currently pointing nowhere: L54 (intro), L902 (install opening), L937 (idempotent-overlay note).

**Why add the section, not remove the refs**:
- The 3 references are well-placed editorially — each points readers to the pegging rationale at the moment of encountering the pegging-relevant concept (intro, install, update-discipline).
- Removing them leaves each location without a rationale link, increasing the chance of pegging-strategy confusion down the line.
- The original chunks 1+2 explicitly added these forward-refs anticipating that chunk 3 (now this change) would create the target. The cut left the target unbuilt; building it here finishes the work cleanly.

**Section content** (mirroring `orbit-conventions` `Upstream version pinning` baseline requirement):
- Pinned version: `@fission-ai/openspec@1.3.1`
- Upgrade discipline: a deliberate change proposal, not silent maintenance
- Why orbit doesn't auto-ingest upstream improvements: explicit acceptance per design D-arch-1 in `lean-overlay-and-add-orbit-onboard` archive
- Doc-only enforcement; install-script future-work pointer (#26)

### D-verification-1: Tight propose-time sandbox verification — questions resolved before artifacts written

**Decision**: All 3 sandbox-resolvable open questions from the explore (Q1: bare `init` behavior; Q2: `v0.1.0` git tag existence; Q4: `init --force` actual behavior) were resolved during propose-time via inline sandbox + local checks. The findings became explore Decisions D5/D6/D7. No `@review:` markers carry forward into the generated artifacts.

**Why resolve now vs defer to apply-phase markers**: 
- Sandbox cost is low (`mktemp -d` + npx invocations + ~3 minutes total wall time)
- Resolving now produces fully-grounded artifacts; resolving via markers leaves the artifacts with unresolved questions that apply-time pushback has to revisit
- The earlier we know "did this premise hold," the cheaper it is to recover if it didn't

**Verified facts (carry into the rewrite)**:
- D5 (Q1): `npx -y @fission-ai/openspec@1.3.1 init` non-interactive errors with "No tools detected and no --tools flag provided." → README L916 needs `--tools claude` added.
- D6 (Q2): `git tag -l 'v0.1*'` returns `v0.1.0` → L995 `git clone --branch v0.1.0` reference is valid as-is.
- D7 (Q4): `init --force --tools claude` restores upstream-pristine state for modified upstream skills AND preserves user-created `.claude/` content outside upstream paths AND preserves `openspec/` content (verified end-to-end with md5 hash check + file-existence checks). → L1013-1016 warning is overly cautious; rewrite with accurate scope statement.

### D-verification-2: Tight apply-time sandbox verification — only the documented CLI commands

**Decision**: At apply time, after the README rewrite is in place, run a fresh sandbox (`mktemp -d` + same Node version) and verify the specific CLI commands the rewritten README documents execute as the prose claims. Verify:
- `openspec init --tools claude` produces 10 skills + 10 commands as the rewritten counts table claims
- The documented `cp -r` overlay sequence works against the post-init state and produces the expected post-overlay layout
- `rm -rf .claude/skills/feedback` does the right thing (no error if absent; clean removal if present)
- The documented Updating-orbit flow (`cp -r` re-overlay + `rm -rf feedback`) works on a post-install state without producing unexpected file additions/deletions
- The documented Uninstall flow (remove 4 orbit commands + 4 orbit skill dirs + `init --force --tools claude`) restores upstream-pristine state for upstream skills

**Why tight, not broad**:
- The broader install-surface state (skill counts, command names, config defaults) was sandbox-verified at propose time and on 2026-05-23. Re-verifying that adds ceremony without added confidence.
- The new risk is specifically in the CLI commands the rewrite would document — that's what tight verification covers.
- Broad verification can be invoked on-demand if tight verification surfaces something unexpected.

**Escalation procedure**: if apply-time tight sandbox verification surfaces a new premise problem (e.g., something propose-time sandbox missed because of state-ordering quirks), the apply phase halts and a `@review(escalated):` marker captures the finding for user resolution. Don't ship documentation based on unverified CLI behavior.

### D-conventions-1: Codify README-matches-install-surface as an `orbit-conventions` requirement

**Decision**: ADD one requirement to `orbit-conventions` baseline: "Install documentation describes actual install surface." Five scenarios codify the rule: skill/command counts match install output, CLI invocations are non-interactively executable, README-modifying changes pair with sandbox verification, upgrade/overlay-change proposals include README synchronization, and user-facing behavior warnings are accurate (neither understated nor overstated).

**Why codify, not leave as discipline**:
- This exact drift bug has already caused two cluster-2 cuts. Issue #6's body claimed v1.3.1 produces "11 upstream skills + feedback" — wrong. Original `lean-overlay-and-add-orbit-onboard` chunks 3-5 spec content was authored against those wrong assumptions; the sandbox verification on 2026-05-23 caused the cut.
- Twice-burned is "obviously" the threshold for codifying — a baseline requirement makes future changes self-aware about README impact during apply, not only at sandbox-verification crisis time.
- The requirement is small (~20 lines of spec text); the leverage is large (every future orbit-conventions modification or pinned-version upgrade will know to consider README sync).

**Scenarios** (per orbit-conventions `Severity ladder` / `Actionable findings` convention — phrased as testable WHEN/THEN pairs):

1. **README skill/command counts match install output** — `init --tools claude` produces files that match the README's "What you should see" enumeration exactly.
2. **CLI invocations in README are non-interactively executable** — every CLI command the README documents (init, overlay copy, prune, update, uninstall) succeeds (or exits with the documented error/warning) in a non-interactive shell; no command produces an unexpected error or hang from missing flags.
3. **README-modifying changes pair with sandbox verification** — any change proposal that modifies README install/update/uninstall sections SHALL include an apply-time fresh-sandbox verification task that runs the documented CLI sequence end-to-end; verification failures escalate via `@review(escalated):` rather than ship undocumented behavior. (Codifies the discipline THIS change itself demonstrates.)
4. **Upgrade and overlay-change proposals include README sync** — when a change proposal upgrades the pinned upstream version (per `Upstream version pinning`) OR changes overlay file disposition (per `Overlay file disposition`), the proposal SHALL include a README install-section synchronization task.
5. **User-facing behavior warnings are accurate, not pessimistic** — README warnings about CLI command behavior (e.g., `init --force` overwrite scope, `cp -r` non-deletion) SHALL accurately describe what the command does in the pinned version, neither understating nor overstating risk. Overly-cautious warnings that don't match actual behavior are themselves drift. (Codifies the D7 finding about `init --force` being safer than the existing prose implied.)

**Alternatives considered**:
- **Leave as unwritten discipline** — rejected. Discipline relied on for two cluster-2 attempts and failed both times.
- **Add scenarios to existing `Upstream version pinning` requirement** — viable but bundles README-sync semantics with version-pin semantics; cleaner to have a dedicated requirement for the documentation-vs-install-surface rule.
- **Make this a separate capability (e.g., `orbit-install-docs`)** — rejected. The rule is a single requirement; standing up a new capability for it is over-engineering. `orbit-conventions` is the right home.

### D-verification-3: Update project memory after apply-time verification

**Decision**: After apply-time sandbox verification passes, update `[[readme-install-section-staleness]]` project memory to reflect that the README install/update/uninstall sections are no longer stale. The memory was explicitly marked as temporary scaffolding for this change (per its body: "Delete it at archive-time of the follow-up").

**When to delete vs update**: 
- This change resolves the README install section staleness specifically
- The orbit-onboard SKILL body staleness is a separate concern still tracked at the bottom of that memory ("Related skill bodies still upstream-bodied")
- Delete the README-install-section content; keep the skill-body content (until the follow-up archives)
- Alternatively: split the memory into two files, one for README (delete after this change) and one for skill bodies (delete after the follow-up). Simpler to just trim the README content from the existing file and rename.

**Implementation note**: This is project-memory hygiene, NOT a spec-driven task that lives in tasks.md. The apply phase performs it after sandbox verification passes but before user-validation. Captured here in design.md so the apply phase knows it's part of completeness.

## Risks / Trade-offs

- **README L898-L1020 rewrite is bulky** — ~120 lines of contiguous prose. Risk: edit conflicts if the user is concurrently editing the README. Mitigation: change apply is bounded to one section; user can defer concurrent edits or coordinate via the apply chunk emit.

- **Apply-time sandbox verification may surface a new premise problem** — `init --force` behavior was verified at propose time (D7) but the propose-time check exercised a specific orbit-overlay-emulation state. Apply-time should run the verification against the actual orbit overlay (`cp -r` from the orbit repo, not synthesized modifications) for a more realistic check. Risk: orbit's actual overlay state contains files the propose-time emulation didn't (e.g., orbit-authored commands, custom subdirectories under skills). Mitigation: tight apply-time verification (D-verification-2) catches this; escalation procedure routes through `@review(escalated):` if needed.

- **README-vs-spec drift over time** — this change aligns README prose with the post-archive `orbit-conventions` baseline. If `orbit-conventions` is later updated without a parallel README update, drift returns. Mitigation: future spec-modifying changes should include README-sync tasks where the change touches install-related requirements. Not enforced by tooling; convention-by-discipline.

- **Memory cleanup at apply time** — D-verification-3 prescribes trimming `[[readme-install-section-staleness]]` after verification passes. If the apply phase forgets, the memory remains marked "stale" even after the rewrite. Mitigation: apply task explicit ("update memory after sandbox passes"); memory body itself describes when to delete so future agents can self-correct.

- **Orbit-onboard follow-up depends on this archiving cleanly** — if this change ships with a subtle README error that the orbit-onboard follow-up references as ground truth, the error propagates. Mitigation: tight apply-time sandbox verification + a user-validation handoff (read the rewritten section cold and confirm clarity) before archive.
