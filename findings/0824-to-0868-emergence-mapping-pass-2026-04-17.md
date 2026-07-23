# 0824-to-0868 Emergence Mapping Pass

Date: 2026-04-17

## Summary

This pass shifted the runtime question earlier again and directly mapped the first cold-lane emergence from shared `0824` staging into flat selector `0868`.

Main result:

- the crossover now has a bounded owned interval
- it occurs between `LOGC 0x260000` and `LOGC 0x270000`
- and under the current cold no-autoexec lane it lands directly in object `3`, not early object `2`

## New Owned Artifacts

- [0824-to-0868-emergence-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/0824-to-0868-emergence/0824-to-0868-emergence-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T163237Z/batch-summary.json)

## Harness Update

This pass also tightened the harness itself:

- `run_branch_family_batch.py` now parses `selinfo cs` output cleanly

That matters because the emergence preset intentionally asks the debugger for:

- `selinfo cs`
- `ldt`

instead of hard-coding a selector before the crossover is known.

## Emergence Bracket

All runs used:

- no autoexec steering
- target `0x17719`
- `selinfo cs`
- `ldt`

### Bounded `0870` Side

- `LOGC 0x240000`
  - child run `20260417T163237Z`
  - selected live stop `0870:000003E4`
  - object `1`
  - selector base `0x0000A3C0`
  - selector limit `0x0000FFFF`
- `LOGC 0x260000`
  - child run `20260417T163312Z`
  - selected live stop `0870:000003B7`
  - object `1`
  - selector base `0x0000A3C0`
  - selector limit `0x0000FFFF`

### Flat `0868` Side

- `LOGC 0x270000`
  - child run `20260417T163347Z`
  - selected live stop `0868:00024831`
  - object `3`
  - selector base `0x00000000`
  - selector limit `0xFFFFFFFF`
- `LOGC 0x280000`
  - child run `20260417T163423Z`
  - selected live stop `0868:0002481C`
  - object `3`
  - selector base `0x00000000`
  - selector limit `0xFFFFFFFF`
- `LOGC 0x300000`
  - child run `20260417T163458Z`
  - selected live stop `0868:000182E1`
  - object `3`
  - selector base `0x00000000`
  - selector limit `0xFFFFFFFF`

Across the full bracket, the first live VRT re-entry point still remained:

- `0824:0000006A`

## Findings

### 1. The Earliest Owned 0824-to-0868 Crossover Is Now Bounded

This is the main closure from the pass.

The cold no-autoexec lane stayed on bounded `0870` through:

- `LOGC 0x260000`

and had already crossed into flat `0868` by:

- `LOGC 0x270000`

So the project now owns a crossover interval, not just a vague “somewhere earlier” story.

### 2. The Crossover Lands In Object 3 Under The Current Cold Lane

The first flat `0868` emergence did **not** land in early object `2`.

At the first owned flat budget:

- `LOGC 0x270000`

the selected live stop was already:

- object `3`
- `0868:00024831`

That matters because it separates two questions that might otherwise look related:

- when does selector `0868` first appear
- when do we get a useful early object-2 frontier

Under the current cold lane, those are not the same event.

### 3. Shared `0824` Staging Remains Stable

Every run in the bracket still first re-entered at:

- `0824:0000006A`

So the overall runtime shape is now tighter than before:

- `0824` shared staging
- bounded `0870` through `0x260000`
- flat `0868` by `0x270000`
- and object-level behavior depends on the lane after that

### 4. The Next Runtime Question Is Now Smaller And Better Posed

We no longer need to ask:

- “roughly when does `0868` show up?”

We can now ask:

- what happens inside the narrow `0x260000 -> 0x270000` crossover window
- and whether any nearby steering or finer budget bracket can reach flat `0868` without immediately falling into object `3`

## Practical Interpretation

This pass gives the project a real transition seam to work against.

It does **not** solve the earliest useful object-2 problem yet.
But it tells us where the selector crossover is and shows that the current cold lane crosses too late to help by itself.

That is still a meaningful reduction in uncertainty.

## Recommended Next Move

The strongest next move is now:

- **Fine-Grained 0824-to-0868 Crossover Bracket Pass**

Focus:

- subdivide the `0x260000 -> 0x270000` window
- keep the no-autoexec lane as the control
- test whether any finer crossover budget lands in flat `0868` at a more informative offset than the current object-3 entries

## Bottom Line

The project now owns the earliest selector crossover interval:

- bounded `0870` through `LOGC 0x260000`
- flat `0868` by `LOGC 0x270000`

Under the current cold lane, that crossover lands directly in object `3`.

So the next useful work is not broad selector hunting anymore.
It is fine-grained crossover mapping inside that newly bounded window.
