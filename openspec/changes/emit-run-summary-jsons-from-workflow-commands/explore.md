# emit-run-summary-jsons-from-workflow-commands

## Premise

Today, orbit's editorial commands (`/opsx:review`, `/opsx:address-reviews`, `/opsx:audit-drift`, `/opsx:archive`) emit structured JSONs to `openspec/changes/<name>/.orbit-runs/` with a `next_recommended` field — orbit's own recommendation for the next step. Workflow commands (`/opsx:explore`, `/opsx:propose`, `/opsx:apply`, `/opsx:verify`) don't emit anything. State is recoverable only indirectly, via artifact presence.

This forces downstream consumers (orbit-status, dashboards, CI) to **synthesize** a recommendation from artifact presence for pre-review changes. orbit-status currently ships a Tier-2 synthesis layer at `orbit-status-recommendation/spec.md:23–28` containing 4 rules that exist purely to fill the gap left by silent workflow commands.

**Goal**: workflow commands (and other artifact-mutating skills) emit `.orbit-runs/<command>-<TS>.json` so downstream consumers read orbit's own recommendation everywhere. orbit-status's Tier-2 synthesis layer becomes deletable.

**Tracking issue**: [#8](https://github.com/las-sal/openspec-orbit/issues/8) — `P1`, `v0.1.0_issuecluster_1`.

## Decisions

### D1: `next_recommended` stays a verbatim string

The producer (orbit) emits `next_recommended` as a single string, possibly prose for ambiguous recommendations ("either A or B"). The consumer (orbit-status) does best-effort parsing for a leading `/opsx:<verb> [args]` token; if parse fails, the verbatim string is preserved in `reason`.

**Rationale**: orbit-status today already implements verbatim-string-with-best-effort-parse at `orbit-status-recommendation/spec.md:7`. Keeping producer side as string-only avoids a redundant schema lock-in and preserves prose nuance for ambiguous cases. Parsing is on the right side of the producer/consumer boundary. Revisit if a future consumer can't tolerate parsing.

### D2: Per-variant filenames for propose-shaped commands

`/opsx:new`, `/opsx:continue`, `/opsx:ff` each emit JSONs with their own command-name prefix in the filename:

- `explore-<TS>.json`
- `propose-<TS>.json`
- `new-<TS>.json`
- `continue-<TS>.json`
- `ff-<TS>.json`
- `apply-<TS>.json`
- `verify-<TS>.json`
- `review-external-<TS>.json`
- `audit-drift-<TS>.json` (standalone runs only — inline audit-drift during archive is captured in `archive-<TS>.json`)

**Rationale**: orbit-status sorts and groups `.orbit-runs/` entries by filename prefix. Per-variant filenames preserve entry-point provenance for free; downstream tools can route on prefix without inspecting JSON body. Same shape, distinct origins.

### D3: Top-level `kind` field added in v1

Every run-summary JSON includes a `kind` field with one of three values:

- `kind: "workflow"` — forward-progressing commands: `explore`, `propose`, `new`, `continue`, `ff`, `apply`, `verify`
- `kind: "editorial"` — evaluative or resolution-focused: `review`, `address-reviews`, `audit-drift`, `review-external`
- `kind: "lifecycle"` — terminal transitions: `archive`

**Rationale**: makes implicit categorization explicit. Aligns with issue #8's own "workflow commands vs editorial commands" wording. archive gets its own `lifecycle` value because it doesn't fit either category cleanly — it transitions the change out of the active set rather than progressing or evaluating it.

### D6: Orbit's emit-layer wraps upstream skills without modifying them

The new emit behavior added by #8 is a **wrapper layer**, not a behavioral modification of the upstream commands themselves. Each wrapped command's behavior stays as upstream defines it; the emit-layer runs after the command completes, inspects its output, and writes `<command>-<TS>.json` with the spine + extensions + `next_recommended`.

Concretely: `/opsx:verify` is the clearest example. Verify's job stays exactly what upstream defines: run `verify-change`, report pass/fail with findings. It does NOT gain marker-drop behavior (that's `/opsx:review --mark`'s job). It does NOT gain spec-edit shortcuts. The recommendation logic for verify-fail (route to apply / review --mark / fix validation error) lives in the emit-layer, not in verify.

This principle generalizes to all wrapped commands:

- `/opsx:explore`, `/opsx:propose`, `/opsx:apply`, `/opsx:verify` — upstream skills, unchanged. Emit-layer wraps each with JSON output.
- `/opsx:new`, `/opsx:continue`, `/opsx:ff` — upstream skills, unchanged. Emit-layer wraps each.

Where orbit MODIFIES an upstream skill (the `## Orbit additions` pattern used today for `openspec-explore`, `openspec-propose`, `openspec-archive-change`), that's a separate concern from #8. #8 only adds emit; it does not introduce new behavior into any upstream skill.

**Rationale**: keeps the upstream/orbit boundary clean. Upstream skills receive `openspec update`-driven improvements without orbit's emit work interfering. Emit-layer is a thin observability shell that can evolve independently of the upstream skill's logic.

### D5: Bare-mode `/opsx:explore` does not emit; crystallization requires explicit warning

Bare-mode explore (no name argument) does NOT emit a JSON. It is treated as pre-commitment, ephemeral thinking. The user can crystallize at any point ("create a name and emit findings"), but the AI MUST surface an explicit warning at the moment of crystallization rather than transitioning silently.

Sketch of the warning text:

```
⚠ Crystallizing exploration into a named change: <name>

This will:
  • Create openspec/explore/<name>/explore.md (decisions captured so far)
  • Emit openspec/explore/<name>/.orbit-runs/explore-<TS>.json
  • Make <name> visible to `openspec list`, orbit-status, and any other consumers
  • Start the change's audit trail — every subsequent orbit command emits a JSON here

Before crystallization: ephemeral in-conversation thinking.
After crystallization: persistent on-disk presence; abandonment requires
formal archive/discard (not just deleting the directory once it's in changes/).

Confirm? [Y/n]
```

**Rationale**: bare-mode exploration is fundamentally pre-commitment; emitting JSONs before a name exists would either pollute project-scope (option b, rejected) or require per-session UUID directories (option c, rejected). Letting the user explore freely without commitment matches the "offer-don't-auto" stance and avoids accidentally seeding a partial change that the user didn't intend to start. The warning aligns with [#7](https://github.com/las-sal/openspec-orbit/issues/7) (don't auto-invoke) and [#15](https://github.com/las-sal/openspec-orbit/issues/15) (inflection points surface consequences).

### D4: Shared spine (5 required fields) + per-command extensions

Every emit MUST include:

```
command          string   identifies which command emitted
timestamp        ISO-8601 UTC string, also embedded in filename
change           string   the change name (or null for project-scope cmds)
final_assessment string   narrative of what just happened
next_recommended string   verbatim recommendation (per D1)
kind             enum     "workflow" | "editorial" | "lifecycle" (per D3)
```

Per-command extensions vary by command (initial draft below; refined as design proceeds):

```
explore        mode, decisions_captured, open_questions_count, crystallized_to_name
propose+vars   artifacts_created, delta_count, from
apply          tasks_completed, tasks_remaining, chunk, chunk_complete
verify         verdict, findings_count, findings_summary
review-ext     mode, prompt_path, target, awaiting_findings
audit-drift    categories_run, findings_by_category, findings_total
```

**Rationale**: matches the spine already in use by `review-*.json`, `address-reviews-*.json`, `archive-*.json`. Backward-compatible with orbit-status's existing reader.

### D7: `/opsx:apply` emits per chunk-end, not per session

Apply emits one `apply-<TS>.json` per chunk completion, plus an additional emit on mid-chunk session pause. Rules:

1. **Each chunk completion → emit.** When the last task in chunk N is checked, write `apply-<TS>.json` with `chunk: "N of M"`, `chunk_complete: true`. `next_recommended` advances to next chunk (`/opsx:apply <name>`) or to `/opsx:verify <name>` on apply-complete.
2. **Mid-chunk session pause → emit.** If the user stops or hands off mid-chunk-N, emit with `chunk: "N of M"`, `chunk_complete: false`. Captures in-flight state for resumability.
3. **No-chunking apply → single emit at session end.** Short changes without an explicit chunk preamble emit once with `chunk: null`.

**Schema (refines D4's per-command extension for apply):**

```
apply-<TS>.json
  tasks_completed              int   (running total across all chunks)
  tasks_remaining              int
  chunk                        string | null   ("3 of 5" or null)
  chunk_name                   string | null   ("phase+attention engine")
  chunk_complete               bool
  tasks_completed_this_session int   (delta since prior apply JSON — forensic key)
```

**Rationale**: enables post-hoc forensic timeline ("which chunk introduced this regression?"), provides clean resume boundaries between sessions, supports orbit-status's chunk-aware summary rendering, and creates the natural inflection point that [#22](https://github.com/las-sal/openspec-orbit/issues/22) (intra-apply review cadence) depends on. Worked example from orbit-status bootstrap: a 5-chunk apply over 42 minutes would have produced 5 chunk JSONs documenting exactly which tasks landed in each chunk's window — bounding the blast radius for any post-hoc bisection to tasks-from-chunk-N rather than the full 76-task universe.

### D8: #8 emits don't forward-look at other issues' future behavior changes

When a recommendation points at a command whose default behavior is expected to change in a future issue (e.g., `/opsx:review --as system` whose default reviewer mode will be inverted to external by [#20](https://github.com/las-sal/openspec-orbit/issues/20)), #8's emit-layer recommends the canonical command name and lets the future issue update *what that command does*. Emit text does NOT preview or anticipate future-issue behavior.

**Rationale**:
1. Clean responsibility split — #8 is "emit recommendations to the right next command"; #20 is "make that command's default smarter." Each issue stays in its lane.
2. No forward-reference rot — if a future issue's design shifts before landing, forward-looking text in emit becomes wrong and needs cleanup.
3. Automatic propagation — when a future issue ships, every prior emit that recommended its command benefits automatically without `#8` having to anticipate.

### D9: Verify-pass recommends `/opsx:review --as system` as canonical, with `/opsx:archive` surfaced in reason

When standalone `/opsx:verify` passes, the run-summary JSON's `next_recommended` reads:

```
"/opsx:review --as system <name> — verification clean; formal pre-archive
 review recommended (or /opsx:archive <name> if you're skipping the
 editorial pass)"
```

orbit-status's tier-1 reader best-effort parses the leading `/opsx:review --as system <name>` token into `command` and `args` fields. The reason text preserves the alternative `/opsx:archive` path for users who want to skip the formal review.

**Rationale**: orbit's canonical post-apply flow is `apply → verify → review --as system → archive`. The system-mode review is where the editorial value lands (5 passes beyond verify-change). Skipping it is possible but loses that signal — so the primary recommendation honors the canonical path. Per [#15](https://github.com/las-sal/openspec-orbit/issues/15) (inflection points list options), the alternative is surfaced in reason without making the primary command field ambiguous. The user's autonomy to ignore the recommendation and go directly to archive is preserved without the emit defaulting away from the canonical advice.

### D11: `/opsx:explore` (named mode) recommends `/opsx:propose` only as decisions accumulate

Named-mode explore emits at session boundaries with a recommendation that adapts to maturity:

- **Early (0–1 decisions captured)**:
  `"/opsx:explore <name> — continue capturing thinking"`
- **Mid (2–3 decisions captured)**:
  `"/opsx:explore <name> — continue thinking, or /opsx:propose <name> if ready to formalize"`
- **Mature (4+ decisions and ≤1 open question)**:
  `"/opsx:propose <name> — substantial thinking captured; ready to formalize the design (or /opsx:explore <name> to keep refining)"`

orbit-status parses the leading `/opsx:<verb> <name>` token in each case; the alternative path is surfaced in reason per D9's pattern.

**Rationale**: explore is fundamentally thinking-time, not workflow time — the recommendation shouldn't push toward propose prematurely. The decision-count threshold (2+ for "ready" / 4+ for "mature") matches the orbit `openspec-explore` skill's existing crystallization heuristic for bare mode (2+ substantive decisions). The "≤1 open question" condition gates "mature" because lots of open questions means more thinking is still warranted; the user has to either resolve them or move them to design.md before formalizing.

Bare-mode explore does not emit per D5.

### D12: Propose-shaped commands recommend `/opsx:review`; continue's logic depends on artifact completeness

`/opsx:propose`, `/opsx:new`, `/opsx:ff` all produce the canonical artifact set (proposal.md, design.md, tasks.md, specs/) in one go; their recommendation is always:

```
next_recommended: "/opsx:review <name> — proposal artifacts ready; review before apply"
```

This honors [#9](https://github.com/las-sal/openspec-orbit/issues/9) (propose should recommend review, not apply).

`/opsx:continue` is the partial-artifact case (continuing artifact generation when prior session created only some). Its emit is artifact-completion-aware:

```
continue-<TS>.json
  IF artifacts_complete (proposal + design + tasks + specs/ all present):
    next_recommended: "/opsx:review <name> — all proposal artifacts now present"
  ELSE:
    next_recommended: "/opsx:continue <name> — <next missing artifact> still pending"
```

**Rationale**: per [#9](https://github.com/las-sal/openspec-orbit/issues/9), the canonical post-propose step is review (not apply). The propose-family variants (`/opsx:new`, `/opsx:continue`, `/opsx:ff`) are entry-point differences. (Originally framed as "same recommendation" for all; refined during external system review EW1 — `/opsx:new` and `/opsx:continue` are artifact-completion-aware rather than propose-shaped because they don't produce the canonical artifact set in one invocation. Only `/opsx:propose` and `/opsx:ff` are propose-shaped. `/opsx:continue` defers to upstream `isComplete`; `/opsx:new` typically recommends `/opsx:continue` after scaffold-only invocation.)

### D13: `/opsx:verify` fail-mode recommendations (formalization of earlier conversation)

Per D6, verify's behavior is unchanged from upstream; this rule lives in the emit-layer that wraps it.

```
verify-<TS>.json (refines D9 which covers the pass case)

  IF verdict == "fail" AND tasks_remaining > 0:                  # mode ① incomplete
    next_recommended: "/opsx:apply <name> — N tasks remain unchecked;
                       complete implementation before re-verifying"

  ELIF verdict == "fail" AND openspec_validate_failed:            # mode ③ spec validation
    next_recommended: "Fix artifact validation errors: <validator message verbatim>"
    # command/args stay null; this is a non-orbit-command action

  ELIF verdict == "fail":                                          # mode ② impl-vs-spec gap
    next_recommended: "/opsx:review --as system <name> —
                       N spec scenarios fail against implementation;
                       system review will surface findings in its
                       scorecard for you to walk each as code fix
                       or spec revision"

  ELIF verdict == "warn":                                          # passes with warnings
    next_recommended: "/opsx:review --as system <name> — verification
                       passed with N warnings; system review recommended"
```

**Rationale**: matches D6's principle (verify stays upstream; emit-layer classifies the output). Mode ② recommends bare `/opsx:review --as system <name>` (the original D13 sketch had `--mark` to drop markers for `/opsx:address-reviews`, but EW3 during the external system review caught that `--mark` is proposal-mode only and silently ignored in system mode per `orbit-review/spec.md`'s `Requirement: --mark flag is proposal-mode only`; system-mode marker writing is v2 work tracked at the relevant follow-up). User walks each system-review finding manually to decide code-vs-spec. Mode ③ surfaces the validator's own message verbatim because the fix path is a direct artifact edit, not an orbit command.

### D14: `/opsx:review-external` T0 recommendation is multi-step prose

At T0 (prompt packaged, findings not yet returned), the run-summary JSON's `next_recommended` is multi-step prose describing the user action plus the follow-up command:

```
review-external-<TS>.json (T0)
  next_recommended: "Paste openspec/changes/<name>/.orbit-runs/external-prompt-<mode>-<TS>.md
                     into the target AI, save the response as
                     openspec/changes/<name>/.orbit-runs/external-<mode>-<TS>.md,
                     then /opsx:address-reviews <name> --from-file <path>"
```

orbit-status best-effort parses for a leading `/opsx:<verb>`; this string leads with "Paste..." so the parse fails — `command` and `args` stay null, and the full verbatim string is surfaced in `reason`. That's per D1 (consumer parse) and matches orbit-status's existing handling for prose recommendations (see `orbit-status-recommendation/spec.md:7`).

**Rationale**: review-external's T0 marks a state that requires a USER action (the external paste) before the next orbit command. There's no single `/opsx:<verb>` that captures the workflow at T0; prose accurately reflects the multi-step reality. The lack of structured parse is correct here — when the user sees this in orbit-status's `recommended_next.reason`, they read the multi-step instructions and act accordingly.

T1 (findings returned, address-reviews invoked) is the existing `/opsx:address-reviews` emit and isn't affected by this decision.

### D10: audit-drift standalone on clean findings defers to prior workflow recommendation

When standalone `/opsx:audit-drift` runs mid-flow and produces zero findings, the run-summary JSON's `next_recommended` is **copied from the most recent prior `.orbit-runs/*.json` for the same change** (excluding the just-written audit-drift JSON itself). The `final_assessment` notes "drift check clean; deferring to prior workflow state."

When standalone audit-drift produces findings:
```
next_recommended: "/opsx:address-reviews <name> --from-file <this-json> —
                   <N> drift(s) detected; resolve before next workflow step"
```

(Once [#4](https://github.com/las-sal/openspec-orbit/issues/4)'s superseder [#10](https://github.com/las-sal/openspec-orbit/issues/10) lands and `/opsx:address-reviews` auto-discovers internal JSONs, the `--from-file` flag becomes optional.)

**Rationale**: audit-drift on clean is informational — it shouldn't reset the workflow's narrative; it should reaffirm it. If the user was mid-apply and ran a sanity drift check, the workflow recommendation should still be "keep applying," not "archive now." Deferring to the prior recommendation preserves continuity. The inline audit-drift sweep during `/opsx:archive` is unaffected by this decision — it stays captured in `archive-<TS>.json` per the existing convention (and gets the per-finding fork from [#17](https://github.com/las-sal/openspec-orbit/issues/17)).

### D15: orbit-status tier-2 cleanup is a follow-up, not part of #8

orbit's emit work (#8) ships first; orbit-status's tier-2 synthesis ruleset becomes redundant the moment workflow-command JSONs start appearing but stays in place as a backstop. orbit-status's NEXT change after #8 lands deletes the tier-2 ruleset and simplifies to `tier-1 + tier-3` only.

**Rationale**: cross-repo coordination in a single change/PR complicates review and testing. orbit's emit-first sequencing is risk-free for orbit-status — tier-1 already reads `.orbit-runs/*.json` via the verbatim-string + best-effort-parse pattern (per `orbit-status-recommendation/spec.md:7`); new sources (explore/propose/apply/verify/etc. JSONs) feed into the same reader without code change. Dead-code tier-2 is harmless until orbit-status formally retires it.

**Coordination invariant**: orbit-status's tier-1 schema expectations are the load-bearing constraint on #8's JSON schema. As long as #8 emits the spine (`command`, `timestamp`, `change`, `final_assessment`, `next_recommended`) per D1 + D4, orbit-status's tier-1 reader works unchanged. The `kind` field added in D3 is consumed only by future tier-1 enhancements (if any); existing tier-1 ignores it.

**Verification status (completed during exploration)**: code-level scan of `/Users/sal/code/orbit-status/.claude/skills/openspec-status/bin/opsx-status` confirmed tier-2 is purely synthesis-output. `source_label` is set in `build_change_record` (lines 721, 728, 741, 758, 772, 778, 791, 797, 806) and emitted as a JSON field at line 817 — never read as a branch condition elsewhere in the code. No tests assert "if tier-2 fired, then X happens" because no such branch exists. When #8 lands, tier-2 becomes dead code that can be deleted without ripple effects. orbit-status's follow-up change can confidently `rm` the tier-2 logic and simplify to `tier-1 + tier-3` only.

## Considered & out

- **`/opsx:sync-specs`** — excluded from emit scope. Deprecated upstream (absorbed by `openspec archive` in v1.3.1); slated for removal by [#6](https://github.com/las-sal/openspec-orbit/issues/6). Designing emit semantics would be wasted work.
- **`/opsx:bulk-archive`** — wrapper around `/opsx:archive`. Each inner archive emits its own JSON; outer wrapper adds no additional "what's next" signal.
- **`/opsx:onboard`** — meta command (guided walkthrough). Doesn't transition a change's state; no useful `next_recommended`. Separately, the upstream onboard is slated for orbit-additions extension via [#23](https://github.com/las-sal/openspec-orbit/issues/23).
- **Structured `next_recommended` at producer side** — considered (object with `command`, `args`, `reason`); rejected per D1 in favor of string + consumer parse.
- **Schema versioning field** — skipped for v1. The `command` field implicitly versions; explicit `schema_version` adds overhead without current consumer demand.

## References

- Tracking issue: [#8](https://github.com/las-sal/openspec-orbit/issues/8) — Workflow commands should emit run-summary JSONs
- Dependency: [#6](https://github.com/las-sal/openspec-orbit/issues/6) — Overlay drop bundled upstream files (must sequence first or in parallel; defines the final command surface)
- Adjacent: [#9](https://github.com/las-sal/openspec-orbit/issues/9) — propose should recommend `/opsx:review` next (sub-requirement of D4's recommendation logic)
- Downstream consumer: `orbit-status` repo, `openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/specs/orbit-status-recommendation/spec.md` — defines Tier-1/Tier-2/Tier-3 reader contract
- Editorial JSON shape: `openspec/changes/archive/2026-05-18-bootstrap-openspec-orbit/.orbit-runs/` and `~/code/orbit-status/openspec/changes/archive/2026-05-20-bootstrap-orbit-status-cli/.orbit-runs/`
