---
name: openspec-apply-change
description: "Implement tasks from an OpenSpec change. Use when the user wants to start implementing, continue implementation, or work through tasks."
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---
Implement tasks from an OpenSpec change.

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If ambiguous, run `openspec list --json` to get available changes and use the **AskUserQuestion tool** to let the user select

   Always announce: "Using change: <name>" and how to override (e.g., `/opsx:apply <other>`).

2. **Check status to understand the schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand:
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - Which artifact contains the tasks (typically "tasks" for spec-driven, check status for others)

3. **Get apply instructions**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   This returns:
   - `contextFiles`: artifact ID -> array of concrete file paths (varies by schema - could be proposal/specs/design/tasks or spec/tests/implementation/docs)
   - Progress (total, complete, remaining)
   - Task list with status
   - Dynamic instruction based on current state

   **Handle states:**
   - If `state: "blocked"` (missing artifacts): show message, suggest using openspec-continue-change
   - If `state: "all_done"`: congratulate, suggest archive
   - Otherwise: proceed to implementation

4. **Read context files**

   Read every file path listed under `contextFiles` from the apply instructions output.
   The files depend on the schema being used:
   - **spec-driven**: proposal, specs, design, tasks
   - Other schemas: follow the contextFiles from CLI output

5. **Show current progress**

   Display:
   - Schema being used
   - Progress: "N/M tasks complete"
   - Remaining tasks overview
   - Dynamic instruction from CLI

6. **Implement tasks (loop until done or blocked)**

   For each pending task:
   - Show which task is being worked on
   - Make the code changes required
   - Keep changes minimal and focused
   - Mark task complete in the tasks file: `- [ ]` → `- [x]`
   - Continue to next task

   **Pause if:**
   - Task is unclear → ask for clarification
   - Implementation reveals a design issue → suggest updating artifacts
   - Error or blocker encountered → report and wait for guidance
   - User interrupts

7. **On completion or pause, show status**

   Display:
   - Tasks completed this session
   - Overall progress: "N/M tasks complete"
   - If all done: suggest archive
   - If paused: explain why and wait for guidance

**Output During Implementation**

```
## Implementing: <change-name> (schema: <schema-name>)

Working on task 3/7: <task description>
[...implementation happening...]
✓ Task complete

Working on task 4/7: <task description>
[...implementation happening...]
✓ Task complete
```

**Output On Completion**

```
## Implementation Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 7/7 tasks complete ✓

### Completed This Session
- [x] Task 1
- [x] Task 2
...

All tasks complete! Ready to archive this change.
```

**Output On Pause (Issue Encountered)**

```
## Implementation Paused

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 4/7 tasks complete

### Issue Encountered
<description of the issue>

**Options:**
1. <option 1>
2. <option 2>
3. Other approach

What would you like to do?
```

**Guardrails**
- Keep going through tasks until done or blocked
- Always read context files before starting (from the apply instructions output)
- If task is ambiguous, pause and ask before implementing
- If implementation reveals issues, pause and suggest artifact updates
- Keep code changes minimal and scoped to each task
- Update task checkbox immediately after completing each task
- Pause on errors, blockers, or unclear requirements - don't guess
- Use contextFiles from CLI output, don't assume specific file names

**Fluid Workflow Integration**

This skill supports the "actions on a change" model:

- **Can be invoked anytime**: Before all artifacts are done (if tasks exist), after partial implementation, interleaved with other actions
- **Allows artifact updates**: If implementation reveals design issues, suggest updating artifacts - not phase-locked, work fluidly

---

# Orbit additions

## Three execution disciplines (apply throughout this command)

The three execution disciplines from `orbit-conventions` apply: read-before-reference, change completeness, pushback. See `openspec/specs/orbit-conventions/spec.md`.

## Run-summary emit (chunk-aware)

