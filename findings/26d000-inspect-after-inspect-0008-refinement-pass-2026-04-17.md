# `0x26D000` Inspect-After-Inspect `0008` Refinement Pass

Date: 2026-04-17

## Summary

This pass refined the only currently owned live `0008` reproduction shape:

- a later inspect-after-inspect slot inside a mixed batch

The pass asked one narrow question:

- which immediate predecessor is actually needed before the final inspect-after-inspect probe

Main result:

- the exact successful shape did **not** replay `0008`
- but the final probe still stayed on object `1` when the opening `no-inspect` was preserved
- removing the opening `no-inspect` collapsed the final probe back to flat `0868`

So the best new closure is not “how to force `0008`,” but:

- opening `no-inspect` appears to be important for keeping the final probe on the object-`1` side at all
- the earlier inspect-after-no-inspect step is not required for that broader object-`1` outcome

## New Owned Artifacts

- [26d000-inspect-after-inspect-0008-refinement-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-inspect-after-inspect-0008-refinement/26d000-inspect-after-inspect-0008-refinement-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T205433Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T205703Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-0008-success-shape-replay`
- `selector-26d000-0008-predecessor-variants`

## Results

### Exact Successful-Shape Replay

- `26D000_0008_shape_no_inspect_s1`
  - `0870:00000425`
  - object `1`
- `26D000_0008_shape_inspect_after_no_inspect_s2`
  - `0868:00024827`
  - object `3`
- `26D000_0008_shape_inspect_bridge_s3`
  - `0868:0002B824`
  - object `3`
- `26D000_0008_shape_inspect_after_inspect_probe_s4`
  - `0870:000003E4`
  - object `1`

### Variant A: Remove Opening `no-inspect`

- `26D000_0008_variant_all_inspect_v1`
  - `0868:0002B824`
  - object `3`
- `26D000_0008_variant_all_inspect_v2`
  - `0870:00000C02`
  - object `1`
- `26D000_0008_variant_all_inspect_v3`
  - `0868:0002B821`
  - object `3`
- `26D000_0008_variant_all_inspect_probe_v4`
  - `0868:0002B824`
  - object `3`

### Variant B: Preserve Opening `no-inspect`, Remove Earlier `inspect-after-no-inspect`

- `26D000_0008_variant_noinspect_noinspect_v5`
  - `0868:00023796`
  - object `3`
- `26D000_0008_variant_noinspect_noinspect_v6`
  - `0868:00023927`
  - object `3`
- `26D000_0008_variant_noinspect_then_inspect_v7`
  - `0868:00025154`
  - object `3`
- `26D000_0008_variant_noinspect_then_inspect_probe_v8`
  - `0870:00001162`
  - object `1`

## Findings

### 1. The Exact Successful Shape Is Not Stable Enough To Replay `0008`

This is the first thing to state clearly.

The exact replay of the newly successful mixed-history shape did not return `0008`.

Instead:

- opening no-inspect -> bounded `0870`
- final inspect-after-inspect probe -> bounded `0870`

So the rare `0008` branch remains unstable even inside the best current reproduction lane.

### 2. Opening `no-inspect` Appears Important For Reaching Object `1` On The Final Probe

This is the strongest positive closure from the pass.

Compare the final probe outcomes:

- exact replay with opening `no-inspect` preserved -> `0870:000003E4`
- Variant B with opening `no-inspect` preserved -> `0870:00001162`
- Variant A with opening `no-inspect` removed -> `0868:0002B824`

That pattern strongly suggests:

- opening `no-inspect` is important for keeping the later inspect-after-inspect probe on the object-`1` side

Even though it did not recreate `0008`, it still materially changed the branch family away from flat `0868`.

### 3. The Earlier `inspect-after-no-inspect` Step Is Not Required For A Later Object-`1` Probe

Variant B removed the earlier inspect-after-no-inspect step entirely:

- `no-inspect`
- `no-inspect`
- `inspect`
- final inspect-after-inspect probe

The final probe still landed on object `1`:

- `0870:00001162`

So that earlier inspect-after-no-inspect step is not necessary for the broader object-`1` outcome.

What it may still influence is the rarer choice between:

- `0008`
- and `0870`

But it is no longer a candidate requirement for merely reaching object `1`.

### 4. Removing The Opening `no-inspect` Breaks The Final Probe Back To Flat `0868`

Variant A is the clearest negative control in the pass.

Sequence:

- inspect
- inspect
- inspect
- final inspect-after-inspect probe

Outcome of the final probe:

- flat `0868`

So “later inspect-after-inspect by itself” is not enough.

The object-`1` side at the final probe appears to need something contributed by an earlier no-inspect opening.

### 5. The Immediate Problem Has Shifted From `0008` Reproduction To `0870` Versus `0008` Drift

After this pass, the best model is:

- opening `no-inspect` helps keep the final probe in an object-`1` family
- but the specific rare `0008` outcome still drifts away into the more common `0870` branch

That means the next useful question is no longer just:

- “how do we reach object `1`?”

We can do that.

The sharper question is now:

- what makes an opening-no-inspect object-`1` probe choose `0008` instead of `0870`?

## Practical Interpretation

This pass did not give a stable `0008` replay, but it materially improved the dependency picture.

We now know:

- the rare branch is not stable on demand yet
- opening `no-inspect` is an important ingredient for the later object-`1` probe
- the earlier inspect-after-no-inspect step is not required for that broader object-`1` outcome

That is useful because it narrows the next search to the smaller problem of family drift inside an already object-`1`-capable shape.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Opening-`no-inspect` Object-`1` Drift Pass**

Focus:

- keep an opening `no-inspect` in place
- cluster short object-`1`-capable shapes that end on inspect-after-inspect
- compare repeated runs of:
  - `no-inspect -> inspect -> inspect -> inspect`
  - `no-inspect -> no-inspect -> inspect -> inspect`
- test whether the final probe drifts between:
  - `0870`
  - `0008`
  - or flat `0868`

## Bottom Line

The rare `0008` branch still did not replay on demand.

But this pass showed that opening `no-inspect` is important for keeping the final inspect-after-inspect probe on the object-`1` side, while the earlier inspect-after-no-inspect step is not required for that broader outcome.
