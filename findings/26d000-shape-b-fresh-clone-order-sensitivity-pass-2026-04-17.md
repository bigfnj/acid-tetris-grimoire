# `0x26D000` Shape-B Fresh-Clone Order-Sensitivity Pass

Date: 2026-04-17

## Summary

This pass followed the fresh-clone reset result:

- fresh clones could reopen the early Shape B families
- but not deterministically

The narrow question here was:

- do those restored bridge families follow clone position in the batch

To answer that, I ran:

- one fresh three-clone batch in order `1 -> 2 -> 3`
- one fresh three-clone batch in order `3 -> 1 -> 2`
- one isolated single-clone control batch

Main result:

- every bridge probe across all three runs stayed flat `0868`
- only the earliest three-clone batch still leaked object-`1` families one slot earlier at the second `no-inspect` step

So the strongest new closure is:

- simple batch slot position is not enough to explain the earlier fresh-clone bridge reopenings
- the remaining factor is more likely tied to broader DOSBox or process-environment freshness outside the game tree

## New Owned Artifacts

- [26d000-shape-b-fresh-clone-order-sensitivity-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-shape-b-fresh-clone-order-sensitivity/26d000-shape-b-fresh-clone-order-sensitivity-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T225259Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T225821Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T230342Z/batch-summary.json)

Fresh-clone fixture root:

- [shape-b-order-sensitivity-20260417T225112Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-order-sensitivity-20260417T225112Z)

## Harness Presets

This pass added and exercised:

- `selector-26d000-shape-b-order-a-123`
- `selector-26d000-shape-b-order-b-312`
- `selector-26d000-shape-b-order-single-control`

## Results

### Order A: `1 -> 2 -> 3`

#### Slot 1

- `no-inspect` -> `0868:00025154`
- `no-inspect-again` -> `0870:000003D2`
- `bridge` -> `0868:00025417`

#### Slot 2

- `no-inspect` -> `0868:0002514F`
- `no-inspect-again` -> `0008:000014A6`
- `bridge` -> `0868:00023A1A`

#### Slot 3

- `no-inspect` -> `0868:0002B855`
- `no-inspect-again` -> `0008:00001491`
- `bridge` -> `0868:00015081`

### Order B: `3 -> 1 -> 2`

#### First Clone In Batch

- `no-inspect` -> `0868:0002B828`
- `no-inspect-again` -> `0868:0002538E`
- `bridge` -> `0868:00023A1D`

#### Second Clone In Batch

- `no-inspect` -> `0868:0002B863`
- `no-inspect-again` -> `0868:0002B54D`
- `bridge` -> `0868:0002B81E`

#### Third Clone In Batch

- `no-inspect` -> `0868:00025385`
- `no-inspect-again` -> `0868:00023A2D`
- `bridge` -> `0868:00024815`

### Single-Clone Control

- `no-inspect` -> `0868:00025419`
- `no-inspect-again` -> `0868:00025A6C`
- `bridge` -> `0868:0002B883`

### Hash Diffs

All seven clone hash-diff files remained empty:

- batch-a-slot-1 -> `0`
- batch-a-slot-2 -> `0`
- batch-a-slot-3 -> `0`
- batch-b-slot-1 -> `0`
- batch-b-slot-2 -> `0`
- batch-b-slot-3 -> `0`
- control-slot-1 -> `0`

## Findings

### 1. Batch Position Did Not Reopen The Bridge

This is the main closure.

Across:

- order A
- order B
- and the isolated control

every bridge stayed flat `0868`.

So the earlier fresh-clone bridge reopenings do not replay under a simple:

- first clone
- second clone
- third clone

position model.

### 2. The Earliest Batch Still Leaked Object-`1` Families One Slot Earlier

Order A still showed live object-`1` behavior at the second `no-inspect` step:

- `0870` once
- `0008` twice

But those families did not carry forward to the bridge.

That means the signal did not disappear entirely; it retreated one slot earlier.

### 3. Later Batches Fully Collapsed Even At The Second `no-inspect` Step

This is just as important.

In:

- order B
- and the isolated control

the second `no-inspect` step stayed flat `0868` every time.

So even the pre-bridge leak seen in order A is not stable across fresh clone sets run later in the session.

### 4. Clone Identity Is Not The Best Explanation

Since:

- all clone trees were created from the same source
- all hash diffs stayed empty
- and both order permutations collapsed at the bridge

there is no good evidence that a particular clone identity owns a specific bridge family.

The difference is more consistent with:

- broader environment freshness
- process-level order across batches
- or another external factor outside the cloned tree itself

This is an inference from the evidence, not a direct proof.

### 5. The Frontier Has Shifted Outside The Game Directory And Outside Simple Slot Order

At this point we have ruled out:

- in-tree file mutation
- simple Shape-A interruption reset
- simple clone slot order

That makes the next strongest axis:

- DOSBox-side environment freshness

Examples to isolate next:

- temporary config directories
- home/config/cache paths
- per-run debugger-side state outside the mounted clone

## Practical Interpretation

This pass was a negative result, but a strong one.

We now know:

- fresh clones matter sometimes
- but simple slot order does not explain the bridge reopenings
- and the effect still is not being written back into the game tree

So the next pass should move outward from the clone tree and start controlling the DOSBox environment itself.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` DOSBox-Environment Freshness Pass**

Focus:

- keep the short Shape B lane fixed
- run it against a fresh clone while varying only external environment state such as:
  - `HOME`
  - `XDG_CONFIG_HOME`
  - disposable DOSBox config/cache locations
- compare whether bridge outcomes return to:
  - `0008`
  - `0870`
  - or remain flat `0868`

## Bottom Line

The order-sensitivity pass did not reopen the Shape B bridge.

All bridge probes stayed flat `0868`, while only the earliest three-clone batch still leaked `0870` and `0008` one slot earlier. The strongest remaining explanation is now DOSBox or process-environment freshness outside the game directory.
