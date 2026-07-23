# Lower-Upper-Edge Boundary Isolation Pass

Date: 2026-04-17

## Summary

This pass narrowed the probe-sensitive cold-lane boundary to the smaller interval suggested by the previous bifurcation pass:

- `0x26C800`
- `0x26D000`
- `0x26D800`

Main result:

- the inspected-vs-uninspected family split still exists at `0x26C800` and `0x26D000`
- by `0x26D800`, both sides converge on the same flat `0868` object-`3` family
- the split is not controlled by prompt-side inspection alone, because the `0x26C800` pair inverted the earlier `0x26C000` orientation

## New Owned Artifacts

- [lower-upper-edge-boundary-isolation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/lower-upper-edge-boundary-isolation/lower-upper-edge-boundary-isolation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T190300Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-lower-upper-edge-boundary`

It compares three nearby budgets twice:

- once with `selinfo cs` + `ldt`
- once with no pre-arm inspection

## Results

### `0x26C800`

- inspected
  - `0870:00000818`
  - object `1`
  - bounded selector `0870`
- no inspect
  - `0868:0002B82A`
  - object `3`
  - flat selector `0868`

### `0x26D000`

- inspected
  - `0868:00025418`
  - object `3`
  - flat selector `0868`
- no inspect
  - `0870:000003FC`
  - object `1`
  - bounded selector `0870`

### `0x26D800`

- inspected
  - `0868:00024828`
  - object `3`
- no inspect
  - `0868:00024810`
  - object `3`

## Findings

### 1. The Probe-Sensitive Split Still Exists At `0x26C800` And `0x26D000`

This confirms that the instability zone did not end at `0x26C000`.

Owned results:

- `0x26C800`
  - inspected -> bounded `0870` object `1`
  - no inspect -> flat `0868` object `3`
- `0x26D000`
  - inspected -> flat `0868` object `3`
  - no inspect -> bounded `0870` object `1`

So both points still sit inside the family-choice seam.

### 2. The Split Collapses By `0x26D800`

At `0x26D800`, both paths converged:

- inspected -> `0868:00024828`
- no inspect -> `0868:00024810`

Both runs landed in:

- flat selector `0868`
- object `3`
- the same established `0x2481x -> 0x2482x` neighborhood

That makes `0x26D800` the first owned point in this narrowed interval where the family choice no longer appears probe-sensitive.

### 3. Prompt-Side Inspection Is Not The Sole Hidden Lever

This is the most important interpretation update from the pass.

The earlier isolation result at `0x26C000` was:

- inspected -> flat `0868`
- no inspect -> bounded `0870`

But at `0x26C800`, the orientation flipped:

- inspected -> bounded `0870`
- no inspect -> flat `0868`

If prompt-side inspection were the only causal control, we would expect the orientation to stay consistent.

Instead, the owned evidence now supports a stronger conclusion:

- inspection can bias the seam
- but a second hidden condition is also participating in family choice inside the `0x26C800 -> 0x26D000` zone

### 4. The Useful Boundary Is Now Much Tighter

After this pass, the working boundary is:

- still split-sensitive at `0x26C800`
- still split-sensitive at `0x26D000`
- converged to flat `0868` object `3` by `0x26D800`

So the next best isolation target is no longer the whole `0x26C000 -> 0x26E000` range.

It is now the smaller band:

- `0x26C800 -> 0x26D800`

with emphasis on repeated paired runs that can separate:

- genuine stochastic drift
- run-order effects
- remaining probe-sensitive bias

## Practical Interpretation

This pass gives us a more actionable shape than the previous broad bifurcation result.

We now know:

- the family split survives into the middle of the lower upper-edge window
- the split collapses by `0x26D800`
- inspection is part of the story, but not a complete explanation

That means the next pass should stop asking where the seam roughly is and start asking what stabilizes or flips it inside the last unresolved band.

## Recommended Next Move

The strongest next move is now:

- **Boundary Orientation Repeat Pass**

Focus:

- repeat paired inspected/uninspected runs at `0x26C800` and `0x26D000`
- reverse run ordering to test whether the family orientation follows execution order rather than inspection alone
- confirm whether the split is stochastic, order-sensitive, or governed by another hidden state carry

## Bottom Line

The cold-lane family split remains active at `0x26C800` and `0x26D000`, but it collapses by `0x26D800`.

That tightens the remaining uncertainty to a much smaller band and also shows that prompt-side inspection is not the only hidden control, because the split orientation can invert between nearby budgets.
