# Single-Budget `0x26D000` Carry Isolation Pass

Date: 2026-04-17

## Summary

This pass isolated the last unstable owned budget:

- `0x26D000`

It removed `0x26C800` entirely and ran only `0x26D000` in one dedicated batch with:

- alternating inspected and uninspected order
- consecutive inspect-only repeats
- consecutive no-inspect repeats

Main result:

- the bounded fallback reproduced even in a pure `0x26D000` batch
- it occurred only once, on the final run of the batch
- it was not inspect-specific, because this time the fallback appeared on the last no-inspect run rather than on an inspected run

## New Owned Artifacts

- [single-budget-26d000-carry-isolation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/single-budget-26d000-carry-isolation/single-budget-26d000-carry-isolation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T192723Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-26d000-carry-isolation`

It runs `0x26D000` eight times:

- inspect then no inspect
- no inspect then inspect
- inspect twice in a row
- no inspect twice in a row

## Results

### Alternating Block

- `26D000_inspect_first_a`
  - `0868:00025154`
  - object `3`
  - flat selector `0868`
- `26D000_no_inspect_second_a`
  - `0868:0002B826`
  - object `3`
  - flat selector `0868`
- `26D000_no_inspect_first_b`
  - `0868:000253C7`
  - object `3`
  - flat selector `0868`
- `26D000_inspect_second_b`
  - `0868:00025389`
  - object `3`
  - flat selector `0868`

### Inspect-Only Repeats

- `26D000_inspect_repeat_c1`
  - `0868:000253E4`
  - object `3`
  - flat selector `0868`
- `26D000_inspect_repeat_c2`
  - `0868:0002482E`
  - object `3`
  - flat selector `0868`

### No-Inspect Repeats

- `26D000_no_inspect_repeat_d1`
  - `0868:00023A3D`
  - object `3`
  - flat selector `0868`
- `26D000_no_inspect_repeat_d2`
  - `0870:000003DB`
  - object `1`
  - bounded selector `0870`

## Findings

### 1. The `0x26D000` Fallback Reproduces Without `0x26C800`

This closes an important uncertainty from the previous pass.

The bounded fallback is not dependent on a mixed `0x26C800` plus `0x26D000` batch shape.

In this dedicated `0x26D000` run family:

- seven runs landed flat `0868` object `3`
- one run landed bounded `0870` object `1`

So the seam is genuinely owned at `0x26D000` itself.

### 2. The Fallback Is Not Inspect-Specific

In the previous repeat pass, the only bounded fallback happened on:

- the final inspected run

In this pass, the only bounded fallback happened on:

- the final no-inspect run

That is a strong negative result against a simple probe-style explanation.

The surviving flip is no longer plausibly described as:

- “inspection causes the fallback”
- or “no inspection causes the fallback”

### 3. The Strongest Current Signal Is Late Batch Position

In this pass:

- runs `1` through `7` all stayed flat `0868`
- run `8` alone fell back to bounded `0870`

Combined with the previous pass, where the only bounded fallback appeared on the final `0x26D000` run there as well, the strongest current interpretation is:

- the surviving seam is sensitive to late sequence position
- or to another cumulative carry that correlates with late position

That is stronger than the older interpretations based on:

- inspect-versus-no-inspect style
- local pair order

### 4. Same-Style Repeats Alone Do Not Immediately Trigger The Fallback

This pass also tested whether consecutive same-style runs would trigger the seam directly.

Observed:

- inspect-only repeat `c1` -> flat `0868`
- inspect-only repeat `c2` -> flat `0868`
- no-inspect repeat `d1` -> flat `0868`
- no-inspect repeat `d2` -> bounded `0870`

So consecutive style alone is not sufficient.

The fallback still appears deeper into the overall sequence, not simply on the second consecutive inspect or second consecutive no-inspect run.

### 5. The Remaining Question Has Shifted Again

After this pass, the best open question is no longer:

- whether `0x26D000` can flip in isolation

That is now answered: yes, it can.

The next question is:

- whether the fallback follows absolute run position in the batch
- or whether it depends on a narrower local prehistory near the end

## Practical Interpretation

This pass materially tightened the remaining instability model.

We now know:

- `0x26D000` is the real seam
- the seam survives even in a pure single-budget batch
- the flip is not probe-style-specific
- the best current predictor is “very late in the batch,” not “inspect” or “no inspect”

That means the next pass should stop varying probe style broadly and instead permute run position deliberately.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Batch-Position Permutation Pass**

Focus:

- run the same `0x26D000` variants in shorter batches with different ordering
- move the currently failing no-inspect tail case earlier
- move an inspect case into the final slot in a separate batch
- test whether the bounded fallback follows:
  - absolute last position
  - last two positions
  - or a specific local predecessor pattern

## Bottom Line

The bounded `0x26D000` fallback reproduces in isolation, so it is not an artifact of carrying `0x26C800` ahead of it.

But it still appears only at the very end of the batch, and it changed from an inspected run to an uninspected run, which makes late batch position the strongest current explanation.
