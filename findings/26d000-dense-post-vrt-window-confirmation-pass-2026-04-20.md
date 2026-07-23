# `0x26D000` Dense Post-`VRT` Window Confirmation Pass

Date: 2026-04-20

## Summary

This pass followed the first post-`VRT` delay refinement result:

- the bridge-side object-`1` window looked real
- the live hits were:
  - `0.10` -> `0008:00001E05`
  - `0.20` -> `0008:00000CA7`
- the window already looked discontinuous because `0.15` had fallen back to `0868`

The narrow question here was:

- do the two live bridge points at `0.10` and `0.20` reproduce under a denser ladder, or does a different bridge family emerge inside the surrounding window

The broader three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran a seven-point dense ladder:

- `0.09`
- `0.10`
- `0.11`
- `0.18`
- `0.19`
- `0.20`
- `0.21`

Main result:

- neither earlier `0008` bridge hit reproduced
- the strongest new lead was:
  - `0.18` -> `0870:00000823`

So the best new closure is:

- the live window is still real
- but its family is not yet stable
- and the strongest current bridge lead has shifted to `0870` at `0.18`

## New Owned Artifacts

- [26d000-dense-post-vrt-window-confirmation-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-dense-post-vrt-window-confirmation/26d000-dense-post-vrt-window-confirmation-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T161638Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T161824Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T162010Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T162157Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T162344Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T162530Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T162717Z/batch-summary.json)

Fixture root:

- [dense-post-vrt-window-confirmation-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/dense-post-vrt-window-confirmation-20260420T000000Z)

## Harness Preset

This pass reused:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

Tooling note:

- the batch harness parser was tightened during this pass so future runs prefer the selected live selector hint when several numeric `selinfo` queries are present
- for this pass, the child DOSBox logs are the source of truth for the selected bridge descriptor because the ladder itself ran just before that fix landed

## Results

### Delay `0.09`

- `opening` -> `0870:00000813`
- `second` -> `0868:0002B826`
- `bridge` -> `0868:0002B882`

### Delay `0.10`

- `opening` -> `0868:00023927`
- `second` -> `0868:0002391A`
- `bridge` -> `0868:0002B81E`

### Delay `0.11`

- `opening` -> `0868:00023979`
- `second` -> `0868:0002514F`
- `bridge` -> `0868:00025393`

### Delay `0.18`

- `opening` -> `0868:0002392C`
- `second` -> `0008:00000C90`
- `bridge` -> `0870:00000823`

### Delay `0.19`

- `opening` -> `0870:0000083C`
- `second` -> `0868:0002B819`
- `bridge` -> `0868:00024827`

### Delay `0.20`

- `opening` -> `0870:00000829`
- `second` -> `0868:000256C8`
- `bridge` -> `0868:0002B82A`

### Delay `0.21`

- `opening` -> `0868:0002577F`
- `second` -> `0868:0002B81E`
- `bridge` -> `0868:0002B824`

### Bridge Descriptor Closure

The selected bridge descriptors were:

- `0.18` -> selector `0870`
  - base `0x0000A3C0`
  - limit `0x0000FFFF`
- all other points -> selector `0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`

Across all seven bridge logs:

- `0008`, `0870`, and `0868` numeric descriptors were present
- `0824` still did not return a populated numeric descriptor

### File Diffs

Clone tree hash diffs:

- `d0p09-clone` -> `0`
- `d0p10-clone` -> `0`
- `d0p11-clone` -> `0`
- `d0p18-clone` -> `0`
- `d0p19-clone` -> `0`
- `d0p20-clone` -> `0`
- `d0p21-clone` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. The Earlier `0008` Bridge Hits Did Not Reproduce

This is the main confirmation result.

The two previously live bridge points:

- `0.10`
- `0.20`

both collapsed back to flat `0868` object `3` in this denser sweep.

So repeatability of the earlier `0008` bridge family is not yet established.

### 2. A New Stronger Bridge Lead Emerged At `0.18`

This is the most important new positive result.

At:

- `0.18`

the second slot reached:

- `0008:00000C90`

and the bridge advanced to:

- `0870:00000823`

That is currently the strongest bridge-side lead in the dense neighborhood.

### 3. The Local Window Remains Discontinuous

This remains the key shape property.

If the bridge family were stable across the neighborhood, `0.19` and `0.20` should have stayed on the object-`1` side once `0.18` did.

They did not:

- `0.19` -> `0868:00024827`
- `0.20` -> `0868:0002B82A`

So the delay window remains fragmented rather than smooth.

### 4. Opening-Slot Object-`1` Is Still Not Sufficient

Three points reached object `1` before the bridge:

- `0.09` opening -> `0870:00000813`
- `0.19` opening -> `0870:0000083C`
- `0.20` opening -> `0870:00000829`

But none of those points reached a bridge-side object-`1` result.

So the bridge continues to depend on more than simply “opening slot reached object `1`.”

### 5. The Useful Bridge Family Has Shifted From `0008` To `0870`

In the previous pass, the strongest bridge-side object-`1` hits were on:

- `0008`

In this pass, the only bridge-side object-`1` hit was on:

- `0870`

That shift matters. It suggests the current live region is not merely unstable in timing, but may also be bifurcating across selector families.

### 6. `0824` Still Did Not Reappear

As in the previous passes, no bridge run reached:

- `0824`

and no bridge log returned a populated numeric `selinfo 0824` descriptor.

So the practical frontier is still downstream of the rare `0824` family.

## Practical Interpretation

This pass meaningfully changes the target.

We now know:

- the previous `0008` bridge hits are not yet repeatable
- the strongest new bridge lead is `0870` at `0.18`
- and the dense neighborhood is still discontinuous

So the best next move is not another wide delay sweep. It is a focused stabilization pass around the new `0.18` lead.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Delay-`0.18` Bridge Family Stabilization Pass**

Focus:

- repeat delay `0.18` across several fresh clones
- only add a very tight bracket such as:
  - `0.175`
  - `0.18`
  - `0.185`
  if needed after the first repeat cluster
- preserve bridge-side numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`
- treat stability of the bridge-side `0870` family as the immediate goal before returning to wider window searches

## Bottom Line

The dense confirmation pass did not validate the earlier `0008` bridge hits at `0.10` and `0.20`.

Instead, it surfaced a new strongest bridge-side lead at:

- `0.18` -> `0870:00000823`

The next strongest move is to stabilize that `0.18` bridge family directly.
