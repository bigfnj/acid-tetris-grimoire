# `0x26D000` Delay-`0.18` Bridge Family Stabilization Pass

Date: 2026-04-20

## Summary

This pass followed the dense post-`VRT` window confirmation result:

- the earlier bridge-side `0008` hits had not reproduced
- the strongest new bridge lead was:
  - `0.18` -> `0870:00000823`

The narrow question here was:

- is that `0.18` bridge-side `0870` family actually repeatable across fresh clones
- and if not, do the immediate bracket points `0.175` and `0.185` reveal a small local pocket

The broader three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran:

- five fresh repeats at `0.18`
- two fresh repeats at `0.175`
- two fresh repeats at `0.185`

Main result:

- only one of the five exact `0.18` repeats reached bridge-side `0870`
- both bracket points stayed flat `0868` at the bridge

So the best new closure is:

- the `0.18` bridge-side `0870` family is real
- but it is sparse, not stable
- and fine delay bracketing alone does not explain it

## New Owned Artifacts

- [26d000-delay-0p18-bridge-family-stabilization-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-delay-0p18-bridge-family-stabilization/26d000-delay-0p18-bridge-family-stabilization-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164004Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164150Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164337Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164523Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164710Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T164942Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T165129Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T165315Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T165501Z/batch-summary.json)

Fixture roots:

- [delay-0p18-bridge-family-stabilization-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/delay-0p18-bridge-family-stabilization-20260420T000000Z)
- [delay-0p18-tight-bracket-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/delay-0p18-tight-bracket-20260420T000000Z)

## Harness Preset

This pass reused:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

No additional tooling changes were needed inside the pass itself.

## Results

### Exact `0.18` Repeats

#### Repeat 1

- `opening` -> `0868:000239F3`
- `second` -> `0868:0002577A`
- `bridge` -> `0868:0002B552`

#### Repeat 2

- `opening` -> `0868:00025151`
- `second` -> `0868:00025165`
- `bridge` -> `0870:00000835`

#### Repeat 3

- `opening` -> `0868:000239F3`
- `second` -> `0868:0002B821`
- `bridge` -> `0868:0002B882`

#### Repeat 4

- `opening` -> `0008:000014A8`
- `second` -> `0868:000257B4`
- `bridge` -> `0868:0002B821`

#### Repeat 5

- `opening` -> `0870:00000430`
- `second` -> `0868:0002B821`
- `bridge` -> `0868:0002B82A`

### Tight Bracket `0.175`

#### Repeat 1

- `opening` -> `0868:0002B826`
- `second` -> `0868:0002481E`
- `bridge` -> `0868:00025F6D`

#### Repeat 2

- `opening` -> `0868:00015099`
- `second` -> `0868:000257A9`
- `bridge` -> `0868:0002B885`

### Tight Bracket `0.185`

#### Repeat 1

- `opening` -> `0868:000258B3`
- `second` -> `0868:0002B880`
- `bridge` -> `0868:00023BBF`

#### Repeat 2

- `opening` -> `0868:0002B824`
- `second` -> `0868:0002B87C`
- `bridge` -> `0868:000239F2`

### Bridge Descriptor Closure

The only bridge-side object-`1` result in the whole pass was:

- exact `0.18` repeat 2 -> selector `0870`
  - base `0x0000A3C0`
  - limit `0x0000FFFF`

All other bridge runs stayed on:

- selector `0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`

### File Diffs

Clone tree hash diffs:

- `0.18-r1` -> `0`
- `0.18-r2` -> `0`
- `0.18-r3` -> `0`
- `0.18-r4` -> `0`
- `0.18-r5` -> `0`
- `0.175-r1` -> `0`
- `0.175-r2` -> `0`
- `0.185-r1` -> `0`
- `0.185-r2` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. The Bridge-Side `0870` Family At `0.18` Did Not Stabilize

This is the main closure.

Across five exact `0.18` repeats:

- only one run reached bridge-side object `1`
- four runs fell back to flat `0868` object `3`

So the `0.18` bridge-side `0870` family is real, but sparse.

### 2. The Immediate Bracket Did Not Reveal A Local Pocket

Both tight bracket points:

- `0.175`
- `0.185`

stayed entirely flat at the bridge in both repeats.

So the `0.18` hit did not widen into a clean local plateau or symmetric pocket.

### 3. Opening-Slot Object-`1` Still Was Not Sufficient

Even inside the `0.18` repeat cluster, object `1` appeared before the bridge in several ways:

- `0.18-r4` opening -> `0008:000014A8`
- `0.18-r5` opening -> `0870:00000430`

Yet both runs still collapsed back to flat `0868` at the bridge.

So the bridge depends on more than simply reaching object `1` early in the lane.

### 4. The Rare Live Sequence Is Still The Target

The one successful exact `0.18` repeat had:

- `opening` on flat `0868`
- `second` on flat `0868`
- `bridge` on `0870`

That is interesting because it means the successful bridge-side `0870` run did not need an obvious earlier object-`1` precursor inside the same three-slot lane.

This is an inference from the observed selected live stops.

### 5. `0824` Still Did Not Reappear

As in the previous passes:

- no bridge run reached `0824`

So the practical frontier is still below the rare `0824` family.

## Practical Interpretation

This pass narrows the strategy again.

We now know:

- `0.18` is the best current delay value for the bridge-side `0870` family
- but that family is not stable enough to treat the delay itself as sufficient
- and the immediate bracket does not explain the hit

So the next best move is to stop refining delay for a moment and instead correlate the rare `0.18` bridge-side `0870` hit against precursor slot behavior across a larger repeat sample.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Delay-`0.18` Precursor Correlation Pass**

Focus:

- repeat exact delay `0.18` across a somewhat larger sample
- treat:
  - opening-slot selector family
  - second-slot selector family
  as the primary comparison axis
- preserve bridge-side numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`
- prefer correlation and clustering over more fine-grained delay sweeps

## Bottom Line

The bridge-side `0870` hit at `0.18` is real but not stable.

Only one of five exact `0.18` repeats reproduced it, and both `0.175` and `0.185` stayed flat at the bridge. The next strongest move is to correlate that rare `0.18` bridge-side `0870` hit against precursor slot behavior rather than keep shrinking the delay bracket.
