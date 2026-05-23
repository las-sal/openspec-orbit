# Findings — lean-overlay-and-add-orbit-onboard (proposal-mode review iter-4, fresh-context Claude)

Bridge file converting `review-proposal-2026-05-23T20-06-43Z.json` for `/opsx:address-reviews` ingest.

**Source**: fresh-context Claude subagent --full pass on 2026-05-23T20:06:43Z (iter-4).
**Counts**: 0 CRITICAL, 4 WARNING, 1 SUGGESTION, 0 stale.

All 5 are mechanical residue from iter-6 cleanup — exactly the change-completeness discipline pattern.

---

## WARNING

### FF4-1 — tasks.md task 2.3 + task 5.4a reference removed task 3.6

`tasks.md:26,56`. Iter-6 renumbered 3.6 → 3.5; two cross-refs missed. s/task 3.6/task 3.5/g (2 instances).

### FF4-2 — design.md D-conventions-1 numbering skips item 5

`design.md:152-170`. Sequence: 1, 2, 3, 4, 6, 7, 8, 9, 10, 11 — no item 5. Renumber or note.

### FF4-3 — tasks.md Chunk 1 comment under-scopes vs Section 1 heading

`tasks.md:4`. Comment says "Overlay cleanup — delete feedback skill" but heading at L16 says "...+ sweep stale spec language". Update comment to match.

### FF4-5 — explore.md D7 heading still singles out sync as co-decision

`explore.md:52`. Heading "delete feedback, keep openspec-sync-specs" lacks the iter-6 amendment marker. Either annotate heading or rename to match iter-6 D-arch-4 rename.

---

## SUGGESTION

### FF4-4 — orbit-conventions baseline Purpose still has old `Distribution model — overlay, not CLI fork` verbatim

`openspec/specs/orbit-conventions/spec.md:5`. Baseline Purpose enumerates "the **distribution model** (overlay, not CLI fork)" — verbatim old title. After archive sync, Purpose summary and renamed Requirement title disagree. Analogous to known-issue but unflagged. Add task 1.4 (parallel to task 1.3) OR add to design.md item 4 known-issues bullet.
