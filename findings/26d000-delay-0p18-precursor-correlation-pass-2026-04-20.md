# `0x26D000` Delay-`0.18` Precursor Correlation Pass

Date: 2026-04-20

## Summary

This pass followed the exact-delay `0.18` stabilization result:

- the bridge-side `0870` family was real
- but only `1` of `5` exact `0.18` repeats had reproduced it
- fine delay bracketing around:
  - `0.175`
  - `0.185`
  did not widen the bridge-side object-`1` window

The narrow question here was:

- across a larger exact-delay `0.18` cluster, do opening-slot or second-slot selector families predict the rare bridge-side object-`1` families
- or do the useful bridge-side families emerge without a stable visible precursor

The broader three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I added eight fresh exact-delay `0.18` repeats and merged them with the five exact-delay `0.18` runs already owned by the stabilization pass, producing a thirteen-run exact-delay cluster.

Main result:

- no visible opening-slot or second-slot selector family is necessary or sufficient for bridge-side object `1`
- the merged exact-delay cluster produced:
  - `8` bridge-side `0868`
  - `4` bridge-side `0870`
  - `1` bridge-side `0008`

So the best new closure is:

- precursor-family correlation is mostly closed
- the useful control is likely bridge-local or late-session-local
- the next strongest move is to decouple bridge-slot timing from the earlier two slots

## New Owned Artifacts

- [26d000-delay-0p18-precursor-correlation-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-delay-0p18-precursor-correlation/26d000-delay-0p18-precursor-correlation-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T170232Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T170419Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T170605Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T170752Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T170938Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T171125Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T171311Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T171458Z/batch-summary.json)

Input artifacts reused by this pass:

- [26d000-delay-0p18-bridge-family-stabilization-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-delay-0p18-bridge-family-stabilization/26d000-delay-0p18-bridge-family-stabilization-results-2026-04-20.json)

Fixture root:

- [delay-0p18-precursor-correlation-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/delay-0p18-precursor-correlation-20260420T000000Z)

## Harness Preset

This pass reused:

- `selector-26d000-post-vrt-delay-refinement-single-clone`

No tooling changes were needed inside the pass itself.

## Results

### New Exact `0.18` Repeats

#### Repeat 6

- `opening` -> `0868:00023979`
- `second` -> `0868:00023979`
- `bridge` -> `0870:000003E9`

#### Repeat 7

- `opening` -> `0870:000003DF`
- `second` -> `0868:0002B821`
- `bridge` -> `0868:00023979`

#### Repeat 8

- `opening` -> `0870:00000425`
- `second` -> `0868:00023A61`
- `bridge` -> `0868:0002B82A`

#### Repeat 9

- `opening` -> `0868:00023A3B`
- `second` -> `0008:00000C81`
- `bridge` -> `0870:000003DF`

#### Repeat 10

- `opening` -> `0870:0000082C`
- `second` -> `0868:0002B826`
- `bridge` -> `0868:0002B87F`

#### Repeat 11

- `opening` -> `0868:0002481E`
- `second` -> `0868:00023923`
- `bridge` -> `0868:00025419`

#### Repeat 12

- `opening` -> `0868:0002B828`
- `second` -> `0868:00023923`
- `bridge` -> `0008:000014A6`

#### Repeat 13

- `opening` -> `0870:00000425`
- `second` -> `0868:0002B882`
- `bridge` -> `0870:000003EB`

### Merged Exact `0.18` Cluster Counts

Across all thirteen exact `0.18` runs:

- bridge selectors:
  - `0868` -> `8`
  - `0870` -> `4`
  - `0008` -> `1`
- bridge object-`1` runs -> `5`
- opening object-`1` runs -> `6`
- second object-`1` runs -> `1`

Merged selector flow counts:

- `opening -> bridge`
  - `0868 -> 0868` -> `3`
  - `0868 -> 0870` -> `3`
  - `0868 -> 0008` -> `1`
  - `0008 -> 0868` -> `1`
  - `0870 -> 0868` -> `4`
  - `0870 -> 0870` -> `1`
- `second -> bridge`
  - `0868 -> 0868` -> `8`
  - `0868 -> 0870` -> `3`
  - `0868 -> 0008` -> `1`
  - `0008 -> 0870` -> `1`

### Bridge Descriptor Closure

