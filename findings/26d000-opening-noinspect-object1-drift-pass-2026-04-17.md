# `0x26D000` Opening-`no-inspect` Object-`1` Drift Pass

Date: 2026-04-17

## Summary

This pass followed the previous refinement result:

- preserving an opening `no-inspect` keeps the later inspect-after-inspect probe on the object-`1` side more often than all-inspect openings

The narrow question here was:

- which opening-`no-inspect` short shape is actually carrying the live object-`1` seam

Two candidate lanes were repeated twice:

- Shape A: `no-inspect -> inspect -> inspect -> inspect`
- Shape B: `no-inspect -> no-inspect -> inspect -> inspect`

Main result:

- Shape A is unstable and sheds object-`1` behavior into the middle slots rather than the final probe
- Shape B is stronger: its bridge slot reproduced object `1` in both repeats, and its final probe reached object `1` in one repeat

So the strongest new closure is:

- the live selector seam under opening-`no-inspect` histories is now best understood as a **bridge-slot drift problem**
- not a pure final-probe problem

## New Owned Artifacts

- [26d000-opening-noinspect-object1-drift-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-opening-noinspect-object1-drift/26d000-opening-noinspect-object1-drift-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T210704Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T211152Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-opening-noinspect-drift-shape-a`
- `selector-26d000-opening-noinspect-drift-shape-b`

## Results

### Shape A: `no-inspect -> inspect -> inspect -> inspect`

#### Repeat 1

- `26D000_drift_a1_no_inspect`
  - `0868:00023923`
  - object `3`
- `26D000_drift_a2_inspect_after_no_inspect`
  - `0870:00000BFE`
  - object `1`
- `26D000_drift_a3_inspect_bridge`
  - `0868:00023979`
  - object `3`
- `26D000_drift_a4_inspect_probe`
  - `0868:0002B821`
  - object `3`

#### Repeat 2

- `26D000_drift_a5_no_inspect`
  - `0868:00023923`
  - object `3`
- `26D000_drift_a6_inspect_after_no_inspect`
  - `0008:00000C75`
  - object `1`
- `26D000_drift_a7_inspect_bridge`
  - `0870:00000430`
  - object `1`
- `26D000_drift_a8_inspect_probe`
  - `0868:0002B559`
  - object `3`

### Shape B: `no-inspect -> no-inspect -> inspect -> inspect`

#### Repeat 1

- `26D000_drift_b1_no_inspect`
  - `0868:000252AC`
  - object `3`
- `26D000_drift_b2_no_inspect_again`
  - `0868:0002B82A`
  - object `3`
- `26D000_drift_b3_inspect_bridge`
  - `0008:000014E4`
  - object `1`
- `26D000_drift_b4_inspect_probe`
  - `0868:00023927`
  - object `3`

#### Repeat 2

- `26D000_drift_b5_no_inspect`
  - `0868:0002515B`
  - object `3`
- `26D000_drift_b6_no_inspect_again`
  - `0868:0002B824`
  - object `3`
- `26D000_drift_b7_inspect_bridge`
  - `0870:00000818`
  - object `1`
- `26D000_drift_b8_inspect_probe`
  - `0870:000003E4`
  - object `1`

## Findings

### 1. Shape A Is Not A Stable Final-Probe Object-`1` Lane

This needs to be stated first.

Both Shape A final probes stayed flat:

- `a4` -> `0868:0002B821`
- `a8` -> `0868:0002B559`

But the middle slots did not stay flat:

- `a2` -> bounded `0870`
- `a6` -> rare `0008`
- `a7` -> bounded `0870`

So Shape A is live, but it is not preserving object-`1` behavior all the way to the final probe.

### 2. Shape B Keeps The Bridge Slot Live In Both Repeats

This is the strongest positive signal from the pass.

The bridge slot in Shape B hit object `1` in both repeats:

- `b3` -> `0008:000014E4`
- `b7` -> `0870:00000818`

That means the opening-`no-inspect` plus second-`no-inspect` lane is reliably holding a live selector seam at the bridge slot, even though the specific family still drifts.

### 3. Shape B Can Still Carry The Final Probe To Object `1`

Shape B repeat `2` is the best single result in the pass:

- `b8` -> `0870:000003E4`

Repeat `1` still collapsed:

- `b4` -> `0868:00023927`

So Shape B does not make the final probe stable yet, but it is clearly stronger than Shape A for keeping the late short sequence on the object-`1` side.

### 4. Rare `0008` Has Shifted Toward Bridge-Slot Contexts

Rare `0008` resurfaced twice in this pass:

- `a6` -> inspect-after-no-inspect in Shape A repeat `2`
- `b3` -> inspect bridge in Shape B repeat `1`

Neither reproduction was the final probe.

That matters because it changes the modeling target:

- `0008` is still real
- but it currently appears to surface most naturally at the bridge slot or immediate pre-probe slot inside opening-`no-inspect` histories

### 5. The Final Probe Looks Downstream Of Bridge-Slot Family Selection

This is the best overall interpretation after the two batches.

The evidence is:

- Shape A allows object-`1` drift in the middle, but still loses the final probe
- Shape B keeps the bridge slot on object `1` in both repeats and carries the final probe to object `1` once

So the main live seam is no longer best described as:

- “what makes the final inspect-after-inspect probe choose `0008` or `0870`?”

Instead it is better described as:

- “what family does the bridge slot choose under opening-`no-inspect` histories, and when does that family carry forward to the final probe?”

## Practical Interpretation

This pass did not stabilize the final probe, but it clarified where the real selector pressure lives.

We now know:

- Shape A is not the best lane for late object-`1` preservation
- Shape B is stronger than Shape A
- the bridge slot is the most productive object-`1`-capable seam under these opening-`no-inspect` histories
- rare `0008` and bounded `0870` are both still live there

That means the next pass should stop treating the final probe as the primary frontier and should instead characterize the bridge slot directly.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Bridge-Slot Object-`1` Drift Pass**

Focus:

- keep the opening-`no-inspect` histories fixed
- cluster repeated runs around the bridge slot in both surviving lanes
- classify whether the bridge slot drifts between:
  - `0008`
  - `0870`
  - flat `0868`
- treat the final inspect-after-inspect probe as a downstream confirmation slot rather than the primary selector seam

## Bottom Line

The opening-`no-inspect` lanes are not equal.

Shape A sheds object-`1` behavior into middle slots and loses the final probe, while Shape B keeps the bridge slot live in both repeats and can still carry the final probe to object `1`. The bridge slot is now the strongest live selector seam.
