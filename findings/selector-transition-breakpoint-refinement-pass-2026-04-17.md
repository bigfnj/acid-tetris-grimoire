# Selector-Transition Breakpoint Refinement Pass

Date: 2026-04-17

## Summary

This pass used the new batch harness to refine the selector-transition model rather than just proving the selector split exists.

The goal was to answer a specific question:

- is the useful late object-2 to object-3 threshold cliff caused by a selector change
- or does it happen later inside the already-flat `0868` family

Main result:

- the late cliff happens inside selector `0868`
- not at the selector transition itself

That is the strongest runtime-model narrowing we have gotten since the selector split was first documented.

## New Owned Artifacts

- [selector-transition-refinement-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/selector-transition-refinement/selector-transition-refinement-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T162059Z/batch-summary.json)

## Harness Preset

This pass added and exercised the new batch preset:

- `selector-transition-refinement`

It brackets both families:

- bounded `0870` cold lane
- flat `0868` frontier lane

## Results

### 1. Cold `0870` Family Stayed Bounded

- `cold_bootstrap_100000`
  - child run `20260417T162059Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0870:00000425`
  - object `1`
  - armed `0870:03830`
  - selector base `0x0000A3C0`
  - selector limit `0x0000FFFF`
- `cold_inventory_260000`
  - child run `20260417T162134Z`
  - first live VRT stop `0824:0000006A`
  - selected live stop `0870:000003F2`
  - object `1`
  - armed `0870:17719`
  - selector base `0x0000A3C0`
  - selector limit `0x0000FFFF`

So even by `LOGC 0x260000`, the cold no-autoexec family still stayed on bounded selector `0870`.

### 2. Late `0868` Frontier Stayed Flat Across The Threshold Bracket

- `frontier_threshold_3bf000`
  - child run `20260417T162210Z`
  - selected live stop `0868:000178B9`
  - object `2`
- `frontier_threshold_3c0800`
  - child run `20260417T162245Z`
  - selected live stop `0868:000178BB`
  - object `2`
- `frontier_threshold_3c1000`
  - child run `20260417T162320Z`
  - selected live stop `0868:000236BE`
  - object `3`

All three frontier runs reported the same selector metadata:

- selector `0868`
- base `0x00000000`
- limit `0xFFFFFFFF`
- flags `11011`

That means the object-2 to object-3 flip occurred without leaving selector `0868`.

## Findings

### 1. The Late Object-2/Object-3 Cliff Is Not A Selector Transition

This is the core result.

The refinement batch held selector `0868` constant across:

- `LOGC 0x3BF000`
- `LOGC 0x3C0800`
- `LOGC 0x3C1000`

Yet the selected live object changed from:

- object `2`

to:

- object `3`

So the late threshold cliff is an in-family runtime split, not the moment where execution first crosses into flat selector `0868`.

### 2. The Cold Lane Still Does Not Naturally Grow Into `0868`

The bounded cold lane stayed on selector `0870` at both:

- `LOGC 0x100000`
- `LOGC 0x260000`

So the current no-autoexec family still does not simply "age into" the flat frontier family by spending more instruction budget.

That is useful because it keeps narrowing where the real transition seam must live.

### 3. `0824:0000006A` Remains The Shared Staging Point

All five runs in this pass still re-entered live protected-mode code first at:

- `0824:0000006A`

So the broad runtime shape remains:

- shared `0824` staging
- then split into bounded `0870` or flat `0868`
- then later object-level divergence inside `0868`

### 4. The Best Next Move Shifts Earlier Again

Because the late object-2/object-3 cliff is now shown to be internal to `0868`, there is less value in spending another pass on late `0868` helper targets.

The more important open question is now:

- where and how does execution emerge from `0824` into `0868` in the first place

## Practical Interpretation

This pass sharpened the runtime model in a way that changes prioritization.

Before this pass, it was still plausible that the late threshold behavior might reflect a selector crossover.
After this pass, that explanation is no longer the best one.

The selector crossover happens earlier.
The object-2/object-3 split happens later, inside already-flat `0868`.

## Recommended Next Move

The strongest next move is now:

- **0824-to-0868 Emergence Mapping Pass**

Focus:

- use the harness to bracket finer early budgets and/or nearby target families around first `0868` emergence
- stop treating late `0868` object2/object3 drift as the selector transition itself
- keep the original helper-family closure retry gated until a run actually beats `0868:000178B1`

## Bottom Line

The selector-transition refinement pass showed that the late useful threshold cliff is not caused by changing selectors.

It happens inside selector `0868`.

That moves the real open runtime question earlier in the chain:

- not "what happens at the late `0868` cliff?"
- but "how does execution first emerge from `0824` into flat `0868`?"
