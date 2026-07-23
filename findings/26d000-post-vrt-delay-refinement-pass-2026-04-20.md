# `0x26D000` Post-`VRT` Delay Refinement Pass

Date: 2026-04-20

## Summary

This pass followed the anomaly-seeded replay result:

- coarse exec-break seeds could recreate the precursor anomaly statuses
- but the strongest useful bridge result came from a mild first post-`VRT` timing shift
- specifically:
  - `post_vrt_initial_command_delay_seconds = 0.2`
  - bridge -> `0008:0000149A`

The narrow question here was:

- does that bridge-side object-`1` result widen into a stable window when the first post-`VRT` command delay is refined around `0.2`

The broader three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran a six-point delay ladder on fresh clones:

- `0.00`
- `0.05`
- `0.10`
- `0.15`
- `0.20`
- `0.25`

Main result:

- bridge-side object `1` appeared at:
  - `0.10`
  - `0.20`
- both hits landed on selector `0008`
- the in-between point `0.15` fell back to flat `0868`
- the outer points `0.00`, `0.05`, and `0.25` also fell back to flat `0868`

So the best new closure is:

- the useful bridge window is real
- but it is discontinuous, not monotonic
- and the current live bridge family is `0008`, not `0824`

## New Owned Artifacts

- [26d000-post-vrt-delay-refinement-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-post-vrt-delay-refinement/26d000-post-vrt-delay-refinement-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T155519Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T155705Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T155851Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T160037Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T160223Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T160409Z/batch-summary.json)

Fixture root:

- [post-vrt-delay-refinement-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/post-vrt-delay-refinement-20260420T000000Z)

## Harness Preset

This pass added and exercised:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

Tooling fix discovered during the pass:

- `run_branch_family_batch.py` now prefers the selected live selector hint when multiple numeric `selinfo` queries are present

That matters here because the bridge slot queried:

- `0824`
- `0008`
- `0870`
- `0868`

and the child DOSBox logs, not the pre-fix batch summaries, are the source of truth for the selected bridge descriptor in this pass.

## Results

### Delay `0.00`

- `opening` -> `0868:00025160`
- `second` -> `0868:00023926`
- `bridge` -> `0868:00023A21`

### Delay `0.05`

- `opening` -> `0868:0002B880`
- `second` -> `0868:0002B883`
- `bridge` -> `0868:00024801`

### Delay `0.10`

- `opening` -> `0870:000003B0`
- `second` -> `0868:000261D6`
- `bridge` -> `0008:00001E05`

### Delay `0.15`

- `opening` -> `0870:000003D4`
- `second` -> `0868:00023954`
- `bridge` -> `0868:0002B821`

### Delay `0.20`

- `opening` -> `0868:0002B87F`
- `second` -> `0868:0002B821`
- `bridge` -> `0008:00000CA7`

### Delay `0.25`

- `opening` -> `0008:00000EFC`
- `second` -> `0868:00023A89`
- `bridge` -> `0868:000239F3`

### Bridge Descriptor Closure

For the bridge slot:

- `0.10` selected descriptor `0008`:
  - base `0x00008240`
  - limit `0x0000FFFF`
- `0.20` selected descriptor `0008`:
  - base `0x00008240`
  - limit `0x0000FFFF`
- `0.00`, `0.05`, `0.15`, and `0.25` selected descriptor `0868`:
  - base `0x00000000`
  - limit `0xFFFFFFFF`

Across all six bridge logs:

- `0008`, `0870`, and `0868` numeric descriptors were present
- `0824` still did not return a populated numeric descriptor

### File Diffs

Clone tree hash diffs:

- `d0p00-clone` -> `0`
- `d0p05-clone` -> `0`
- `d0p10-clone` -> `0`
- `d0p15-clone` -> `0`
- `d0p20-clone` -> `0`
- `d0p25-clone` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. The Bridge-Side Object-`1` Result Is Real But Narrow

This is the main result.

Bridge-side object `1` appeared at exactly two ladder points:

- `0.10` -> `0008:00001E05`
- `0.20` -> `0008:00000CA7`

So the useful post-`VRT` timing window is real.

### 2. The Window Is Not Monotonic

This is the most important shape correction.

If the bridge improved smoothly with increasing delay, `0.15` should have stayed on the object-`1` side between the two positive points.

It did not.

Instead:

- `0.15` fell back to `0868:0002B821`

So the live delay region is discontinuous, not a simple “more delay is better” slope.

### 3. The Useful Bridge Family In This Window Is `0008`, Not `0824`

Both bridge-side object-`1` hits armed and stopped on:

- selector `0008`

No bridge run reached:

- `0824`
- or `0870`

So the current strongest refinement path is improving the `0008` bridge family, not forcing a direct return to `0824`.

### 4. Opening-Slot Object-`1` Is Not Sufficient By Itself

Three ladder points reached object `1` before the bridge:

- `0.10` opening -> `0870:000003B0`
- `0.15` opening -> `0870:000003D4`
- `0.25` opening -> `0008:00000EFC`

But only one of the two opening-`0870` cases carried through to a bridge-side object-`1` result, and the opening-`0008` case at `0.25` still collapsed to flat `0868` at the bridge.

So an opening object-`1` stop is still not enough by itself to guarantee a useful bridge outcome.

### 5. Numeric Capture Is Working, But `0824` Remains Blank

This pass tightened the evidence quality.

Numeric bridge capture consistently preserved descriptors for:

- `0008`
- `0870`
- `0868`

But `0824` still did not produce a populated numeric descriptor in any bridge log.

That does not prove `0824` is impossible. It does reinforce that the current refinement window is working through the `0008` family instead.

## Practical Interpretation

This pass sharpens the next move.

We now know:

- the bridge-side object-`1` window is real
- it is not monotonic
- and its strongest currently reproducible family is `0008`

So the next pass should stay tightly inside the strongest region and test repeatability there before we spend more time on wider delay sweeps or on the coarser anomaly seeds.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Dense Post-`VRT` Window Confirmation Pass**

Focus:

- keep `pre_atet_exec_break_count = 1`
- keep the broader three-slot shape
- densify around the strongest region, for example:
  - `0.09`
  - `0.10`
  - `0.11`
  - `0.18`
  - `0.19`
  - `0.20`
  - `0.21`
- preserve bridge-side numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`
- treat repeatability of the `0008` bridge family as the immediate goal before chasing `0824` again

## Bottom Line

The post-`VRT` delay ladder confirmed a real bridge-side object-`1` window, but it is narrow and discontinuous.

The best bridge hits were:

- `0.10` -> `0008:00001E05`
- `0.20` -> `0008:00000CA7`

The next strongest move is a dense confirmation pass around those two live points.
