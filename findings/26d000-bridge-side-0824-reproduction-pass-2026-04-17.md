# `0x26D000` Bridge-Side `0824` Reproduction Pass

Date: 2026-04-17

## Summary

This pass followed the probe-session progression result:

- session progression was not monotonic
- the highest-value new lead was a bridge-side object-`1` stop at `0824:00001003`

The narrow question here was:

- can that bridge-side `0824` family be reproduced in otherwise clean fresh single-clone Shape B runs if the bridge slot collects explicit numeric selector evidence

The short Shape B lane stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran six separate fresh-clone single-session batches with a new bridge-focused preset.

Main result:

- no bridge-side `0824` reproduced
- every bridge run flattened to `0868` in object `3`
- the only object-`1` leak in the whole sweep appeared one slot earlier, on clone 5 second `no-inspect`, at `0870:00000C97`

So the best new closure is:

- the rare `0824` bridge family does not reopen under clean fresh-clone bridge isolation alone
- and a pre-bridge object-`1` stop is not sufficient by itself to promote the bridge into object `1`

## New Owned Artifacts

- [26d000-bridge-side-0824-reproduction-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-bridge-side-0824-reproduction/26d000-bridge-side-0824-reproduction-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T234915Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T235101Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T235248Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T235434Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T235620Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T235806Z/batch-summary.json)

Fixture root:

- [shape-b-bridge-0824-repro-20260418T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-bridge-0824-repro-20260418T000000Z)

## Harness Preset

This pass added and exercised:

- `selector-26d000-shape-b-bridge-0824-repro-single-clone`

Tooling fix discovered during the pass:

- `run_branch_family_batch.py` now parses numeric `selinfo` output even when DOSBox leaves the descriptor label blank

That mattered here because bridge-side numeric selector queries emitted usable descriptor lines for `0008`, `0870`, and `0868`, but the older parser would have dropped them.

## Results

### Clone 1

- `no-inspect` -> `0868:00025395`
- `no-inspect-again` -> `0868:0002B883`
- `bridge` -> `0868:0002B87C`

### Clone 2

- `no-inspect` -> `0868:00025393`
- `no-inspect-again` -> `0868:0002B882`
- `bridge` -> `0868:00025395`

### Clone 3

- `no-inspect` -> `0868:00025160`
- `no-inspect-again` -> `0868:00024812`
- `bridge` -> `0868:0002515B`

### Clone 4

- `no-inspect` -> `0868:000253C0`
- `no-inspect-again` -> `0868:0002B87C`
- `bridge` -> `0868:0002B882`

### Clone 5

- `no-inspect` -> `0868:0002B883`
- `no-inspect-again` -> `0870:00000C97`
- `bridge` -> `0868:0002B824`

### Clone 6

- `no-inspect` -> `0868:0002514F`
- `no-inspect-again` -> `0868:0002B885`
- `bridge` -> `0868:000239FC`

### Bridge-Side Numeric Selector Evidence

All six bridge logs consistently preserved:

- `0008` -> base `0x00008240`, limit `0x0000FFFF`
- `0870` -> base `0x0000A3C0`, limit `0x0000FFFF`
- `0868` -> base `0x00000000`, limit `0xFFFFFFFF`

And all six bridge logs consistently failed to return a populated numeric `selinfo 0824` descriptor.

### File Diffs

Clone tree hash diffs:

- `clone-1` -> `0`
- `clone-2` -> `0`
- `clone-3` -> `0`
- `clone-4` -> `0`
- `clone-5` -> `0`
- `clone-6` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. Bridge-Side `0824` Did Not Reproduce

This is the main closure.

Across six clean fresh single-clone sessions:

- every bridge run stayed on `0868`
- every bridge run stayed in object `3`

So the rare session-4 bridge-side `0824` family is not reopened by clean bridge isolation alone.

### 2. The Strongest Remaining Live Signal Fell Back One Slot Earlier

Only one run in the whole sweep reached object `1`:

- clone 5 second `no-inspect` -> `0870:00000C97`

That means the strongest surviving live family is still pre-bridge, not bridge-side.

### 3. A Pre-Bridge Object-`1` Stop Is Not Sufficient To Promote The Bridge

Clone 5 is the key comparison.

Even after reaching:

- `0870:00000C97`

the following bridge still flattened to:

- `0868:0002B824`

So “arrive in object `1` one slot earlier” is not enough by itself to produce a bridge-side object-`1` result.

### 4. Numeric Bridge Capture Worked, But `0824` Stayed Blank

This is the main tooling-side closure.

The bridge logs consistently returned numeric selector descriptors for:

- `0008`
- `0870`
- `0868`

but not for:

- `0824`

That does not prove `0824` is impossible. But it does show that the session-4 `0824` lead is not behaving like a stable bridge-side descriptor that can simply be queried on demand in clean flat-`0868` bridge runs.

This is an inference from the captured debugger output.

### 5. The Differentiating State Still Is Not In The Clone Trees

All six clone hash diffs stayed empty.

So this pass again rules out:

- clone-local on-disk mutation

as the cause of the bridge-side rarity.

## Practical Interpretation

This pass usefully narrows the target.

We now know:

- bridge-side `0824` is rarer than the second-slot object-`1` families under clean fresh-clone single-session conditions
- the bridge does not follow automatically just because the preceding slot reached object `1`
- and the session-4 `0824` lead likely belongs to a broader anomaly family, not to clean bridge isolation by itself

So the next best move is to stop treating `0824` as a pure bridge-only reproduction problem and instead test whether it depends on the broader session-4 probe anomalies.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Session-4 Anomaly Correlation Pass**

Focus:

- repeat the broader single-clone session-progression shape instead of isolated bridge-only sweeps
- watch specifically for:
  - `post_vrt_rebreak_not_observed`
  - `vrt_selector_not_discovered`
- test whether bridge-side `0824` only reappears when one of those earlier probe-flow anomalies is also present

## Bottom Line

The bridge-side `0824` family did not reproduce across six fresh single-clone Shape B sessions.

Every bridge flattened to `0868`, even when one run reached `0870` one slot earlier. The best remaining explanation is that the original `0824` lead belongs to a broader session-anomaly family rather than to clean bridge isolation alone.
