# Sketch: `/opsx:propose` modifications

> **Status**: design sketch. Not implementation. Captured from explore-mode conversation 2026-05-17.
> **Aligns to**: orbit guiding principle 1 (openspec coherence — preserves upstream behavior; adds explore-consumption flow on top).

## What stays the same

Upstream's `/opsx:propose` generates the standard openspec change artifacts in one step:
- `proposal.md`
- `design.md`
- `specs/<capability>/spec.md` (delta specs)
- `tasks.md`

Output goes into `openspec/changes/<name>/`. The command takes a change name and (usually) a description.

This base behavior is preserved entirely. Orbit's additions activate only when an explore staging directory exists.

## What changes

Two additions:

1. **Consume `explore.md`** — if `openspec/explore/<name>/explore.md` exists, propose reads it as the authoritative seed for the generated artifacts.
2. **Promote the staging directory** — propose **moves** `openspec/explore/<name>/` → `openspec/changes/<name>/`, then writes the generated artifacts into the change dir alongside the moved `explore.md`.

## Two modes

```
Mode 1:  CONSUME MODE — staging dir exists
─────────────────────────────────────────────────
  openspec/explore/<name>/ exists with explore.md.

  /opsx:propose <name>
       │
       ├─ Read openspec/explore/<name>/explore.md
       ├─ Read sibling files (sketches, draft conventions)
       ├─ Handle Open questions (prompt → resolve / defer / abandon)
       ├─ Generate proposal.md / design.md / specs/ / tasks.md
       │  in a temp working area
       ├─ Move openspec/explore/<name>/ → openspec/changes/<name>/
       └─ Write generated artifacts into openspec/changes/<name>/

  Final state:
    openspec/changes/<name>/
      ├── explore.md            (preserved from explore)
      ├── sketches/             (preserved from explore)
      ├── proposal.md           (generated)
      ├── design.md             (generated)
      ├── specs/<cap>/spec.md   (generated)
      └── tasks.md              (generated)


Mode 2:  STANDALONE MODE — no staging dir
─────────────────────────────────────────────────
  openspec/explore/<name>/ does not exist.

  /opsx:propose <name> [<description>]
       │
       ├─ Same as upstream propose
       └─ Generate artifacts from description alone

  No move; no explore.md. Just the generated artifacts in
  openspec/changes/<name>/.
```

## Consume mode — section mapping

How each part of `explore.md` informs the generated artifacts:

| `explore.md` section | Feeds into |
|---|---|
| **Premise** | `proposal.md` motivation / "Why" section |
| **Decisions** | Spec delta requirements + `design.md` decisions + `tasks.md` task list |
| **Open questions** | Resolved before generating (preferred) OR carried into artifacts as `@review:` markers (deferred) |
| **Considered & out** | `design.md` "Alternatives considered" section |
| **References** | Contextual reads for artifact generation; cited where relevant |
| **Guiding principles** (if present) | Inform tone/constraints across all artifacts; not directly copied |

