# Upper-Edge Crossover Stability Pass

Date: 2026-04-17

## Summary

This pass tested whether the top of the owned cold-lane crossover window was stable or bifurcating.

Main result:

- the upper edge produced a stable flat `0868` cluster
- but that result directly conflicts with the previous fine-grained pass
- so the cold-lane seam is not behaving like one deterministic threshold

## New Owned Artifacts

- [upper-edge-crossover-stability-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/upper-edge-crossover-stability/upper-edge-crossover-stability-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T172935Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-emergence-upper-edge`

It repeats nearby cold no-autoexec budgets:

- `0x26C000`
- `0x26E000`
- `0x26F000` twice
- `0x270000` twice

## Results

All six runs in the batch landed on:

- selector `0868`
- object `3`

Detailed stops:

- `0x26C000`
  - `0868:000238E3`
  - object `3`
- `0x26E000`
  - `0868:0002B883`
  - object `3`
- `0x26F000` repeat A
  - `0868:00024815`
  - object `3`
- `0x26F000` repeat B
  - `0868:0002482E`
  - object `3`
- `0x270000` repeat A
  - `0868:00024816`
  - object `3`
- `0x270000` repeat B
  - `0868:00024810`
  - object `3`

Across all six runs:

- the first live VRT re-entry point remained `0824:0000006A`
- selector metadata stayed flat:
  - base `0x00000000`
  - limit `0xFFFFFFFF`
  - flags `11011`

## Findings

### 1. The Upper Edge Contains A Stable Flat-Side Cluster

Within this batch, the flat-side result was not random.

The repeated `0x26F000` and `0x270000` runs all landed in the same narrow neighborhood:

- roughly `0868:00024810 -> 0868:0002482E`

So there is a real repeatable flat-side cluster at the upper edge.

### 2. The Cold-Lane Seam Is Better Described As Bifurcating Than Deterministic

This batch conflicts directly with the previous fine-grained pass, which kept:

- `0x262000`
- `0x264000`
- `0x266000`
- `0x268000`
- `0x26A000`

on bounded `0870`.

Now, the upper-edge batch shows:

- `0x26C000`
- `0x26E000`
- `0x26F000`
- `0x270000`

all on flat `0868`.

Taken together, the simplest interpretation is:

- the cold no-autoexec lane is not crossing a single calm threshold
- it is bifurcating into different run families near the top of the window

### 3. The Stable Flat-Side Cluster Still Falls Straight Into Object 3

Even though the upper-edge flat cluster is now owned and repeatable, it is still not the kind of landing we want most.

It repeatedly lands in:

- object `3`

not in an informative early object-2 seam.

So this pass improves determinism on the flat side, but it does not yet improve usefulness.

### 4. The Right Next Question Is What Separates The Run Families

The key unknown is no longer:

- “does flat `0868` exist near the upper edge?”

That answer is yes.

The key unknown is now:

- what hidden condition makes the cold lane stay bounded in one pass and jump flat in another

## Practical Interpretation

This pass is important because it changes the kind of runtime problem we are solving.

We are no longer just mapping a crossover threshold.
We are now trying to isolate what causes the cold lane to choose one family or the other near the upper edge.

## Recommended Next Move

The strongest next move is now:

- **Cold-Lane Bifurcation Isolation Pass**

Focus:

- keep the same no-autoexec control
- repeat one or two upper-edge budgets while varying only one possible hidden influence at a time
- especially run-order effects, startup timing drift, or prompt-side inspection order

## Bottom Line

The upper-edge crossover is not just jittery.

It now looks bifurcated:

- one family stays bounded `0870`
- another family lands repeatably in flat `0868` object `3`

That makes isolation of the hidden branch condition the next strongest move.
