<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):   Add references/internal-findings-format.md +
                       update SKILL.md + opsx/address-reviews.md
                       (parser logic prose + auto-detect + JSON path)
  Chunk 2 (group 2):   Worked example + format-error handling +
                       cross-reference update in external-findings-format.md
  Chunk 3 (group 3):   Validation + user-validation handoff

Total: 3 implementation chunks. No sandbox verification needed: this is
prose + parser-contract documentation (no CLI behavior change in the user-
invocable sense — the new behavior is internal to the address-reviews skill
prose). Sync-specs at archive applies the MODIFIED requirement to baseline.
-->

## 1. Reference doc + parser logic prose

- [x] 1.1 Create `.claude/skills/openspec-address-reviews/references/internal-findings-format.md` mirroring the structure of `external-findings-format.md` (5 sections: Expected file format / Parser contract / Malformed input handling / Tolerated variations / Worked example). Document: (a) the supported input shape (`review-<mode>-*.json` per the orbit-review run-summary-schema reference), (b) the parser contract mapping JSON fields → virtual-marker fields (`severity` → severity, `title` → title, `file` + `line` → file:line, `recommendation` → description, fixed `source: "internal-review"`), (c) the `command` field discriminator (only `"review"` accepted; other values rejected), (d) malformed input handling (JSON parse failure, missing `findings[]`, unsupported `command`), (e) tolerated variations (missing optional fields like `line` → file-only marker), (f) a concrete worked example showing a 2-finding ingest.

- [x] 1.2 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s `--from-file <path> ingest` section to describe the new auto-detect routing logic: leading-character sniff → markdown OR JSON parser → virtual markers. Add subsection or paragraph describing the JSON path specifically, citing `references/internal-findings-format.md` for the parser contract. Update the SKILL's existing markdown-only language to reflect that markdown is now ONE of two supported formats (not THE supported format). Specifically: the walk-section prose MUST document that fresh pushback applies to JSON-virtual markers even though the JSON's own `stale_suppressed[]` array already filtered stale findings at review time (per D-pushback-1; the same lifecycle discipline applies regardless of marker provenance).

- [x] 1.3 Update `.claude/commands/opsx/address-reviews.md` to mirror SKILL.md changes from 1.2. Per orbit-modified command-file convention, the body sections mirror the SKILL (excluding frontmatter); the auto-detect logic + new JSON path mention appear in both files.

- [x] 1.4 Lightly update `.claude/skills/openspec-address-reviews/references/external-findings-format.md` to add a one-line cross-reference at the top: "This is ONE of two supported `--from-file` formats; the other is internal review JSON — see `internal-findings-format.md`." No structural changes to the file beyond this cross-reference note.

- [x] 1.5 Update `README.md` in two sites: (a) the `--from-file` flag description (currently ~L422 in the `/opsx:address-reviews` command-reference block — `ingest external-review findings as virtual markers`) broadens to mention both supported formats; (b) the "External-review markdown findings format" sub-header (currently ~L769 in the `Run-summary JSON emission` section) gains a sibling note pointing at `internal-findings-format.md` for the JSON shape. Verify line numbers at edit time (README may have shifted since this task was written); the section anchors are stable even if line numbers move.

## 2. Worked example + format-error handling

- [x] 2.1 Add a JSON-ingest worked example to `.claude/skills/openspec-address-reviews/SKILL.md` showing a typical internal-review `--from-file` ingest (e.g., a `review-system-*.json` with 1 SUGGESTION finding being walked → trivial-fix-resolved → resolution log). Position adjacent to the existing markdown worked example so users can compare. Same example structure as the markdown one (input snippet → command invocation → expected resolution log).

- [x] 2.2 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s graceful-degradation section to add the new format-error cases: (a) neither-format-detected error (with the concrete error shape from design.md D-format-error-1), (b) JSON parse failure error, (c) unsupported-`command`-field error. Each error case includes a brief description of what triggers it + the recovery advice (user fixes file and re-runs).

- [x] 2.3 Verify the existing markdown ingest worked example in SKILL.md still reads coherently after the auto-detect + JSON additions — i.e., the markdown example should clarify it's the markdown path, not the only path. Lightly edit if needed.

## 3. Validation + user-validation handoff

- [x] 3.1 Run `openspec validate address-reviews-accepts-internal-json --strict`; resolve any validation findings.

- [x] 3.2 (User-validation) User reads the new `references/internal-findings-format.md` and confirms:
  - (a) Parser contract is unambiguously implementable from the doc alone (field mappings clear; `command` discriminator clear; malformed input cases enumerated)
  - (b) Worked example matches a real `review-<mode>-*.json` from the recent dogfood cycles (e.g., the iter-2 system review for harden-review-mode-recommendations would parse cleanly)
  - (c) Tolerated variations and strict requirements are clearly delineated (mirroring the external-findings doc's strict/lenient split)

- [x] 3.3 (User-validation) User reads the updated SKILL.md `--from-file <path> ingest` section and confirms:
  - (a) The auto-detect routing logic is clear (sniff leading character → markdown OR JSON)
  - (b) The JSON worked example demonstrates a realistic single-finding case end-to-end
  - (c) Format-error messages are clear enough that a user encountering them can self-diagnose
  - (d) The markdown path's behavior is unchanged from the user's perspective (zero regression in the cross-AI loop)

- [x] 3.4 (User-validation) User reads the orbit-address-reviews delta spec's new + modified scenarios (especially the 3 malformed-input scenarios) and confirms each scenario's WHEN/THEN is testable.

- [x] 3.5 If user-validation surfaces no blocking findings, the change is ready for `/opsx:review` (proposal-mode pre-apply review per the canonical flow). Then `/opsx:apply` → `/opsx:review --as system` → `/opsx:archive`. Optional empirical test: dogfood by feeding the just-archived change's own `review-*-*.json` files through `--from-file` and confirm the parser accepts them; this validates the JSON path against real orbit-emitted shapes.
