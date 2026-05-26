## ADDED Requirements

### Requirement: Review mode decision framework

The system SHALL codify the decision criteria for choosing among orbit's three review modes (in-context, `--fresh`, external). This is the normative-force complement to README's "Choosing a review mode" section, which documents the same framework as user-facing guidance. The framework exists because empirical evidence (notably `bootstrap-orbit-status-cli`'s 3-of-3 finding — in-context system review missed all 3 real implementation bugs that external review caught) demonstrates that mode choice materially affects review-finding completeness.

The three modes:

1. **In-context** (default `/opsx:review`) — the current AI session does the review against its accumulated context. Fast iteration; low cross-AI cost. Anchoring risk: AI may be biased toward confirming work it has visibility into.
2. **`--fresh` internal subagent** (`/opsx:review --fresh`) — spawns a fresh subagent within the same Claude process. Loses session-level anchoring; same model architecture so model-level biases remain. Middle-ground between in-context and external.
3. **External** (`/opsx:review-external`) — packages a self-contained prompt for a different AI session (different model + different chat + completely separate context). Maximum independence; cross-AI round-trip cost. Best for substantive changes + code-vs-spec verification.

#### Scenario: In-context default for first-pass review with low anchoring risk

- **WHEN** the user invokes `/opsx:review` for the first review pass of a change AND no design ambiguity has been flagged AND the change is small (e.g., docs-only, single-file, or otherwise low-stakes)
- **THEN** in-context (the default) is the appropriate mode; session-level anchoring is an asset (the AI knows the context) rather than a liability; cross-AI cost is unnecessary. The framework does NOT require `--fresh` or external for low-stakes first passes.

#### Scenario: `--fresh` for anchoring-break in multi-iteration cases

- **WHEN** the same in-context AI session has reviewed an artifact 2+ times AND the user wants to verify convergence without paying cross-AI round-trip cost
- **THEN** `/opsx:review --fresh` is the appropriate mode. The fresh subagent loses the iter-1 + iter-2 in-session anchoring while preserving same-model semantics; it confirms that the multi-iteration in-context reviews didn't accumulate blind spots. Use `--fresh` as a lighter alternative to external when same-model coverage is acceptable.

#### Scenario: External for system-mode + substantive cross-AI verification

- **WHEN** running system-mode review (verifying code-vs-spec coherence where the AI may have authored the code in a recent session — high anchoring risk per `bootstrap-orbit-status-cli` evidence) OR the change is substantive enough that cross-AI cross-check pays for itself (e.g., breaking changes, security-sensitive code, multi-capability changes)
- **THEN** `/opsx:review-external` is the appropriate mode. The cross-AI round-trip cost is justified by the independence — a different model + different session has no anchoring to the code the original AI wrote. Per orbit-review's `Final assessment phrasings depend on mode` requirement, system-mode review's "no CRITICAL" final-assessment recommends external by default for this reason; the recommendation is advisory, not a gate.

#### Scenario: Framework is documented in README + spec; drift between the two is caught

- **WHEN** the README's "Choosing a review mode" section drifts from this requirement's normative criteria (e.g., README claims `--fresh` is appropriate for X but the spec scenarios don't reflect that)
- **THEN** `/opsx:audit-drift` Category 3 (cross-doc consistency) SHALL surface the drift on next run; orbit's docs and spec stay aligned via the audit-drift discipline rather than ad-hoc reconciliation.

#### Scenario: Framework guidance does NOT enforce cycle patterns

- **WHEN** a change is "substantial" by the README's guidance (would benefit from external review per the framework) BUT the user chooses to archive without running external
- **THEN** orbit's archive flow proceeds; the framework is guidance, NOT a normative cycle-pattern enforcement. The decision to NOT add cycle-pattern enforcement is intentional — orbit's role is to surface the recommendation (per orbit-review's iteration-aware final-assessment), not to refuse archive of changes that skip the recommendation.
