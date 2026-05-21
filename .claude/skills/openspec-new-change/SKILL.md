---
name: openspec-new-change
description: "Start a new OpenSpec change using the experimental artifact workflow. Use when the user wants to create a new feature, fix, or modification with a structured step-by-step approach."
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---
Start a new change using the experimental artifact-driven approach.

**Input**: The user's request should include a change name (kebab-case) OR a description of what they want to build.

**Steps**

1. **If no clear input provided, ask what they want to build**

   Use the **AskUserQuestion tool** (open-ended, no preset options) to ask:
   > "What change do you want to work on? Describe what you want to build or fix."

   From their description, derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).

   **IMPORTANT**: Do NOT proceed without understanding what the user wants to build.

2. **Determine the workflow schema**

   Use the default schema (omit `--schema`) unless the user explicitly requests a different workflow.

   **Use a different schema only if the user mentions:**
   - A specific schema name → use `--schema <name>`
   - "show workflows" or "what workflows" → run `openspec schemas --json` and let them choose

   **Otherwise**: Omit `--schema` to use the default.

3. **Create the change directory**
   ```bash
   openspec new change "<name>"
   ```
   Add `--schema <name>` only if the user requested a specific workflow.
   This creates a scaffolded change at `openspec/changes/<name>/` with the selected schema.

4. **Show the artifact status**
   ```bash
   openspec status --change "<name>"
   ```
   This shows which artifacts need to be created and which are ready (dependencies satisfied).

5. **Get instructions for the first artifact**
   The first artifact depends on the schema (e.g., `proposal` for spec-driven).
   Check the status output to find the first artifact with status "ready".
   ```bash
   openspec instructions <first-artifact-id> --change "<name>"
   ```
   This outputs the template and context for creating the first artifact.

6. **STOP and wait for user direction**

**Output**

After completing the steps, summarize:
- Change name and location
- Schema/workflow being used and its artifact sequence
- Current status (0/N artifacts complete)
- The template for the first artifact
- Prompt: "Ready to create the first artifact? Just describe what this change is about and I'll draft it, or ask me to continue."

**Guardrails**
- Do NOT create any artifacts yet - just show the instructions
- Do NOT advance beyond showing the first artifact template
- If the name is invalid (not kebab-case), ask for a valid name
- If a change with that name already exists, suggest continuing that change instead
- Pass --schema if using a non-default workflow

---

# Orbit additions

## Three execution disciplines (apply throughout this command)

The three execution disciplines from `orbit-conventions` apply: read-before-reference, change completeness, pushback. See `openspec/specs/orbit-conventions/spec.md`.

## Run-summary emit (one-shot at command completion)

(Per `orbit-run-summary-emit` capability — openspec-orbit#8)

`/opsx:new` is a one-shot command per `orbit-run-summary-emit`'s `Emit timing semantics` requirement (emit ONCE on natural command completion). After the change is scaffolded (and any initial artifacts created), write:

```
openspec/changes/<name>/.orbit-runs/new-<TS>.json
```

Where `<TS>` is ISO-8601 UTC with hyphens. Create `.orbit-runs/` if it doesn't exist. The filename prefix is `new-` (NOT `propose-`) to preserve entry-point provenance per the per-variant filename rule.

### JSON shape

Per the universal spine in `orbit-conventions`'s `Internal-run JSON summary format` + per-command extensions:

```json
{
  "command": "new",
  "timestamp": "<ISO-8601 UTC>",
  "change": "<name>",
  "final_assessment": "<narrative, e.g., 'Scaffolded new change <name>; created proposal.md as the first artifact.'>",
  "next_recommended": "/opsx:continue <name> — continue generating artifacts (next: design/specs/tasks per schema)",
  "kind": "workflow",
  "artifacts_created": ["proposal"],
  "delta_count": 0,
  "from": "scratch"
}
```

(`/opsx:new` typically creates only the initial scaffolded proposal; subsequent artifact generation happens via `/opsx:continue` or `/opsx:ff`. If the user provides enough context for a more complete initial scaffold, `artifacts_created` may include additional artifacts and `next_recommended` may advance to `/opsx:review` directly.)