# Boundary Orientation Repeat Pass

Date: 2026-04-17

## Summary

This pass repeated the two remaining split-sensitive budgets:

- `0x26C800`
- `0x26D000`

Each budget was run twice with reversed inspected-versus-uninspected ordering to test whether the seam orientation follows:

- probe style
- local pair ordering
- or a broader hidden state carry

Main result:

- `0x26C800` no longer split in this batch and landed flat `0868` object `3` in all four repeats
- `0x26D000` landed flat `0868` object `3` in three of four repeats and only fell back to bounded `0870` object `1` on the final inspected run
- the seam does not follow a simple inspected-versus-uninspected rule or a simple first-versus-second ordering rule

## New Owned Artifacts

- [boundary-orientation-repeat-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/boundary-orientation-repeat/boundary-orientation-repeat-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T191803Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-boundary-orientation-repeat`

It repeats each budget four times:

- inspect then no inspect
- then no inspect then inspect

## Results

### `0x26C800`

- inspect first A
  - `0868:00025424`
  - object `3`
  - flat selector `0868`
- no inspect second A
  - `0868:0002B82A`
  - object `3`
- no inspect first B
  - `0868:0002B826`
  - object `3`
- inspect second B
  - `0868:00023926`
  - object `3`
  - flat selector `0868`

### `0x26D000`

- inspect first A
  - `0868:0002521A`
  - object `3`
  - flat selector `0868`
- no inspect second A
  - `0868:0002B882`
  - object `3`
- no inspect first B
  - `0868:000252A7`
  - object `3`
- inspect second B
  - `0870:00000C95`
  - object `1`
  - bounded selector `0870`

## Findings

### 1. `0x26C800` No Longer Behaves Like A Split-Sensitive Budget

This is the biggest change from the previous pass.

The prior boundary batch had:

- inspected `0x26C800` -> bounded `0870`
- uninspected `0x26C800` -> flat `0868`

But in this repeat batch, all four `0x26C800` runs landed in flat `0868` object `3`.

That means `0x26C800` is not a stable bidirectional seam in owned evidence anymore.

Under the current repeat conditions, it behaves like part of the flat-side family.

### 2. `0x26D000` Still Sits Inside The Real Instability Zone

`0x26D000` remained mixed:

- three runs landed flat `0868` object `3`
- one run landed bounded `0870` object `1`

So the instability did not disappear.

It narrowed upward.

The last clearly unstable owned budget is now `0x26D000`, not `0x26C800`.

### 3. The Seam Does Not Follow A Simple Inspect-Versus-No-Inspect Rule

If probe style alone controlled the seam, we would expect inspected runs to cluster together and uninspected runs to cluster together.

That did not happen.

Owned evidence from this batch:

- `0x26C800`
  - both inspected runs -> flat `0868`
  - both uninspected runs -> flat `0868`
- `0x26D000`
  - first inspected run -> flat `0868`
  - second inspected run -> bounded `0870`
  - both uninspected runs -> flat `0868`

So probe style can still matter in some contexts, but it is not the whole causal rule.

### 4. The Seam Also Does Not Follow A Simple Local Pair-Order Rule

This batch reversed the local order inside each budget pair:

- inspect then no inspect
- no inspect then inspect

If local pair order were the main driver, we would expect the first or second slot to behave consistently across both budgets.

Instead:

- `0x26C800` stayed flat in every slot
- `0x26D000` only flipped on the final run of the whole batch

That makes the current best interpretation:

- there may be a broader late-sequence or run-history carry
- or there is genuine stochastic drift inside the remaining seam

In either case, the hidden condition is more global than just the local inspected/uninspected ordering within one pair.

### 5. The Remaining Uncertainty Has Shifted To A Single Budget

After this pass, the seam is best described as:

- `0x26C800`
  - effectively flat-side under repeat conditions
- `0x26D000`
  - still unstable
- `0x26D800`
  - converged flat-side from the prior pass

So the unresolved band has tightened again.

The practical target is now:

- `0x26D000`

## Practical Interpretation

This pass materially simplifies the search space.

We no longer need to treat both `0x26C800` and `0x26D000` as equally unstable.

Instead:

- `0x26C800` now looks mostly absorbed into the flat-side family
- `0x26D000` remains the only owned budget still showing a live family flip

That points away from another broad interval sweep and toward a single-budget isolation pass.

## Recommended Next Move

The strongest next move is now:

- **Single-Budget `0x26D000` Carry Isolation Pass**

Focus:

- run `0x26D000` in shorter dedicated batches rather than behind `0x26C800`
- compare:
  - inspect first
  - no inspect first
  - repeated same-style runs
- test whether the bounded fallback appears only when `0x26D000` is late in a longer sequence or whether it can reproduce in isolation

## Bottom Line

`0x26C800` no longer split in the repeat batch, so it has effectively fallen out of the instability zone.

`0x26D000` is now the main unresolved seam, and the surviving flip does not match a simple inspect-style or local pair-order rule.