The bridge-side object-`1` families in the merged exact `0.18` cluster were:

- selector `0870`
  - base `0x0000A3C0`
  - limit `0x0000FFFF`
- selector `0008`
  - base `0x00008240`
  - limit `0x0000FFFF`

The dominant collapsed bridge family remained:

- selector `0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`

As in the earlier passes:

- numeric `0824` stayed absent as a selected live bridge family

### File Diffs

New clone tree hash diffs:

- `r1-clone` -> `0`
- `r2-clone` -> `0`
- `r3-clone` -> `0`
- `r4-clone` -> `0`
- `r5-clone` -> `0`
- `r6-clone` -> `0`
- `r7-clone` -> `0`
- `r8-clone` -> `0`

So this pass again observed no writes into the mounted clone trees.

## Findings

### 1. No Visible Precursor Selector Family Is Necessary

This is the main closure.

Bridge-side object `1` appeared five times in the merged exact-delay cluster:

- four times on `0870`
- once on `0008`

But those bridge-side object-`1` results emerged under different precursor shapes:

- `0868, 0868 -> 0870`
- `0868, 0008 -> 0870`
- `0870, 0868 -> 0870`
- `0868, 0868 -> 0008`

So no single visible opening-slot or second-slot selector family is required before the bridge can land on the object-`1` side.

### 2. Opening-Slot Object-`1` Is Not Sufficient

This is the strongest negative correlation result.

Across the thirteen exact `0.18` runs:

- six runs reached object `1` at the opening slot
- only one of those six stayed on the object-`1` side at the bridge

The other five opening-slot object-`1` runs still collapsed back to flat `0868` at the bridge.

So “opening slot reached object `1`” is not a usable predictor.

### 3. Second-Slot Object-`1` Is Not Necessary

Only one exact `0.18` run reached object `1` at the second slot:

- `0.18-r9` -> `0008:00000C81`

That run did advance to bridge-side `0870`, but four other bridge-side object-`1` runs still emerged while the second slot stayed on flat `0868`.

So the second slot can participate, but it is not required.

### 4. Bridge-Side Object-`1` Can Emerge With No Visible Earlier Object-`1`

This is the most useful positive result.

Two bridge-side `0870` hits and the one bridge-side `0008` hit came from:

- `opening` on `0868`
- `second` on `0868`

This is an inference from the selected live stops in the merged cluster.

That means the useful bridge families can emerge even when both earlier visible slots stay on the flat side.

### 5. The Bridge Family Is Mixed, Not Singular

The exact-delay `0.18` cluster did not collapse to one useful bridge family.

Instead it produced:

- `4` bridge-side `0870`
- `1` bridge-side `0008`

So the current live bridge window is still bifurcated across at least two bounded selector families.

### 6. `0824` Still Did Not Reappear

As in the previous passes:

- no merged exact-delay `0.18` run reached bridge-side `0824`

So the practical frontier is still below the rare `0824` anomaly family.

## Practical Interpretation

This pass closes the main precursor-correlation question.

We now know:

- opening-slot selector family is not a usable gate
- second-slot selector family is not a usable gate
- bridge-side object `1` can emerge without any visible earlier object-`1` in the same three-slot shape

So the strongest remaining explanation is that the useful control is bridge-local or late-session-local rather than a simple visible precursor family in the first two slots.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Bridge-Slot Delay Decoupling Pass**

Focus:

- add a selector-aware batch preset that lets:
  - the opening slot
  - the second slot
  - the bridge slot
  use different post-`VRT` initial command delays
- hold the opening and second slots at the working `0.18` lane
- vary only the bridge slot across a small ladder such as:
  - `0.10`
  - `0.18`
  - `0.20`
- preserve numeric selector capture for:
  - `0824`
  - `0008`
  - `0870`
  - `0868`

The success condition is not just another sparse bridge-side object-`1` hit, but evidence that the final inspect-slot timing itself can widen or shift the bridge family.

## Bottom Line

The larger exact-delay `0.18` cluster closed the precursor question more than it strengthened it.

Across thirteen exact `0.18` runs, no visible opening-slot or second-slot selector family was necessary or sufficient for bridge-side object `1`. The next strongest move is to decouple bridge-slot timing from the first two slots and test whether the final inspect slot itself is the true lever.
