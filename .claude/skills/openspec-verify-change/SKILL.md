---
name: openspec-verify-change
description: "Verify implementation matches change artifacts. Use when the user wants to validate that implementation is complete, correct, and coherent before archiving."
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---
Verify that an implementation matches the change artifacts (specs, tasks, design).

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use the **AskUserQuestion tool** to let the user select.

   Show changes that have implementation tasks (tasks artifact exists).
   Include the schema used for each change if available.
   Mark changes with incomplete tasks as "(In Progress)".

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Check status to understand the schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand:
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - Which artifacts exist for this change

3. **Get the change directory and load artifacts**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   This returns the change directory and `contextFiles` (artifact ID -> array of concrete file paths). Read all available artifacts from `contextFiles`.

4. **Initialize verification report structure**

   Create a report structure with three dimensions:
   - **Completeness**: Track tasks and spec coverage
   - **Correctness**: Track requirement implementation and scenario coverage
   - **Coherence**: Track design adherence and pattern consistency

   Each dimension can have CRITICAL, WARNING, or SUGGESTION issues.

5. **Verify Completeness**

   **Task Completion**:
   - If `contextFiles.tasks` exists, read every file path in it
   - Parse checkboxes: `- [ ]` (incomplete) vs `- [x]` (complete)
   - Count complete vs total tasks
   - If incomplete tasks exist:
     - Add CRITICAL issue for each incomplete task
     - Recommendation: "Complete task: <description>" or "Mark as done if already implemented"

   **Spec Coverage**:
   - If delta specs exist in `openspec/changes/<name>/specs/`:
     - Extract all requirements (marked with "### Requirement:")
     - For each requirement:
       - Search codebase for keywords related to the requirement
       - Assess if implementation likely exists
     - If requirements appear unimplemented:
       - Add CRITICAL issue: "Requirement not found: <requirement name>"
       - Recommendation: "Implement requirement X: <description>"

6. **Verify Correctness**

   **Requirement Implementation Mapping**:
   - For each requirement from delta specs:
     - Search codebase for implementation evidence
     - If found, note file paths and line ranges
     - Assess if implementation matches requirement intent
     - If divergence detected:
       - Add WARNING: "Implementation may diverge from spec: <details>"
       - Recommendation: "Review <file>:<lines> against requirement X"

   **Scenario Coverage**:
   - For each scenario in delta specs (marked with "#### Scenario:"):
     - Check if conditions are handled in code
     - Check if tests exist covering the scenario
     - If scenario appears uncovered:
       - Add WARNING: "Scenario not covered: <scenario name>"
       - Recommendation: "Add test or implementation for scenario: <description>"

7. **Verify Coherence**

   **Design Adherence**:
   - If `contextFiles.design` exists:
     - Extract key decisions (look for sections like "Decision:", "Approach:", "Architecture:")
     - Verify implementation follows those decisions
     - If contradiction detected:
       - Add WARNING: "Design decision not followed: <decision>"
       - Recommendation: "Update implementation or revise design.md to match reality"
   - If no design.md: Skip design adherence check, note "No design.md to verify against"

   **Code Pattern Consistency**:
   - Review new code for consistency with project patterns
   - Check file naming, directory structure, coding style
   - If significant deviations found:
     - Add SUGGESTION: "Code pattern deviation: <details>"
     - Recommendation: "Consider following project pattern: <example>"

8. **Generate Verification Report**

   **Summary Scorecard**:
   ```
   ## Verification Report: <change-name>

   ### Summary
   | Dimension    | Status           |
   |--------------|------------------|
   | Completeness | X/Y tasks, N reqs|
   | Correctness  | M/N reqs covered |
   | Coherence    | Followed/Issues  |
   ```

   **Issues by Priority**:

   1. **CRITICAL** (Must fix before archive):
      - Incomplete tasks
      - Missing requirement implementations
      - Each with specific, actionable recommendation

   2. **WARNING** (Should fix):
      - Spec/design divergences
      - Missing scenario coverage
      - Each with specific recommendation

   3. **SUGGESTION** (Nice to fix):
      - Pattern inconsistencies
      - Minor improvements
      - Each with specific recommendation

   **Final Assessment**:
   - If CRITICAL issues: "X critical issue(s) found. Fix before archiving."
   - If only warnings: "No critical issues. Y warning(s) to consider. Ready for archive (with noted improvements)."
   - If all clear: "All checks passed. Ready for archive."

**Verification Heuristics**

- **Completeness**: Focus on objective checklist items (checkboxes, requirements list)
- **Correctness**: Use keyword search, file path analysis, reasonable inference - don't require perfect certainty
- **Coherence**: Look for glaring inconsistencies, don't nitpick style
- **False Positives**: When uncertain, prefer SUGGESTION over WARNING, WARNING over CRITICAL
- **Actionability**: Every issue must have a specific recommendation with file/line references where applicable

