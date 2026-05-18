# openspec-orbit

An opinionated `.claude/` overlay for [@fission-ai/openspec](https://github.com/Fission-AI/OpenSpec) that bakes a tested review-and-capture workflow into spec-driven development.

> **Status**: v1 design exploration. Command/skill sketches complete; implementation pending.

## Relationship to upstream OpenSpec

orbit is **not** a fork of the OpenSpec CLI. It's a downstream overlay that sits next to your project's existing `.claude/` directory after `openspec init` has run. It overrides/extends specific opsx skills and adds new ones — the CLI binary is unchanged.

orbit is developed with the discipline that *if* it were ever upstreamed, the diff should read like a contributor's PR — not like two different products. See `openspec/explore/bootstrap-openspec-orbit/explore.md` for the full set of guiding principles.

## What's new vs. upstream opsx

| Command | Type | What it does |
|---|---|---|
| `/opsx:review-proposal` | new | Editorial pass over change artifacts before apply (9 passes, 3-dimension scorecard) |
| `/opsx:review-system` | new | Editorial pass over the whole product after apply; wraps `verify-change` + 6 system-wide passes |
| `/opsx:audit-drift` | new | Project-wide scan for drift in captured knowledge vs. reality (vocabulary residue, lens staleness, doc consistency, archive coherence) |
| `/opsx:address-reviews` | new | Walk `@review:` markers with pushback discipline; resolves them, removes on completion |
| `/opsx:review-external` | new | Package a review request for an external AI (codex, fresh Claude, etc.); ingests results via `address-reviews --from-file` |
| `/opsx:explore` | modified | Adds capture affordances (conventions, perspectives, critical paths) + authors `explore.md` |
| `/opsx:propose` | modified | Consumes `openspec/explore/<name>/explore.md`; promotes staging dir to changes |
| `/opsx:archive` | modified | Auto-invokes `audit-drift` as a pre-archive sweep |

Three guiding principles shape the design:

1. **Coherence with openspec form and spirit** — orbit speaks openspec's vocabulary, follows its file layout, adopts its reporting conventions.
2. **Up-front cost trumps downstream cost** — defaults lean toward comprehensive; pay the LLM/compute cost now to catch issues before they escape.
3. **Specs as the ultimate source of truth** — orbit's tooling assumes the spec set is authoritative; code is one realization.

## Layout

```
.claude/
├── commands/opsx/           ← slash command bodies (orbit ships overrides + new ones)
└── skills/openspec-*/       ← skill definitions (orbit ships overrides + new ones)

openspec/
├── changes/                 ← in-flight + archived openspec changes (upstream)
├── specs/                   ← canonical baseline specs (upstream)
├── explore/                 ← orbit addition: staging area for /opsx:explore
│   └── <name>/
│       ├── explore.md       ← exploration record (5 sections)
│       └── sketches/        ← optional design sketches
├── lenses/                  ← orbit addition: review judgment layer
│   ├── perspectives.md      ← named callers worth validating from
│   └── critical-paths.md    ← user flows worth walking end-to-end
└── config.yaml              ← openspec config (upstream)
```

## Design references

- **`openspec/explore/bootstrap-openspec-orbit/explore.md`** — full design record (guiding principles, decisions, considered alternatives)
- **`openspec/explore/bootstrap-openspec-orbit/sketches/`** — detailed sketches per command:
  - `review-proposal.md`, `review-system.md`, `audit-drift.md`
  - `address-reviews.md`, `review-external.md`
  - `explore.md`, `propose.md`, `archive.md`

## v2 / future work

Tracked as GitHub issues:

- [#1](https://github.com/las-sal/openspec-orbit/issues/1) — Caching of pass results
- [#2](https://github.com/las-sal/openspec-orbit/issues/2) — `--thorough` extras for review commands
- [#3](https://github.com/las-sal/openspec-orbit/issues/3) — Comprehensive `/opsx:address-reviews` features (paste, cascade, severity, etc.)

Plus `/opsx:distill-specs` (canonical-spec hygiene) deferred to v2; scope notes captured in `explore.md`.

## Installation

*Installation instructions pending — v1 implementation not yet written.* The intended pattern after `openspec init`:

```bash
# After upstream openspec init has set up your project's .claude/
git clone https://github.com/las-sal/openspec-orbit /tmp/orbit
cp -r /tmp/orbit/.claude/commands/opsx/. .claude/commands/opsx/
cp -r /tmp/orbit/.claude/skills/. .claude/skills/
```

Long-term: package as a proper Claude Code plugin (Phase 2).

## License

MIT. See [LICENSE](./LICENSE).
