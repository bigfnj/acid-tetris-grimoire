# `0x26D000` Rare-`0008` Reproduction Pass

Date: 2026-04-17

## Summary

This pass directly replayed the two known `0008` contexts and compared them against adjacent near-miss shapes.

Main result:

- the known-positive replay pair did **not** reproduce `0008`
- a near-miss control **did** reproduce `0008`
- the successful reproduction was:
  - `inspect-after-inspect`
  - in a mixed batch that began with `no-inspect` then `inspect`

So the rare `0008` family is not anchored to the two originally observed positive slots themselves.

It appears to be tied to a rarer broader run family that can emerge later inside a mixed opening sequence.

## New Owned Artifacts

- [26d000-rare-0008-reproduction-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-rare-0008-reproduction/26d000-rare-0008-reproduction-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T203355Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T203620Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-rare-0008-known-replay`
- `selector-26d000-rare-0008-near-miss-controls`

## Results

### Known Positive Replay Batch

- `26D000_0008_known_inspect_open_r1`
  - `0868:0002B87C`
  - object `3`
  - flat `0868`
- `26D000_0008_known_no_inspect_after_inspect_r2`
  - `0868:000253F2`
  - object `3`
  - flat `0868`
- `26D000_0008_known_inspect_open_r3`
  - `0868:0002B883`
  - object `3`
  - flat `0868`
- `26D000_0008_known_no_inspect_after_inspect_r4`
  - `0868:00025424`
  - object `3`
  - flat `0868`

### Near-Miss Control Batch

- `26D000_0008_control_no_inspect_open_c1`
  - `0868:0002B880`
  - object `3`
  - flat `0868`
- `26D000_0008_control_inspect_after_no_inspect_c2`
  - `0868:00025395`
  - object `3`
  - flat `0868`
- `26D000_0008_control_inspect_open_c3`
  - `0868:0002B824`
  - object `3`
  - flat `0868`
- `26D000_0008_control_inspect_after_inspect_c4`
  - `0008:00001068`
  - object `1`
  - selector `0008`

## Findings

### 1. The Originally Known `0008` Contexts Did Not Reproduce

This is the strongest negative result in the pass.

The replay batch directly retried:

- opening inspect
- immediate no-inspect-after-inspect

Both contexts had previously been associated with `0008` in owned evidence.

This time, all four replayed runs landed flat `0868`.

So the rare `0008` family is **not** reliably reproduced just by replaying those original slot shapes.

### 2. A Near-Miss Control Reproduced `0008`

This is the most important positive result.

The only `0008` hit in the pass was:

- `26D000_0008_control_inspect_after_inspect_c4`
  - `0008:00001068`

That run had been designed as a near miss, not as one of the original positives.

So the rare family is not centered on the historical positive labels themselves.

It can emerge from a broader mixed-sequence run family that the earlier positive contexts happened to sample.

### 3. `0008` Now Looks More Compatible With A Later Inspect-After-Inspect State

The successful `0008` reproduction happened at:

- a later inspect-after-inspect slot

within a mixed batch that had already seen:

- no-inspect opening
- inspect-after-no-inspect

That is more specific than “inspect causes `0008`,” but it is also more informative than the previous model.

The current best interpretation is:

- `0008` may require a broader mixed-history run family
- and one productive manifestation of that family is a later inspect-after-inspect slot

### 4. The Repeatable `0870` Branch Was Not The One That Reappeared Here

Another useful detail is what **didn't** happen.

The near-miss control that had previously been linked with bounded `0870`:

- `inspect-after-no-inspect`

did not return to `0870` in this pass.

It stayed flat `0868`.

That means the rare-`0008` branch and the repeatable `0870` branch are both sensitive to broader run-family conditions, not just the immediate local labels we assigned them earlier.

### 5. The Rare Family Is Real, But It Is Not Slot-Deterministic

After this pass, the best owned statement is:

- `0008` is real
- it can be reproduced
- but not by naively replaying the first contexts where we saw it

This is a good closure result because it prevents us from overfitting to the first two sightings.

The real selector is now more likely to be:

- a broader run-history family
- or a hidden branch condition correlated with that family

rather than a simple named slot such as:

- opening inspect
- first no-inspect after inspect

## Practical Interpretation

This pass replaces the “replay the original positives” model with a better one.

We now know:

- the original positive contexts are not sufficient
- a nearby control can still reproduce `0008`
- later inspect-after-inspect is currently the best owned live reproduction shape

That means the next pass should stop privileging the original `0008` sightings and instead refine around the newly confirmed successful control family.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Inspect-After-Inspect `0008` Refinement Pass**

Focus:

- hold the new successful mixed-control shape fixed
- vary only the immediate predecessors around the successful `inspect-after-inspect` slot
- test whether `0008` depends on:
  - opening no-inspect
  - the preceding inspect-after-no-inspect slot
  - or simply arriving at a later inspect-after-inspect state

## Bottom Line

The rare `0008` family did reproduce, but not in the original known-positive replay lane.

It reappeared in a control shape on a later inspect-after-inspect slot, which means the next strongest move is to refine around that newly confirmed live branch.
