# External Review: address-reviews-accepts-internal-json (iteration 1, system mode)

**Reviewer**: GPT-5 Codex
**Date**: 2026-05-26

## CRITICAL

### Audit-drift JSON is rejected even though baseline emits it as address-reviews input
**File**: openspec/changes/address-reviews-accepts-internal-json/specs/orbit-address-reviews/spec.md:24
**Description**: The new requirement says any JSON `command` other than `"review"` (including `"audit-drift"`) must error, but the archived `orbit-run-summary-emit` baseline currently requires standalone audit-drift findings to set `next_recommended` to `/opsx:address-reviews ... --from-file <this-json>` (`openspec/specs/orbit-run-summary-emit/spec.md:403` and `openspec/specs/orbit-run-summary-emit/spec.md:409`), and the audit-drift skill/schema repeat that contract. After this change, the product still emits a recommended command for audit-drift findings that the address-reviews parser is required to reject. Either accept `command: "audit-drift"` JSON in this change (mapping `category` as pass/provenance and walking the same lifecycle), or update/delta the audit-drift/run-summary recommendation contract so it no longer points users at an unsupported `--from-file` path.

## WARNING

### Address-reviews run-summary schema cannot represent internal-review marker provenance
**File**: .claude/skills/openspec-address-reviews/references/run-summary-schema.md:50
**Description**: The new internal parser contract creates virtual markers with `source: "internal-review"`, and this change's own `address-reviews-2026-05-26T23-37-11Z.json` already records `marker_source: "internal-review"`, but the address-reviews run-summary schema still restricts `marker_source` to `"inline" | "external"` and the field note only describes those two cases. Downstream consumers following this schema can reject or misclassify JSON-sourced resolutions. Add `internal-review` to the enum and field notes, and clarify that `external_reviewer` is only populated for external-markdown inputs.

### Review and onboarding surfaces still describe `--from-file` as external-only
**File**: .claude/skills/openspec-review/SKILL.md:13
**Description**: The review skill is the producer of `review-<mode>-*.json`, but it still says resolution flows through address-reviews for inline markers or "`--from-file` external findings", omitting the new internal-review JSON route. The onboarding skill and command mirror repeat the same external-only description in their identity text and command table. This hides the new lifecycle from users and future AIs at the exact surfaces that explain how review findings are resolved. Update the review skill plus onboard skill/command descriptions to say `--from-file` accepts internal review JSON as well as external-review markdown.

## SUGGESTION

### Core lifecycle removal step still names only external virtual markers
**File**: .claude/skills/openspec-address-reviews/SKILL.md:122
**Description**: The new ingest section correctly says marker removal is a no-op for virtual markers regardless of provenance, but the concrete Step 3d removal list still says only "**External (virtual marker)**". A reader implementing from the lifecycle steps can miss JSON virtual markers or treat them as a separate case. Rename this bullet to "Virtual marker (external markdown or internal-review JSON)" and update the triage/source-label examples to show `internal-review` alongside `external`.

### Internal-findings reference makes required run-summary spine fields look optional
**File**: .claude/skills/openspec-address-reviews/references/internal-findings-format.md:34
**Description**: The reference says `final_assessment`, `next_recommended`, `kind`, and other top-level fields "MAY be present", but the canonical review run-summary schema inherits those universal-spine fields as required. Because this same doc says the format MUST match `/opsx:review` output, a fresh implementer could confuse "ignored by the address-reviews parser" with "optional in a valid review JSON file". State that real review JSON MUST include the universal spine but address-reviews ignores those fields, or include the full spine in the expected-format example.

## Notes

Validation passed: `openspec validate address-reviews-accepts-internal-json --strict` and `openspec validate --all --strict` both returned clean.
