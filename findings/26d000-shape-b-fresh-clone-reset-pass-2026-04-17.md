# `0x26D000` Shape-B Fresh-Clone Reset Pass

Date: 2026-04-17

## Summary

This pass followed the carry-decay isolation result:

- Shape B could stay fully collapsed even while Shape A remained live
- that pushed the frontier toward game-directory freshness

The narrow question here was:

- does a truly fresh disposable clone of `Original.Game` restore the early Shape B bridge families

To answer that, the short Shape B lane:

- `no-inspect -> no-inspect -> inspect`

was run once against three fresh clones created from `Original.Game`.

Main result:

- clone `1` bridge -> `0008`
- clone `2` bridge -> flat `0868`
- clone `3` bridge -> `0870`

And all three clone trees stayed hash-identical before and after the runs.

So the strongest new closure is:

- freshness can reopen the early Shape B bridge families
- but the deciding factor is not a file mutation inside the game directory

## New Owned Artifacts

- [26d000-shape-b-fresh-clone-reset-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-shape-b-fresh-clone-reset/26d000-shape-b-fresh-clone-reset-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T224031Z/batch-summary.json)

Fresh-clone fixture root:

- [shape-b-fresh-clone-reset-20260417T223931Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z)

Hash manifests:

- [clone-1.before.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-1.before.sha256)
- [clone-1.after.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-1.after.sha256)
- [clone-1.hash-diff.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-1.hash-diff.txt)
- [clone-2.before.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-2.before.sha256)
- [clone-2.after.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-2.after.sha256)
- [clone-2.hash-diff.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-2.hash-diff.txt)
- [clone-3.before.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-3.before.sha256)
- [clone-3.after.sha256](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-3.after.sha256)
- [clone-3.hash-diff.txt](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-fresh-clone-reset-20260417T223931Z/clone-3.hash-diff.txt)

## Harness Preset

This pass added and exercised:

- `selector-26d000-shape-b-fresh-clone-reset`

## Results

### Clone 1

- `26D000_clone1_no_inspect`
  - `0868:0002B824`
  - object `3`
- `26D000_clone1_no_inspect_again`
  - `0868:00025419`
  - object `3`
- `26D000_clone1_bridge`
  - `0008:000014A6`
  - object `1`

### Clone 2

- `26D000_clone2_no_inspect`
  - `0868:0002481C`
  - object `3`
- `26D000_clone2_no_inspect_again`
  - `0868:00024827`
  - object `3`
- `26D000_clone2_bridge`
  - `0868:0002B87F`
  - object `3`

### Clone 3

- `26D000_clone3_no_inspect`
  - `0868:0002B883`
  - object `3`
- `26D000_clone3_no_inspect_again`
  - `0868:000258E0`
  - object `3`
- `26D000_clone3_bridge`
  - `0870:000003F7`
  - object `1`

### Clone File Hashes

All three clone hash-diff files are empty:

- clone `1` diff bytes -> `0`
- clone `2` diff bytes -> `0`
- clone `3` diff bytes -> `0`

So this pass observed no on-disk file changes inside the mounted clone trees.

## Findings

### 1. Fresh Clones Can Reopen The Early Shape B Bridge Families

This is the main positive result.

Two of the three fresh clones reached object `1` at the bridge:

- clone `1` -> `0008`
- clone `3` -> `0870`

That means a fresh clone really can reopen the early bridge-side families that had been absent in the collapsed Shape B runs.

### 2. Freshness Alone Is Not Deterministic

Clone `2` still collapsed:

- `0868:0002B87F`

So “fresh clone” is not sufficient by itself to force a particular family.

The reset effect is real, but it is not deterministic from game-tree contents alone.

### 3. The Differentiating State Is Not Being Written Back Into The Game Tree

This is the strongest closure from the file-hash side.

All three clone trees were hash-identical before and after their runs.

So the observed difference between:

- clone `1` -> `0008`
- clone `2` -> `0868`
- clone `3` -> `0870`

is not explained by:

- writes to `SETUP.DAT`
- score-table updates
- or other visible file changes inside the mounted game directory

### 4. The Current Model Has Shifted Away From In-Tree Persistence

The strongest current explanation is now:

- an external or order-sensitive factor outside the mounted game tree

That could mean:

- run ordering
- timing-sensitive startup behavior
- DOSBox-side state outside the cloned directory
- or another source of nondeterminism not captured by the file hashes

This is an inference from the evidence, not a direct proof.

### 5. Clone Order Now Looks Like The Highest-Signal Axis

The three bridge outcomes arrived in order:

- clone `1` -> `0008`
- clone `2` -> `0868`
- clone `3` -> `0870`

Since the clone trees were identical and unchanged, the strongest next question is whether:

- batch slot order itself is part of the outcome

That suggests the next pass should explicitly permute clone order or isolate one clone per batch.

## Practical Interpretation

This pass ruled out one big candidate.

We now know:

- fresh clones do matter
- but not because the game tree is mutating during the run
- and not in a deterministic per-clone way

So the next pass should stop focusing on file persistence and move toward order sensitivity and external-state controls.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Shape-B Fresh-Clone Order-Sensitivity Pass**

Focus:

- reuse fresh disposable clones
- permute which clone is run first, second, and third
- compare whether the bridge families follow:
  - clone identity
  - or batch position
- optionally compare one-clone-per-batch control runs against the multi-clone batch order

## Bottom Line

Fresh clones can restore the early Shape B bridge families, but not deterministically.

One clone produced `0008`, one produced `0870`, and one collapsed to `0868`, while all three clone trees remained byte-identical before and after the runs. The strongest remaining explanation is now order sensitivity or another external factor outside the game directory.