Sibling files in the explore dir:
- **`sketches/*.md`** — read for design context; can be cited in `design.md`. Persist alongside generated artifacts.
- **Other captures** (e.g., draft conventions that didn't get promoted to project root) — preserved as-is during the move.

## Handling Open questions

Open questions in `explore.md` are unresolved when propose runs. Three resolution paths, per question:

| Path | What happens |
|---|---|
| **Resolve now** | User answers in conversation; propose generates artifacts with the resolution baked in; the question is moved to a Decision (in the moved `explore.md`). |
| **Defer** | Question is converted to a `@review:` marker in the most relevant generated artifact (proposal/design/spec/tasks). `/opsx:address-reviews` walks it later. |
| **Abandon** | Question is moved to `Considered & out` with a brief rationale ("decided not to address in this change"). |

For each open question, propose prompts via `AskUserQuestion`:

> "Open question: <question text>. Resolve now / Defer (`@review:` marker) / Abandon?"

If many open questions exist, propose can group: "There are N open questions. Resolve them now, defer all to markers, or walk each?"

**Default**: defer. Pushes the work to address-reviews, where it composes with the broader review loop. Matches guiding principle 2 (capture early, resolve in the right place).

## The move operation

```
BEFORE                                    AFTER
──────                                    ─────
openspec/                                 openspec/
├── explore/                               ├── explore/                  (no <name>)
│   └── <name>/                            ├── changes/
│       ├── explore.md                     │   └── <name>/
│       ├── sketches/                      │       ├── explore.md          (moved)
│       └── conventions/                   │       ├── sketches/           (moved)
├── changes/                               │       ├── conventions/        (moved)
└── …                                      │       ├── proposal.md         (new)
                                           │       ├── design.md           (new)
                                           │       ├── specs/<cap>/spec.md (new)
                                           │       └── tasks.md            (new)
                                           └── …
```

**Why move, not copy**: a single canonical location for the work. Leaving a leftover `openspec/explore/<name>/` would be confusing (two dirs for the same change). The exploration *becomes* the change.

**Files preserved during move**: everything in the explore directory. The explore record persists as historical context — important because `Considered & out` may be referenced again to avoid rediscovering rejected options.

## Standalone mode (no explore.md)

If `openspec/explore/<name>/` doesn't exist, propose behaves exactly like upstream — takes a description, generates artifacts, writes to `openspec/changes/<name>/`. No move, no explore.md.

This is the "I know what I want, skip the exploration phase" path. Still valid; no orbit-specific logic activates.

## Edge cases

| Case | Handling |
|---|---|
| Both `openspec/explore/<name>/` and `openspec/changes/<name>/` exist | Halt. Report conflict. Ask user: continue from change dir (discard explore staging)? regenerate from explore (overwrite change)? abort? |
| `openspec/changes/<name>/` exists but `openspec/explore/<name>/` doesn't | Halt. Change already exists; suggest `/opsx:continue` or a different name. |
| `explore.md` is malformed (e.g., missing required sections) | Halt. Report missing/malformed section. User edits explore.md and re-runs. |
| `explore.md` has only a Premise (no Decisions yet) | Warn but proceed. The generated artifacts will be sparse; user can iterate. |
| `explore.md` has many unresolved Open questions | Prompt user with the bulk-handling option (resolve all / defer all / walk each). |

## Heuristics

- **Don't paraphrase decisions** — `explore.md` Decisions carry specific wording the user chose. Generated artifacts should preserve that wording where possible. Reframe only when artifact-shape demands it (e.g., a Decision becomes a spec requirement and needs requirement-spec formatting).
- **Cite the explore source** — generated `design.md` should reference `explore.md` for context, especially the Considered & out section. Anyone reading the change later should be able to trace decisions back to their origin.
- **Preserve sibling files** — sketches, draft conventions, anything else in the explore dir. They might inform downstream work or future audits.
- **Be conservative with @review: markers** — only escalate open questions that genuinely can't be resolved now. If the user can answer in 30 seconds, prompt; don't defer.
- **Don't auto-resolve** — never silently pick a direction for an open question. Always ask.

## Open design questions

1. **What if explore.md was authored by hand (not via `/opsx:explore`)?** Probably fine — propose just reads it. But manual authoring may not follow the strict 5-section convention. Lean: be tolerant, parse what's there, warn on missing sections.
2. **Should propose validate `explore.md` structure before consuming?** Lightly — check for the 5 expected sections (Premise / Decisions / Open questions / Considered & out / References). Missing sections trigger warnings, not errors. Malformed YAML/markdown that prevents reading triggers errors.
3. **Bulk-handle UX for many open questions** — if explore.md has 10+ open questions, walking each is tedious. Already proposed "resolve all / defer all / walk each" — but "defer all" might lose nuance (some questions are better resolved than deferred). Worth keeping the bulk option but encouraging user to walk individually.
4. **Naming inference** — should `/opsx:propose` (no args) try to infer the name from recent `/opsx:explore` activity? Probably yes for ergonomics; otherwise prompt.
5. **What about explore.md edits made *after* propose generates?** Once moved into `openspec/changes/<name>/`, explore.md is historical record. Editing it later is allowed but doesn't trigger regeneration. If user wants to re-propose from updated explore.md, they'd need to delete the change dir first (or use a future `/opsx:re-propose` command).

## Composition with related commands

```
                /opsx:explore [<name>]
                        │
                        ▼
                openspec/explore/<name>/
                  ├── explore.md
                  ├── sketches/
                  └── conventions/
                        │
                        ▼
                /opsx:propose <name>
                        │
                  ┌─────┴─────┐
                  │           │
                  ▼           ▼
            consume mode  standalone mode
                  │           │
                  ▼           ▼
         move explore →  generate
         changes;        artifacts
         generate        from
         artifacts       description
                  │           │
                  └─────┬─────┘
                        ▼
                openspec/changes/<name>/
                  ├── explore.md (consume only)
                  ├── sketches/   (consume only)
                  ├── proposal.md
                  ├── design.md
                  ├── specs/
                  └── tasks.md
                        │
                        ▼
                /opsx:review <name> --as proposal
                        │
                        ▼
                /opsx:address-reviews [<scope>]
                        │
                        ▼
                /opsx:apply <name>
                        │
                        ▼
                code
                        │
                        ▼
                /opsx:review <name> --as system
                        │
                        ▼
                /opsx:address-reviews [<scope>]
                        │
                        ▼
                /opsx:archive <name>
                        │
                        ▼
                /opsx:audit-drift (auto-invoked pre-archive)
```

The full v1 orbit loop, with propose as the bridge from explore → change. explore.md and sibling files persist throughout, becoming permanent historical record once archived.

## What this means for the SKILL.md modifications

Upstream's `openspec-propose` SKILL.md describes proposal generation from a description. Orbit modifies it with:

- A "consume mode" detection step: check for `openspec/explore/<name>/explore.md` first.
- The explore.md section-to-artifact mapping (Premise → proposal motivation, etc.).
- The Open questions handling flow (resolve / defer / abandon).
- The move operation (staging → changes).
- Edge case handling (conflicts, malformed input).

Standalone-mode behavior remains identical to upstream. Same shape for the slash-command body (`.claude/commands/opsx/propose.md`) — append orbit-specific consume-mode behavior.
