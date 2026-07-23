# `0x26D000` Opening/Second Delay Seeding Pass

Date: 2026-04-20

## Summary

This was the one remaining runtime pass allowed by
[26d000-investigation-scope.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/26d000-investigation-scope.md:1).

The narrow question was:

- if the bridge slot stays fixed at post-`VRT` delay `0.18`
- and the opening plus second slots are seeded together at the owned coarse values
  - `0.10`
  - `0.18`
  - `0.20`
- does any coarse scenario reproduce a bounded bridge-family strongly enough to keep the runtime ladder alive

Method:

- reused the per-slot preset:
  - `selector-26d000-bridge-slot-delay-decoupling-single-clone`
- held:
  - `bridge post-VRT initial command delay = 0.18`
- varied:
  - `opening post-VRT initial command delay`
  - `second post-VRT initial command delay`
- kept opening and second coupled at the same coarse value for each scenario
- ran `10` fresh clones per scenario
- executed the batches sequentially so each batch summary and child probe trace stayed unique
- preserved bridge-side numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`

Main result:

- `seed-0p10` -> bridge selector counts:
  - `0868` -> `8`
  - `0008` -> `1`
  - `0870` -> `1`
- `seed-0p18` -> bridge selector counts:
  - `0868` -> `9`
  - `0870` -> `1`
- `seed-0p20` -> bridge selector counts:
  - `0868` -> `9`
  - `0870` -> `1`

So the strongest closure is:

- every remaining coarse runtime scenario kept `0868` as the modal bridge family at `>= 8/10`
- no bounded selector family reached the success threshold from the scope doc
- the runtime ladder is now closed as:
  - flat-`0868` dominant with bounded outliers below reproducibility threshold

## New Owned Artifacts

- [26d000-opening-second-delay-seeding-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-opening-second-delay-seeding/26d000-opening-second-delay-seeding-results-2026-04-20.json)
- [seed-0p10.raw-results.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-opening-second-delay-seeding/seed-0p10.raw-results.json)
- [seed-0p18.raw-results.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-opening-second-delay-seeding/seed-0p18.raw-results.json)
- [seed-0p20.raw-results.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-opening-second-delay-seeding/seed-0p20.raw-results.json)
- [opening-second-delay-seeding-20260420T204156Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/opening-second-delay-seeding-20260420T204156Z)

## Harness Preset

This pass reused:

- `selector-26d000-bridge-slot-delay-decoupling-single-clone`

No tooling changes were needed inside the pass itself.

## Results

### Scenario `seed-0p10`

Parameters:

- `opening = 0.10`
- `second = 0.10`
- `bridge = 0.18`

Bridge-family counts:

- `0868` -> `8`
- `0008` -> `1`
- `0870` -> `1`

Outliers:

- `clone-03` -> `0008:00001E8E`
  - opening -> `0008:00001DE4`
  - second -> `0868:00025A7A`
- `clone-10` -> `0870:00000401`
  - opening -> `0868:00025887`
  - second -> `0868:0002B87C`

### Scenario `seed-0p18`

Parameters:

- `opening = 0.18`
- `second = 0.18`
- `bridge = 0.18`

Bridge-family counts:

- `0868` -> `9`
- `0870` -> `1`

Outlier:

- `clone-07` -> `0870:00001A19`
  - opening -> `0870:00000826`
  - second -> `0868:00025151`

### Scenario `seed-0p20`

Parameters:

- `opening = 0.20`
- `second = 0.20`
- `bridge = 0.18`

Bridge-family counts:

- `0868` -> `9`
- `0870` -> `1`

Outlier:

- `clone-10` -> `0870:0000082C`
  - opening -> `0868:00024815`
  - second -> `0868:0002B81E`

### Cross-Scenario Bridge Closure

Across all thirty fresh-clone runs:

- `0868` -> `26`
- `0870` -> `3`
- `0008` -> `1`
- `0824` -> `0`

No scenario produced:

- bounded `0008` at `>= 8/10`
- bounded `0870` at `>= 8/10`
- bounded `0824` at all

### File Diffs

All clone-tree hash diffs stayed empty across the full fixture root.

So this pass again observed no writes into the mounted clone trees.

## Findings

### 1. Every Remaining Coarse Scenario Is Flat-`0868` Dominant

This is the key scope result.

The three allowed coarse scenarios landed:

- `0.10 / 0.10 / 0.18` -> `0868` on `8/10`
- `0.18 / 0.18 / 0.18` -> `0868` on `9/10`
- `0.20 / 0.20 / 0.18` -> `0868` on `9/10`

So all remaining coarse runtime scenarios satisfy the flat-dominant side of the scope doc.

### 2. No Bounded Bridge Family Cleared The Reproducibility Threshold

The best bounded families were only sparse outliers:

- one `0008` in `seed-0p10`
- one `0870` in `seed-0p10`
- one `0870` in `seed-0p18`
- one `0870` in `seed-0p20`

That is well below the required:

- `>= 8/10`
- across `>= 3` distinct fresh clones

So rule 1 from the scope doc is not met.

### 3. The Outliers Do Not Justify Another Runtime Branch

This matters because the final pass was supposed to decide whether coupled early-slot timing was enough to reopen the bridge family meaningfully.

It was not.

The outliers stayed sparse, and they did not beat the owned determinism floor by enough margin to justify more runtime ladder work.

This is an inference from:

- the new coarse-clone seeding counts
- the earlier determinism-floor result

### 4. Bridge-Side `0824` Is Now Fully Out Of The Runtime Lane

The final allowed runtime pass preserved numeric bridge-side selector capture for `0824`.

Result:

- `0824` never appeared

So the runtime branch no longer has an owned live lead for `0824` at this seam.

## Scope Evaluation

Against [26d000-investigation-scope.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/26d000-investigation-scope.md:1):

- rule 1:
  - **not met**
- rule 2:
  - **met**
- rule 3:
  - **spent and closed**

Mechanical branch label:

- **flat-`0868` dominant with bounded outliers below reproducibility threshold**

## Practical Interpretation

The runtime ladder did what it could do.

Coupling the opening and second slots to the owned coarse values while holding the bridge at `0.18` did not rescue a reproducible bounded bridge family. Instead it strengthened the opposite conclusion:

- all three remaining coarse scenarios are mostly flat `0868`
- bounded `0008` and `0870` are still only outliers

So the right next move is no longer another runtime timing pass.

## Recommended Next Move

Per the scope doc, runtime ladder work on `0x26D000` should now stop.

The next strongest move is:

- **static Ghidra analysis of the caller / bridge-family seam around the `0x26D000` lane**

Priority questions for that static handoff:

- which caller-side state decides whether the bridge later lands in flat `0868` object `3` versus rare bounded object `1`
- whether the bounded outliers share a hidden precondition not visible in the opening and second live stops
- where the late `0x26D000` lane diverges away from the earlier object-`1` families before the bridge slot becomes visible

## Bottom Line

The one remaining authorized runtime pass is complete, and it closes the branch cleanly. With bridge delay fixed at `0.18`, all three allowed coupled opening/second scenarios stayed flat-`0868` dominant across ten fresh clones each: `8/10`, `9/10`, and `9/10`. No bounded bridge family met the reproducibility threshold, all clone trees stayed hash-identical, and the `0x26D000` runtime ladder should now hand off to static Ghidra analysis.
