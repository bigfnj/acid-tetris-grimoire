# Cold-Lane Bifurcation Isolation Pass

Date: 2026-04-17

## Summary

This pass tested a specific possible hidden influence:

- whether our own prompt-side debugger inspection is biasing the cold-lane family choice near the upper edge

Main result:

- yes, inspection matters at `0x26C000`
- but it does not explain the whole flat-side cluster once we reach `0x26F000` and above

## New Owned Artifacts

- [cold-lane-bifurcation-isolation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/cold-lane-bifurcation-isolation/cold-lane-bifurcation-isolation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T174535Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-bifurcation-isolation`

It compares the same upper-edge budgets twice:

- once with `selinfo cs` + `ldt`
- once with no pre-arm inspection at all

## Results

### `0x26C000`

- inspected
  - `0868:0002B882`
  - object `3`
  - flat selector `0868`
- no inspect
  - `0870:00000425`
  - object `1`
  - bounded selector `0870`

### `0x26F000`

- inspected
  - `0868:00024816`
  - object `3`
- no inspect
  - `0868:0002482E`
  - object `3`

### `0x270000`

- inspected
  - `0868:00024831`
  - object `3`
- no inspect
  - `0868:0002481E`
  - object `3`

## Findings

### 1. Prompt-Side Inspection Can Change The Cold-Lane Family At `0x26C000`

This is the strongest direct isolation result so far.

At the same budget:

- inspected run -> flat `0868` object `3`
- uninspected run -> bounded `0870` object `1`

So the probe itself is not perfectly neutral at the lower edge of the upper cluster.

### 2. Prompt-Side Inspection Does Not Explain The Whole Upper-Edge Flat Cluster

At:

- `0x26F000`
- `0x270000`

both inspected and uninspected runs still landed in the same flat `0868` object-3 family.

So the stable flat-side cluster above that point appears robust even without debugger-side inspection.

### 3. The Real Instability Zone Has Shifted Lower

After this pass, the best interpretation is:

- there is a probe-sensitive boundary near `0x26C000`
- but by `0x26F000` the cold lane has largely committed to the flat-side family regardless of inspection

That means the most important unresolved zone is now:

- roughly `0x26C000 -> 0x26E000`

### 4. The Probe Is Part Of The Story, But Not The Whole Story

This is a healthy result because it avoids two bad conclusions:

- “the debugger never affects anything”
- “the whole flat cluster is just a probe artifact”

Neither is true.

The owned evidence now supports a middle position:

- prompt-side inspection can bias the lower boundary
- but the higher flat cluster is still real without it

## Practical Interpretation

This pass turns the bifurcation story from vague instability into a more actionable shape.

We now know that:

- some of the lower-edge family choice is probe-sensitive
- the higher flat-side region is more robust

So the next best pass should stop looking at the whole upper edge and instead isolate the narrower lower boundary where the family still flips.

## Recommended Next Move

The strongest next move is now:

- **Lower-Upper-Edge Boundary Isolation Pass**

Focus:

- compare inspected vs uninspected runs in the narrower `0x26C000 -> 0x26E000` interval
- add repeats at one or two intermediate points
- see whether the family choice converges toward bounded `0870` or flat `0868` as the budget rises

## Bottom Line

Prompt-side inspection is a real bias at `0x26C000`.

But by `0x26F000` and `0x270000`, the flat `0868` object-3 cluster remains even without inspection.

So the bifurcation is real, and the next useful target is the narrower lower boundary where probe sensitivity still matters.