**Graceful Degradation**

- If only tasks.md exists: verify task completion only, skip spec/design checks
- If tasks + specs exist: verify completeness and correctness, skip design
- If full artifacts: verify all three dimensions
- Always note which checks were skipped and why

**Output Format**

Use clear markdown with:
- Table for summary scorecard
- Grouped lists for issues (CRITICAL/WARNING/SUGGESTION)
- Code references in format: `file.ts:123`
- Specific, actionable recommendations
- No vague suggestions like "consider reviewing"

---

# Orbit additions

## Three execution disciplines (apply throughout this command)

The three execution disciplines from `orbit-conventions` apply: read-before-reference, change completeness, pushback. See `openspec/specs/orbit-conventions/spec.md`.

## Run-summary emit (one-shot at command completion)

(Per `orbit-run-summary-emit` capability — openspec-orbit#8)

`/opsx:verify` is a one-shot command per `orbit-run-summary-emit`'s `Emit timing semantics` requirement (emit ONCE on natural command completion). After verify-change reports its outcome, write:

```
openspec/changes/<name>/.orbit-runs/verify-<TS>.json
```

Where `<TS>` is ISO-8601 UTC with hyphens. Create `.orbit-runs/` if it doesn't exist.

### What verify does (unchanged from upstream)

`/opsx:verify` runs upstream `verify-change` and reports pass/fail/warn with findings. This emit-layer is a thin observability wrapper — it does NOT add marker-dropping, spec-edit shortcuts, or any other behavior that changes what verify itself does. The recommendation-classification logic below lives in the emit-layer, NOT in verify.

### JSON shape

Per the universal spine in `orbit-conventions`'s `Internal-run JSON summary format` + per-command extensions:

```json
{
  "command": "verify",
  "timestamp": "<ISO-8601 UTC>",
  "change": "<name>",
  "final_assessment": "<narrative of verify outcome, e.g., 'Verification clean: 79/79 tasks done; 0 spec gaps.'>",
  "next_recommended": "<per classification rules below>",
  "kind": "workflow",
  "verdict": "pass" | "fail" | "warn",
  "findings_count": <int — 0 on clean pass>,
  "findings_summary": { "critical": 0, "warning": 0, "suggestion": 0 }
}
```

### Verdict classification → `next_recommended`

Inspect verify-change's output and classify into one of these states, then compose `next_recommended` accordingly. **Precedence when multiple fail signals fire simultaneously** (highest first): mode ③ > mode ① > mode ② > warn. This directs the user to the root cause first (fix spec → complete impl → resolve gaps).

**Verdict: pass** — verify-change reports no failures.

```
next_recommended: "/opsx:review --as system <name> — verification clean; formal pre-archive review recommended (or /opsx:archive <name> if you're skipping the editorial pass)"
```

orbit-status's tier-1 parses the leading `/opsx:review --as system <name>` token into `command`/`args`. The `/opsx:archive <name>` alternative lives in the reason text for users who want to skip the formal review.

**Verdict: fail — Mode ① (tasks-incomplete)** — tasks.md has unchecked items.

```
next_recommended: "/opsx:apply <name> — N tasks remain unchecked; complete implementation before re-verifying"
```

**Verdict: fail — Mode ② (impl-vs-spec gap)** — tasks all checked but spec scenarios fail against the implementation.

```
next_recommended: "/opsx:review --as system <name> — N spec scenarios fail against implementation; system review will surface findings in its scorecard for you to walk each as code fix or spec revision"
```

The recommendation does NOT use `/opsx:review --as system --mark` because `--mark` is proposal-mode only and silently ignored in system mode (per `orbit-review/spec.md`'s `Requirement: --mark flag is proposal-mode only`). System-mode marker writing is v2 work tracked elsewhere; until then, the user manually walks each system-review finding to decide whether it's a code bug (fix via `/opsx:apply` or direct edit) or a spec revision (edit specs directly).

**Verdict: fail — Mode ③ (openspec-validate failure)** — `openspec validate` itself errors (e.g., malformed spec frontmatter).

```
next_recommended: "Fix artifact validation errors: <validator message verbatim>"
```

No leading orbit command in this case; the reason field carries the validator output. orbit-status's tier-1 parse finds no leading slash command and preserves the full text in `reason`.

**Verdict: warn** — verify passes but with non-blocking findings.

```
next_recommended: "/opsx:review --as system <name> — verification passed with N warnings; system review recommended"
```

### Partial / aborted verify runs (out of scope)

If verify-change reports incomplete output (timed out, transient validator failure), the emit-layer SHALL emit `next_recommended: "Re-run /opsx:verify <name> — prior verify run incomplete (see verify-change output for details)"`. Otherwise these are upstream verify-change concerns; the emit-layer emits whatever verify-change reports.