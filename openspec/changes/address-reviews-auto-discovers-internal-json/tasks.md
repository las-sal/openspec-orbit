<!--
Implementation chunks (per orbit canonical chunking — drives apply chunk-end emits):

  Chunk 1 (group 1):   SKILL.md discovery-step update +
                       opsx/address-reviews.md mirror
                       (auto-discovery logic + Step 1 fallback prose)
  Chunk 2 (group 2):   README + onboard SKILL/command updates +
                       reference-doc note that internal-findings-format.md
                       is also consumed by auto-discovery (not just --from-file)
  Chunk 3 (group 3):   Validation + user-validation handoff

Total: 3 implementation chunks. No sandbox verification needed: this is
prose + behavior-contract documentation (no CLI behavior change — the
auto-discovery logic is internal to the address-reviews skill prose).
Sync-specs at archive applies the MODIFIED requirement to baseline.
-->

## 1. SKILL + command body — discovery-step fallback

- [ ] 1.1 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s Step 1 (Discover) section to add the auto-discovery fallback logic for change-name positional scope. New sub-step structure:
  - (a) Run the existing grep for `@review:` markers in the change directory.
  - (b) If grep finds zero markers AND no `--from-file` flag specified AND positional argument resolves to an existing change directory (active or archived): look in the change's `.orbit-runs/` directory for `review-<mode>-*.json` OR `audit-drift-*.json` files; pick the single most-recent by filename `<TS>` token across all candidate types; if found, feed to the `--from-file` ingest path (content-sniff routes to JSON parser per `references/internal-findings-format.md`).
  - (c) If neither markers nor JSON candidates: emit `No @review: markers in scope and no internal review/audit-drift JSON in .orbit-runs/. Nothing to walk.` and exit cleanly.
  - Preserve current behavior for whole-repo invocation (bare `/opsx:address-reviews`) and path/pattern scope (`/opsx:address-reviews <path>`) — auto-discovery does NOT apply for those.

- [ ] 1.2 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s discovery prose to document the discovery priority order explicitly: markers → JSON candidate → empty. Specifically: the section MUST note that markers always win when present (preserves current behavior); JSON-fallback only triggers when markers are absent; `--from-file` always overrides auto-discovery regardless of marker/JSON state.

- [ ] 1.3 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s discovery prose to document the recency rule: filename `<TS>` token, lexical sort, single global "most recent" wins — `review-<mode>-*.json` and `audit-drift-*.json` compete on the same recency axis (no class preference per D-recency-1).

- [ ] 1.4 Update `.claude/commands/opsx/address-reviews.md` to mirror SKILL.md changes from 1.1–1.3. Per orbit-modified command-file convention, the body sections mirror the SKILL (excluding frontmatter); the auto-discovery fallback + recency rule + `--mark`-no-longer-prerequisite framing appear in both files.

- [ ] 1.5 Update `.claude/skills/openspec-address-reviews/SKILL.md`'s description of `--mark` (currently positioned as "the way to produce markers for the address-reviews workflow"): reframe as "optional convenience for source-level diff-readability"; explicitly state auto-discovery makes `--mark` optional, not prerequisite.

## 2. Reference docs + README + onboard surfaces

- [ ] 2.1 Lightly update `.claude/skills/openspec-address-reviews/references/internal-findings-format.md` to add a sentence at the top noting: "This parser contract is consumed by BOTH explicit `--from-file <path>` invocations AND auto-discovery from `.orbit-runs/` when `/opsx:address-reviews <change-name>` finds no inline markers. The parsing logic and virtual-marker construction are identical regardless of how the JSON entered the lifecycle."

- [ ] 2.2 Update `README.md`'s `/opsx:address-reviews` command-reference description (~L417 area in the command-reference block) to add a note: "When invoked with a change-name positional and no `@review:` markers are found, auto-discovers the most recent internal `review-<mode>-*.json` or `audit-drift-*.json` in the change's `.orbit-runs/` directory and walks it — the canonical `review → address-reviews` workflow is now 2 commands, no `--from-file` or `--mark` pre-step needed." Verify line numbers at edit time.

- [ ] 2.3 Update `.claude/skills/openspec-onboard/SKILL.md` and `.claude/commands/opsx/onboard.md` command-table rows for `/opsx:address-reviews`: append "; auto-discovers internal review/audit-drift JSON when no markers found at change-name scope" to the existing one-line description.

- [ ] 2.4 Update the run-summary-schema reference at `.claude/skills/openspec-address-reviews/references/run-summary-schema.md` to document the new `source` value `"auto-discovered"` in BOTH sites: (a) the JSON-shape enum at L31 (currently `"whole-repo" | "scope" | "from-file"` — extend to add `"auto-discovered"`); AND (b) the field-note prose at L69 describing the `source` field (currently only describes the three existing values — extend to describe `"auto-discovered"` and when it's used: change-name positional scope + markers absent + candidate JSON found in `.orbit-runs/`). Both sites must agree; mismatched enum vs prose is a drift risk caught by future audit-drift Cat 3.

## 3. Validation + user-validation handoff

- [ ] 3.1 Run `openspec validate address-reviews-auto-discovers-internal-json --strict`; resolve any validation findings.

- [ ] 3.2 (User-validation) User reads the updated SKILL.md Step 1 discovery prose and confirms:
  - (a) The priority order (markers → JSON → empty) is clear and matches the spec scenarios
  - (b) The recency rule (filename `<TS>` token, single global most-recent across both candidate types) is unambiguous
  - (c) The `--mark`-no-longer-prerequisite framing reads coherently
  - (d) The whole-repo invocation + path/pattern scope skip-auto-discovery cases are explicit
  - (e) Resolution log captures the auto-discovery decision (source path, source command, why this candidate won)

- [ ] 3.3 (User-validation) User reads the new + modified scenarios in the orbit-address-reviews delta spec (8 net-new scenarios = 1 happy path + 7 edge cases on top of the 2 existing baseline scenarios = 10 scenarios total) and confirms each scenario's WHEN/THEN is testable.

- [ ] 3.4 (User-validation) User reads the updated README + onboard surfaces and confirms the auto-discovery convenience is discoverable for new users (TOC/command-table entry; not buried in skill prose).

- [ ] 3.5 If user-validation surfaces no blocking findings, the change is ready for `/opsx:review` (proposal-mode pre-apply review per the canonical flow). Then `/opsx:apply` → `/opsx:review --as system` → `/opsx:archive`. Optional empirical test: dogfood by running `/opsx:review <some-active-change>` followed by `/opsx:address-reviews <same-change>` with no `--from-file` flag and confirm the auto-discovery fires correctly against a real orbit-emitted shape.
