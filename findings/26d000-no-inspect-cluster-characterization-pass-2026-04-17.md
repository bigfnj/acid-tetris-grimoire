# `0x26D000` No-Inspect Cluster Characterization Pass

Date: 2026-04-17

## Summary

This pass tested the strongest hypothesis left from the position-permutation result:

- that short no-inspect clustering itself is the main selector for the object-`1` side

It used two focused batch families:

- a pure no-inspect ladder
- an inspect-reset ladder

Main result:

- pure no-inspect depth alone did **not** reproduce any object-`1` family in this pass
- the inspect-reset ladder opened on a `0008` object-`1` landing at the inspect seed itself
- the following no-inspect runs after that inspect seed stayed flat `0868`

So the current model has to tighten again:

- no-inspect clustering is not sufficient by itself
- the `0008` family is not cleanly explained as “the first no-inspect after inspect”

## New Owned Artifacts

- [26d000-no-inspect-cluster-characterization-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-no-inspect-cluster-characterization/26d000-no-inspect-cluster-characterization-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T195603Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T195827Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-no-inspect-ladder`
- `selector-26d000-inspect-reset-ladder`

## Results

### Pure No-Inspect Ladder

- `26D000_no_inspect_ladder_1`
  - `0868:0002542F`
  - object `3`
  - flat `0868`
- `26D000_no_inspect_ladder_2`
  - `0868:00024821`
  - object `3`
  - flat `0868`
- `26D000_no_inspect_ladder_3`
  - `0868:00025163`
  - object `3`
  - flat `0868`
- `26D000_no_inspect_ladder_4`
  - `0868:0002B880`
  - object `3`
  - flat `0868`

### Inspect-Reset Ladder

- `26D000_inspect_reset_seed_1`
  - `0008:00001E3F`
  - object `1`
  - selector `0008`
- `26D000_after_inspect_no_inspect_2`
  - `0868:000239EE`
  - object `3`
  - flat `0868`
- `26D000_after_no_inspect_no_inspect_3`
  - `0868:0002482E`
  - object `3`
  - flat `0868`
- `26D000_inspect_reset_4`
  - `0868:00024816`
  - object `3`
  - flat `0868`
- `26D000_after_reset_no_inspect_5`
  - `0868:000253C9`
  - object `3`
  - flat `0868`

## Findings

### 1. Pure No-Inspect Cluster Depth Is Not Sufficient

This is the biggest correction from the prior interpretation.

In the dedicated no-inspect ladder:

- first no-inspect run -> flat `0868`
- second no-inspect run -> flat `0868`
- third no-inspect run -> flat `0868`
- fourth no-inspect run -> flat `0868`

So the owned evidence from this pass does **not** support a deterministic rule like:

- “second no-inspect goes object `1`”
- “third no-inspect goes object `1`”
- or “deep no-inspect clusters force the fallback”

No-inspect depth may still participate, but by itself it is not enough.

### 2. The `0008` Family Can Attach To The Inspect Slot Itself

This is the most useful positive result from the pass.

In the inspect-reset ladder, the very first inspect seed landed at:

- `0008:00001E3F`
- object `1`

That matters because the previous short permutation batch had shown:

- `0008:00001491`

on an early no-inspect-after-inspect slot.

Taken together, the `0008` family is now clearly not owned only on no-inspect continuation.

It can also appear directly on an inspect slot.

### 3. Inspect Does Not Reliably Reset The Next No-Inspect Run Into `0008`

If inspect were a clean reset trigger for the alternate object-`1` branch, we would expect:

- inspect seed -> maybe flat or `0008`
- next no-inspect after inspect -> `0008`

But that did not happen here.

Observed:

- inspect seed -> `0008`
- immediate no-inspect after inspect -> flat `0868`
- second no-inspect after inspect -> flat `0868`
- later inspect reset -> flat `0868`
- no-inspect after reset -> flat `0868`

So the reset story is not simple either.

### 4. The Remaining Instability Is Broader Than “No-Inspect History”

This pass is valuable because it removes a tempting but incomplete explanation.

After the prior permutation pass, it was reasonable to suspect:

- local no-inspect history is the main selector

After this pass, the stronger interpretation is:

- no-inspect history can correlate with object-`1` outcomes
- but it is not sufficient on its own
- and the `0008` family is not uniquely tied to the first no-inspect after inspect

That means the remaining seam still depends on another hidden condition or run-family choice.

### 5. The Object-`1` Side Still Has At Least Two Real Families

Nothing in this pass weakens that closure.

Across the owned passes, `0x26D000` now still has:

- bounded `0870`
- selector `0008`

What changed is the interpretation of how they are selected.

The new evidence says:

- `0870` is not forced by pure no-inspect depth
- `0008` can appear on inspect itself

## Practical Interpretation

This pass gives us a cleaner boundary around what the seam is **not**.

It is not:

- a simple absolute-final-position rule
- a simple inspect-versus-no-inspect rule
- a simple no-inspect-cluster-depth rule
- a simple inspect-reset rule

That is useful narrowing.

The next pass should stop treating the two object-`1` families as a single “fallback side” and instead target the `0008` family directly.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` `0008` Family Confirmation Pass**

Focus:

- run short batches that deliberately start with inspect and repeat inspect-led openings
- compare:
  - inspect-only openings
  - inspect then no-inspect
  - inspect then inspect
  - no-inspect then inspect
- determine whether selector `0008` is primarily:
  - an inspect-slot family
  - an opening-slot family
  - or part of a broader unstable startup family independent of immediate history

## Bottom Line

Pure no-inspect depth did not reproduce the object-`1` side in this pass, so no-inspect clustering alone is not enough.

The strongest new positive signal is that selector `0008` can appear directly on an inspect slot, which makes direct `0008` family isolation the next best move.
