# `0x26D000` Shape-B Bridge Family Stabilization Pass

Date: 2026-04-17

## Summary

This pass followed the direct bridge-slot closure:

- Shape B was the only shortened lane that could still reproduce rare `0008` directly on the bridge
- but that bridge result was unstable across two repeats

The narrow question here was:

- if we hold Shape B fixed and repeat it deeper, does the bridge stabilize around one family

The fixed lane was:

- `no-inspect -> no-inspect -> inspect`

Main result:

- the bridge did not stabilize around `0008`
- instead it decayed across repeats:
  - repeat `1` bridge -> `0008`
  - repeat `2` bridge -> `0870`
  - repeats `3` and `4` bridge -> flat `0868`

So the strongest new closure is:

- Shape B is not a stable direct bridge selector
- it looks more like a carry-decay lane whose family weakens as the repeated history grows

## New Owned Artifacts

- [26d000-shape-b-bridge-family-stabilization-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-shape-b-bridge-family-stabilization/26d000-shape-b-bridge-family-stabilization-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T215837Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-26d000-shape-b-bridge-stabilization`

## Results

### Repeat 1

- `26D000_shape_b_stab_r1_no_inspect`
  - `0868:00023A51`
  - object `3`
- `26D000_shape_b_stab_r1_no_inspect_again`
  - `0870:00000430`
  - object `1`
- `26D000_shape_b_stab_r1_bridge`
  - `0008:000014A8`
  - object `1`

### Repeat 2

- `26D000_shape_b_stab_r2_no_inspect`
  - `0868:0002B826`
  - object `3`
- `26D000_shape_b_stab_r2_no_inspect_again`
  - `0868:00024821`
  - object `3`
- `26D000_shape_b_stab_r2_bridge`
  - `0870:00000810`
  - object `1`

### Repeat 3

- `26D000_shape_b_stab_r3_no_inspect`
  - `0868:000239F8`
  - object `3`
- `26D000_shape_b_stab_r3_no_inspect_again`
  - `0868:00023AD0`
  - object `3`
- `26D000_shape_b_stab_r3_bridge`
  - `0868:0002B893`
  - object `3`

### Repeat 4

- `26D000_shape_b_stab_r4_no_inspect`
  - `0868:00023AC6`
  - object `3`
- `26D000_shape_b_stab_r4_no_inspect_again`
  - `0868:000258E0`
  - object `3`
- `26D000_shape_b_stab_r4_bridge`
  - `0868:0002B885`
  - object `3`

## Findings

### 1. The Bridge Shows An Ordered Decay, Not A Stable Rare Family

This is the main result.

Across the four fixed repeats, the bridge walked through three family states:

- repeat `1` -> `0008`
- repeat `2` -> `0870`
- repeats `3` and `4` -> `0868`

That pattern is much more informative than a simple “sometimes `0008`, sometimes not.”

It suggests:

- the lane is carrying some state that weakens with repeat depth

### 2. Shape B Does Not Converge On `0008`

If Shape B were a stable direct rare-family bridge lane, the longer cluster should have given either:

- repeated `0008`
- or at least a mixed but recurring `0008`

It did not.

`0008` happened once, at the very first bridge only.

So the rare family now looks like:

- an early-depth phenomenon inside the Shape B lane
- not the long-run steady state of that lane

### 3. Bounded `0870` Appears As A Transitional Family

Repeat `2` bridge landed at:

- `0870:00000810`

That sits naturally between the first rare `0008` bridge and the later flat `0868` bridges.

So the best current family order is:

- `0008`
- then `0870`
- then flat `0868`

### 4. Carry Leakage Can Appear Before The Bridge Fully Collapses

The second slot in repeat `1` is also important:

- `r1_no_inspect_again` -> `0870:00000430`

After that, later second-slot no-inspect runs returned to flat `0868`.

That suggests the decaying family is not strictly confined to the bridge:

- it can surface earlier in the lane before the bridge fully loses the object-`1` side

### 5. The Problem Has Shifted Again

The useful question is no longer:

- “how do we stabilize Shape B around `0008`?”

The sharper question is now:

- “what part of the repeated Shape B history causes the carry to decay from `0008` to `0870` to `0868`?”

That is a better next-step framing because the cluster now has clear order.

## Practical Interpretation

This pass made the Shape B frontier more structured.

We now know:

- Shape B is not a stable direct bridge lane
- it does carry rare-family behavior at shallow depth
- that behavior appears to decay with repeat depth in an ordered way

So the next pass should isolate whether the decay depends on:

- absolute repeat position
- uninterrupted repetition
- or accumulated lane history that can be reset or interrupted

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Shape-B Carry-Decay Isolation Pass**

Focus:

- keep Shape B as the primary lane
- compare:
  - a fresh short Shape B batch
  - a restarted mid-depth Shape B batch
  - an interrupted Shape B batch with an inserted non-Shape-B reset or pause shape
- test whether the bridge returns to:
  - `0008`
  - `0870`
  - or stays collapsed at `0868`

## Bottom Line

The fixed Shape B lane did not stabilize around rare `0008`.

Instead, the bridge decayed across the four repeats from `0008` to `0870` to flat `0868`, which makes carry-decay the best current model for this runtime seam.
