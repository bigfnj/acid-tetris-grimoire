# `0x26D000` `0008` Family Confirmation Pass

Date: 2026-04-17

## Summary

This pass directly tested the strongest open interpretation from the previous step:

- whether selector `0008` is primarily tied to inspect slots
- or to opening slots more generally

It used two short inspect-focused batch families:

- an inspect-opening ladder
- a mixed-opening compare

Main result:

- selector `0008` did **not** reproduce in either batch
- inspect-led openings stayed flat `0868`
- inspect in slot `2` after a no-inspect opening fell to bounded `0870`, not `0008`

So the owned `0008` branch is not explained by a simple inspect-slot rule or a simple opening-slot rule.

## New Owned Artifacts

- [26d000-0008-family-confirmation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-0008-family-confirmation/26d000-0008-family-confirmation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T202316Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T202547Z/batch-summary.json)

## Harness Presets

This pass added and exercised:

- `selector-26d000-inspect-opening-ladder`
- `selector-26d000-mixed-opening-compare`

## Results

### Inspect-Opening Ladder

- `26D000_inspect_open_1`
  - `0868:0002B880`
  - object `3`
  - flat `0868`
- `26D000_inspect_open_2`
  - `0868:00023923`
  - object `3`
  - flat `0868`
- `26D000_inspect_open_3`
  - `0868:00024816`
  - object `3`
  - flat `0868`
- `26D000_no_inspect_after_inspect_4`
  - `0870:0000040C`
  - object `1`
  - bounded `0870`

### Mixed-Opening Compare

- `26D000_no_inspect_open_m1`
  - `0868:00025159`
  - object `3`
  - flat `0868`
- `26D000_inspect_after_no_inspect_m2`
  - `0870:000003E9`
  - object `1`
  - bounded `0870`
- `26D000_inspect_after_inspect_m3`
  - `0868:0002B824`
  - object `3`
  - flat `0868`
- `26D000_no_inspect_after_inspect_m4`
  - `0870:000003EB`
  - object `1`
  - bounded `0870`

## Findings

### 1. `0008` Did Not Reproduce Under Deliberate Inspect-Led Openings

This is the central closure from the pass.

The previous pass had left open the idea that `0008` might be:

- an inspect-slot family
- or an opening-slot family

But in the inspect-opening ladder:

- opening inspect -> flat `0868`
- second inspect -> flat `0868`
- third inspect -> flat `0868`

So repeated inspect-led openings did not bring `0008` back.

### 2. Inspect In Slot `2` After No-Inspect Maps To `0870`, Not `0008`

The mixed-opening compare is the strongest disambiguation run in the pass.

Observed:

- slot `1` no-inspect -> flat `0868`
- slot `2` inspect-after-no-inspect -> `0870:000003E9`

That matters because a simpler model could have been:

- “put inspect into slot `2` and `0008` appears”

That model is now closed.

Under this controlled comparison, slot-`2` inspect went to the bounded `0870` family instead.

### 3. Consecutive Inspect Slots Stay Flat

The pass also tested whether `0008` needed inspect continuation rather than just one inspect.

Owned evidence says no:

- inspect-open-2 -> flat `0868`
- inspect-open-3 -> flat `0868`
- inspect-after-inspect in the mixed batch -> flat `0868`

So consecutive inspect does not look like the selector for `0008`.

### 4. The Repeatable Object-`1` Family In This Pass Was `0870`

Although `0008` vanished here, the pass still produced useful positive control.

The following all reached bounded `0870` object `1`:

- `26D000_no_inspect_after_inspect_4`
- `26D000_inspect_after_no_inspect_m2`
- `26D000_no_inspect_after_inspect_m4`

So the pass did not just lose the seam completely.

It showed that:

- `0870` remains a repeatable branch under these inspect/no-inspect opening shapes
- `0008` is the rarer and less stable family

### 5. `0008` Now Looks Like A Rarer Run-Family, Not A Simple Local-History Rule

After this pass, the best owned interpretation is:

- `0008` is real
- but it is not predicted by inspect alone
- and it is not predicted by opening-slot placement alone

That makes `0008` look more like:

- a rarer startup-side run family
- or a less frequently selected branch condition that is not captured by the immediate local history axes we have isolated so far

## Practical Interpretation

This pass is valuable because it closes a tempting but overly simple explanation.

We now know that `0008` is **not**:

- a generic inspect-opening outcome
- a generic inspect-slot-2 outcome
- or a generic consecutive-inspect outcome

That leaves us with a cleaner split:

- `0870` is the repeatable object-`1` family under several local-history shapes
- `0008` is real but rarer, and probably tied to a broader run family rather than the specific local slot patterns we just tested

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Rare-`0008` Reproduction Pass**

Focus:

- stop broad local-history sweeps
- directly replay and vary only the two known positive contexts for `0008`
- compare those against adjacent near-miss shapes
- aim to identify what broader run-family condition distinguishes:
  - `0008`
  - from nearby `0870`
  - and flat `0868`

## Bottom Line

Selector `0008` did not reproduce under deliberate inspect-led opening tests.

That means it is not a simple inspect-slot or opening-slot family, while `0870` remains the repeatable object-`1` branch under these shapes.
