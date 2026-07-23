# `0x26D000` Bridge-Slot Object-`1` Drift Pass

Date: 2026-04-17

## Summary

This pass followed the opening-`no-inspect` drift closure:

- the bridge slot looked like the main live selector seam under the surviving short lanes

The narrow question here was:

- does selector choice actually happen at the bridge slot itself, or is the bridge just inheriting a family chosen one slot earlier

To answer that, the pass shortened both surviving lanes down to their bridge prefixes:

- Shape A: `no-inspect -> inspect -> inspect`
- Shape B: `no-inspect -> no-inspect -> inspect`

Main result:

- Shape A does **not** keep object-`1` behavior at the bridge; its live drift stays one slot earlier
- Shape B can still place rare `0008` directly on the bridge
- but Shape B bridge behavior is still unstable and collapsed to flat `0868` on the second repeat

So the best new closure is:

- Shape B remains the strongest direct bridge candidate
- Shape A is now better understood as a pre-bridge family-drift lane rather than a true bridge lane

## Tooling Correction

The first attempt at this pass exposed a small harness bug:

- child probes launched from the batch harness could silently fall back to `.tools/bin/dosbox-x`
- that non-debug runtime never reached the live VRT prompt needed for selector capture

I corrected [run_branch_family_batch.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/run_branch_family_batch.py:1) so it now propagates the selected `DOSBOX_X_BIN` value into every child probe.

The invalid first-attempt batch summaries were preserved but not used as evidence:

- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T212730Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T213107Z/batch-summary.json)

The corrected evidence for this pass is:

- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T213533Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T213910Z/batch-summary.json)

## New Owned Artifacts

- [26d000-bridge-slot-object1-drift-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-bridge-slot-object1-drift/26d000-bridge-slot-object1-drift-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T213533Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T213910Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-bridge-slot-drift-shape-a`
- `selector-26d000-bridge-slot-drift-shape-b`

## Results

### Shape A: `no-inspect -> inspect -> inspect`

#### Repeat 1

- `26D000_bridge_a1_no_inspect`
  - `0868:0002B882`
  - object `3`
- `26D000_bridge_a2_inspect_after_no_inspect`
  - `0008:000014A8`
  - object `1`
- `26D000_bridge_a3_inspect_bridge`
  - `0868:0002620C`
  - object `3`

#### Repeat 2

- `26D000_bridge_a4_no_inspect`
  - `0868:0002B81E`
  - object `3`
- `26D000_bridge_a5_inspect_after_no_inspect`
  - `0870:00000C4B`
  - object `1`
- `26D000_bridge_a6_inspect_bridge`
  - `0868:0002B824`
  - object `3`

### Shape B: `no-inspect -> no-inspect -> inspect`

#### Repeat 1

- `26D000_bridge_b1_no_inspect`
  - `0868:0002B828`
  - object `3`
- `26D000_bridge_b2_no_inspect_again`
  - `0868:0002B824`
  - object `3`
- `26D000_bridge_b3_inspect_bridge`
  - `0008:00001E05`
  - object `1`

#### Repeat 2

- `26D000_bridge_b4_no_inspect`
  - `0868:0002B882`
  - object `3`
- `26D000_bridge_b5_no_inspect_again`
  - `0868:0002B880`
  - object `3`
- `26D000_bridge_b6_inspect_bridge`
  - `0868:0002B880`
  - object `3`

## Findings

### 1. Shape A Does Not Preserve Object-`1` Behavior To The Bridge

This is the clearest correction from the pass.

In Shape A, the live object-`1` slot is the second slot:

- `a2` -> `0008:000014A8`
- `a5` -> `0870:00000C4B`

But both Shape A bridge slots collapsed back to flat `0868`:

- `a3` -> `0868:0002620C`
- `a6` -> `0868:0002B824`

So Shape A is not a true bridge-lane reproduction path.

It is better understood as:

- a pre-bridge drift lane where the family decision is already visible one slot earlier

### 2. Shape B Is The Only Shortened Lane That Reproduced `0008` Directly On The Bridge

This is the strongest positive signal in the corrected pass.

Shape B repeat `1` ended with:

- `b3` -> `0008:00001E05`

That is a direct bridge-slot `0008` reproduction under the shortened lane.

So Shape B still owns the best current evidence that the bridge itself can be a live rare-family entry point.

### 3. Shape B Bridge Behavior Is Still Not Stable

The second Shape B repeat collapsed completely:

- `b4` -> `0868:0002B882`
- `b5` -> `0868:0002B880`
- `b6` -> `0868:0002B880`

That means Shape B is the strongest direct bridge lane, but it is not yet a reliable one.

The selector choice is still drifting between:

- direct `0008` bridge entry
- and full collapse to flat `0868`

### 4. The Current Frontier Splits Cleanly Into Two Different Roles

After this pass, the two lanes now have clearer responsibilities:

- Shape A is useful for studying the earlier pre-bridge family split
- Shape B is useful for studying direct bridge-slot rare-family stabilization

That is helpful because it means the next pass does not need to keep both lanes coupled in the same way.

### 5. The Harness Fix Matters Operationally

This pass also closed a tooling risk.

Without the child-env propagation fix, batch runs could appear to produce structural negative results while actually running on the wrong DOSBox binary.

That is now closed in the harness, and future batch comparisons should be safer.

## Practical Interpretation

This pass sharpened the frontier instead of broadening it.

We now know:

- Shape A loses object-`1` behavior before the bridge
- Shape B can still place `0008` directly on the bridge
- but Shape B bridge selection remains unstable across repeats

So the best next move is no longer a generic bridge-slot sweep. It is a Shape-B-specific stabilization pass.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Shape-B Bridge Family Stabilization Pass**

Focus:

- hold the shortened Shape B lane fixed:
  - `no-inspect -> no-inspect -> inspect`
- repeat it enough times to cluster direct bridge outcomes
- test whether the bridge slot drifts between:
  - `0008`
  - flat `0868`
  - and any reappearance of `0870`
- treat Shape A as a side reference only when needed for comparison against pre-bridge drift

## Bottom Line

Shortening the lanes changed the model in a useful way.

Shape A no longer looks like a bridge lane at all; its live family split is one slot earlier. Shape B is still the only shortened lane that can reproduce rare `0008` directly on the bridge, but that bridge entry is not stable yet.