(Per `orbit-run-summary-emit` capability — openspec-orbit#8)

`/opsx:apply` is a chunked multi-turn command. Emit timing differs from one-shot commands per `orbit-run-summary-emit`'s `Apply per-chunk-end emission` requirement:

```
openspec/changes/<name>/.orbit-runs/apply-<TS>.json
```

Where `<TS>` is ISO-8601 UTC with hyphens. Create `.orbit-runs/` if it doesn't exist.

### Chunk detection (parse tasks.md preamble)

If the change's `tasks.md` opens with an HTML comment block declaring chunks, the apply emit fires at chunk boundaries. The preamble format (per the spec):

```
<!--
Implementation chunks:
  Chunk 1 (groups 1):    Foundation
  Chunk 2 (groups 2-3):  Workflow emits
  Chunk 3 (groups 4):    Apply behavior
-->
```

**Format constraints**:
- `(groups X[-Y])` supports a single group (`X`) or a contiguous range (`X-Y`). Non-contiguous group sets (`groups 6,8,9`) are NOT supported in v1.
- Chunks MAY span multiple groups. The emit fires when the LAST group in the chunk completes its tasks, not on every group boundary.
- **Malformed preamble handling** (graceful degradation): if the preamble comment block exists but cannot be parsed, log a warning to stderr (`"warning: tasks.md preamble at <path> could not be parsed as chunk declarations; falling back to no-chunking mode"`) and proceed under no-chunking-apply rules (single emit at session end with `chunk: null`).

### Emit rules

Three cases:

1. **Chunk completion** (rule 1): when the last task in chunk N is checked, write `apply-<TS>.json` with `chunk: "N of M"`, `chunk_complete: true`, `chunk_name: <name>`. `next_recommended` advances to next chunk (`/opsx:apply <name>`) or to `/opsx:verify <name>` on apply-complete (chunk N == M).

2. **Mid-chunk session pause** (rule 2): when the user pauses or hands off mid-chunk-N (per the conversation-boundary signals in `Emit timing semantics` — explicit "stop"/"pause" OR AI-initiated wrap), emit with `chunk: "N of M"`, `chunk_complete: false`, and `next_recommended: "/opsx:apply <name>"` to resume.

3. **No-chunking apply** (rule 3): when tasks.md has no chunk preamble, emit once at session end with `chunk: null`, `chunk_complete: true`, and `next_recommended` advancing to `/opsx:verify <name>` on apply-complete or `/opsx:apply <name>` if tasks remain.

### JSON shape

Per the universal spine in `orbit-conventions`'s `Internal-run JSON summary format` + per-command extensions:

```json
{
  "command": "apply",
  "timestamp": "<ISO-8601 UTC>",
  "change": "<name>",
  "final_assessment": "<narrative, e.g., 'Completed chunk 2 of 5 (inventory+parsing); 28 of 76 tasks done.'>",
  "next_recommended": "<per advancement rules below>",
  "kind": "workflow",
  "tasks_completed": <int — running total of checked tasks across all chunks>,
  "tasks_remaining": <int — running total of unchecked tasks (includes user-validation if present)>,
  "user_validation_remaining": <int — subset of tasks_remaining that are acknowledged user-validation; default 0>,
  "chunk": "<N of M>" | null,
  "chunk_name": "<from preamble>" | null,
  "chunk_complete": true | false,
  "tasks_completed_this_session": <int — delta since prior apply JSON, for forensic timeline>
}
```

### `next_recommended` advancement rules

- **Chunk N done, more chunks** (N < M): `/opsx:apply <name> — next chunk: <chunk N+1 name from preamble>`
- **Apply complete** (all chunks done OR no-chunking apply with all tasks done): `/opsx:verify <name>`
- **Mid-chunk pause**: `/opsx:apply <name>` (resume current chunk)

orbit-status's tier-1 reader parses the leading `/opsx:<verb> <name>` token into `command`/`args`.

### Why chunk-aware (forensic + resumability)

Per the design rationale (openspec-orbit#22 + orbit-status retrospective): per-chunk emit enables forensic timeline reconstruction ("which chunk introduced this regression?") and provides clean resume boundaries between sessions. Without chunk-aware emit, post-apply regression bisection has to span the full task universe; with chunk-aware emit, blast radius is bounded to a single chunk's task set (typically 10-20× reduction).