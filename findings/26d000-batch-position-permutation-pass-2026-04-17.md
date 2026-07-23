# `0x26D000` Batch-Position Permutation Pass

Date: 2026-04-17

## Summary

This pass tested the strongest remaining hypothesis from the prior single-budget isolation:

- whether the `0x26D000` fallback mainly follows absolute final position

Instead of one long batch, this pass used two short dedicated four-run batches:

- one with an inspect-after-no-inspect final probe
- one with a no-inspect-after-no-inspect final probe

Each short batch also moved a related local predecessor pattern into an early slot so we could compare:

- early local pattern
- versus final-slot placement

Main result:

- the fallback does **not** require absolute final position
- early slot-`2` probes already flipped into object-`1` families
- final inspect stayed flat in the short inspect-tail batch
- final no-inspect still fell back in the short no-inspect-tail batch

## New Owned Artifacts

- [26d000-batch-position-permutation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-batch-position-permutation/26d000-batch-position-permutation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T194349Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T194616Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-position-permutation-inspect-tail`
- `selector-26d000-position-permutation-no-inspect-tail`

## Results

### Inspect-Tail Short Batch

- `26D000_no_inspect_seed_a1`
  - `0868:00025179`
  - object `3`
  - flat-side family
- `26D000_no_inspect_early_probe_a2`
  - `0870:00000BFB`
  - object `1`
  - bounded fallback family
- `26D000_no_inspect_seed_a3`
  - `0870:00000C02`
  - object `1`
  - bounded fallback family
- `26D000_inspect_final_probe_a4`
  - `0868:00025790`
  - object `3`
  - flat-side family

### No-Inspect-Tail Short Batch

- `26D000_inspect_seed_b1`
  - `0868:0002482E`
  - object `3`
  - flat-side family
- `26D000_no_inspect_early_probe_b2`
  - `0008:00001491`
  - object `1`
  - distinct object-`1` fallback family
- `26D000_no_inspect_seed_b3`
  - `0868:0002B880`
  - object `3`
  - flat-side family
- `26D000_no_inspect_final_probe_b4`
  - `0870:00000425`
  - object `1`
  - bounded fallback family

## Findings

### 1. Absolute Final Position Is Not Required

This is the most important closure from the pass.

In both short batches, the early slot-`2` probe already flipped:

- inspect-tail batch slot `2` -> `0870:00000BFB`
- no-inspect-tail batch slot `2` -> `0008:00001491`

So the seam cannot be explained as:

- “the last run in the batch falls back”

That hypothesis is now closed.

### 2. Final Inspect Does Not Reproduce The Fallback In A Short Batch

In the inspect-tail batch:

- early no-inspect-after-no-inspect already fell back
- the final inspect-after-no-inspect probe returned to flat `0868`

That directly weakens the older interpretation that late inspected runs were special.

The fallback is not simply “whatever lands last,” and it is not simply “inspect in the last slot.”

### 3. Final No-Inspect Still Reproduces The Fallback

In the no-inspect-tail batch:

- the final no-inspect-after-no-inspect probe fell back to `0870:00000425`

So a no-inspect tail is still a productive way to hit the object-`1` side, even in a short four-run batch.

That means the prior long-batch result was not just an artifact of needing seven or eight runs of accumulated history.

### 4. Local No-Inspect Prehistory Matters More Than Absolute Position

The two short batches together point to a narrower interpretation:

- no-inspect runs with recent no-inspect prehistory are especially likely to flip into object `1`
- the flip can happen early
- once it happens, a nearby no-inspect continuation can remain on the object-`1` side

This is visible in the inspect-tail batch:

- slot `2` no-inspect-after-no-inspect -> bounded `0870`
- slot `3` no-inspect-after-no-inspect -> bounded `0870`

That is much more suggestive of a local carry or no-inspect cluster effect than a pure absolute-position effect.

### 5. There Is More Than One Object-`1` Fallback Family

The no-inspect-tail batch produced an especially useful anomaly:

- early slot `2` -> `0008:00001491`

This is not the usual bounded `0870` fallback family.

So the owned `0x26D000` seam can feed at least two object-`1` landing families:

- bounded `0870`
- selector `0008`

That means the remaining closure problem is not just “flat versus bounded.”

It is now:

- which short no-inspect histories select which object-`1` family

## Practical Interpretation

This pass replaces the late-position model with a stronger local-history model.

We now know:

- short batches are sufficient to reproduce the seam
- early local predecessor patterns can already flip `0x26D000`
- final inspect is not enough by itself
- no-inspect clustering is a stronger steering signal than absolute slot number
- object-`1` fallout is not single-family

That is a good narrowing, because it turns the next step from vague carry hunting into explicit no-inspect history classification.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` No-Inspect Cluster Characterization Pass**

Focus:

- keep `0x26D000` fixed
- run short no-inspect-heavy sequences deliberately
- compare:
  - isolated no-inspect
  - two-run no-inspect clusters
  - three-run no-inspect clusters
  - inspect interruptions between no-inspect runs
- classify which local histories land in:
  - flat `0868`
  - bounded `0870`
  - selector `0008`

## Bottom Line

`0x26D000` does not need to be last in the batch to flip.

The seam responds earlier to short no-inspect histories, final inspect alone is not enough, and the object-`1` side now clearly includes at least two landing families.
