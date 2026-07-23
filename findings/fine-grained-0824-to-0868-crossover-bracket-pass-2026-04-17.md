# Fine-Grained 0824-to-0868 Crossover Bracket Pass

Date: 2026-04-17

## Summary

This pass subdivided the newly owned `0x260000 -> 0x270000` crossover window on the cold no-autoexec lane.

Main result:

- the selector crossover is now tighter than before
- but it is not behaving like a smooth monotonic threshold
- it looks more like a jittery upper-edge seam

## New Owned Artifacts

- [0824-to-0868-emergence-fine-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/0824-to-0868-emergence-fine/0824-to-0868-emergence-fine-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T163844Z/batch-summary.json)

## Fine Bracket

The new harness preset:

- `selector-emergence-fine`

tested these cold no-autoexec budgets with constant target and inspection:

- `0x262000`
- `0x264000`
- `0x266000`
- `0x268000`
- `0x26A000`

All used:

- target `0x17719`
- `selinfo cs`
- `ldt`
- no synthetic input

## Results

### Fine Batch

- `0x262000`
  - `0870:000003F7`
  - object `1`
- `0x264000`
  - `0870:000003B4`
  - object `1`
- `0x266000`
  - `0870:00001563`
  - object `1`
- `0x268000`
  - `0870:00000C48`
  - object `1`
- `0x26A000`
  - `0870:000003EB`
  - object `1`

So the fine batch itself stayed entirely on bounded selector `0870`.

### Upper-Edge Stability Reruns

Because that result differed from the earlier coarse emergence pass, this pass also ran direct stability checks at the upper edge:

- `0x26A000` repeat
  - child run `20260417T164218Z-01`
  - `0870:0000080E`
  - object `1`
  - bounded selector `0870`
- `0x270000` repeat
  - child run `20260417T164218Z`
  - `0868:00024828`
  - object `3`
  - flat selector `0868`

## Findings

### 1. The Owned Selector Crossover Is Now Tighter

After the fine batch plus the upper-edge reruns, the project now owns:

- bounded `0870` through `LOGC 0x26A000`
- flat `0868` at `LOGC 0x270000`

That is a tighter selector crossover bound than the previous pass.

### 2. The Seam Looks Jittery, Not Smooth

If the crossover behaved like a simple smooth threshold, we would expect nearby fine-budget samples to drift steadily toward the flat side.

Instead, the fine batch produced several very different bounded `0870` offsets:

- `0x3F7`
- `0x3B4`
- `0x1563`
- `0xC48`
- `0x3EB`

Then the very next repeated upper-edge sample at `0x270000` flipped cleanly to flat `0868`.

That pattern looks less like a calm monotonic transition and more like a jittery seam with path sensitivity near the upper edge.

### 3. The First Repeated Flat Entry Still Drops Into Object 3

The repeated flat-side confirmation at `0x270000` landed at:

- `0868:00024828`
- object `3`

So even after tightening the selector crossover itself, the current cold lane still does not hand us an informative early object-2 frontier.

### 4. The Next Best Move Is Stability-Focused, Not Wider Bracketing

We no longer need a wider coarse bracket.
What we need now is better characterization of the upper-edge seam itself:

- how often does `0x270000` stay flat
- whether `0x26C000`, `0x26E000`, or repeated `0x26F000` / `0x270000` runs bifurcate
- and whether any minimal steering can keep the crossover flat without falling straight into object `3`

## Practical Interpretation

This pass improved the crossover map, but it also changed its character.

The selector seam is now tighter, but less orderly than a single deterministic threshold would suggest.
That is useful because it tells us what kind of runtime problem we are actually dealing with.

## Recommended Next Move

The strongest next move is now:

- **Upper-Edge Crossover Stability Pass**

Focus:

- repeat and cluster runs around `0x26C000 -> 0x270000`
- measure whether the flat-side crossover is stable or bifurcating
- look for any repeatable flat `0868` entry that does not immediately collapse into the same object-3 region

## Bottom Line

The selector crossover is now tighter:

- bounded `0870` through `0x26A000`
- flat `0868` at `0x270000`

But the fine bracket also showed that the seam is jittery near the upper edge, and the first repeated flat entry still lands in object `3`.
