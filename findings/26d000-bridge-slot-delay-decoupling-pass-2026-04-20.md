# `0x26D000` Bridge-Slot Delay Decoupling Pass

Date: 2026-04-20

## Summary

This pass followed the exact-delay `0.18` precursor-correlation closure:

- no visible opening-slot selector family was necessary or sufficient
- no visible second-slot selector family was necessary or sufficient
- the useful bridge-side families therefore looked more bridge-local or late-session-local than simple precursor-gated

The narrow question here was:

- if the opening and second slots stay fixed at the working `0.18` lane, can varying only the bridge-slot post-`VRT` initial command delay reopen or widen the bridge-side object-`1` families

The three-slot shape stayed fixed:

- `no-inspect -> no-inspect -> inspect`

The opening and second slots were held constant at:

- `0.18`

I then varied only the bridge slot across a small ladder:

- `0.10`
- `0.18`
- `0.20`

with two fresh-clone repeats per point.

Main result:

- all six bridge runs stayed on flat `0868`
- no bridge run reached object `1`

So the best new closure is:

- bridge-local timing alone is not sufficient under the current lane shape
- the remaining live explanation has shifted away from pure bridge-local timing and toward coupled lane timing or broader late-session state

## New Owned Artifacts

- [26d000-bridge-slot-delay-decoupling-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-bridge-slot-delay-decoupling/26d000-bridge-slot-delay-decoupling-results-2026-04-20.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T172306Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T172452Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T172638Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T172824Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T173010Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260420T173157Z/batch-summary.json)

Fixture root:

- [bridge-slot-delay-decoupling-20260420T000000Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/bridge-slot-delay-decoupling-20260420T000000Z)

## Harness Preset

This pass added and used:

- `selector-26d000-bridge-slot-delay-decoupling-single-clone`

Tooling change:

- the batch harness can now read:
  - `OPENING_POST_VRT_INITIAL_COMMAND_DELAY_SECONDS`
  - `SECOND_POST_VRT_INITIAL_COMMAND_DELAY_SECONDS`
  - `BRIDGE_POST_VRT_INITIAL_COMMAND_DELAY_SECONDS`
- that lets the broader three-slot lane be replayed while only one slot’s post-`VRT` timing changes

## Results

### Bridge Delay `0.10`

#### Repeat 1

- `opening` -> `0868:00023A16`
- `second` -> `0868:0002B87C`
- `bridge` -> `0868:000258B0`

#### Repeat 2

- `opening` -> `0868:00015036`
- `second` -> `0868:0002B885`
- `bridge` -> `0868:0002B883`

### Bridge Delay `0.18`

#### Repeat 1

- `opening` -> `0868:0002482E`
- `second` -> `0868:000253D6`
- `bridge` -> `0868:0002B582`

#### Repeat 2

- `opening` -> `0868:00025389`
- `second` -> `0870:000003ED`
- `bridge` -> `0868:00025160`

### Bridge Delay `0.20`

#### Repeat 1

- `opening` -> `0868:0002556C`
- `second` -> `0868:000253D6`
- `bridge` -> `0868:00025105`

#### Repeat 2

- `opening` -> `0868:00023A55`
- `second` -> `0868:00023A53`
- `bridge` -> `0868:00023ACB`

### Bridge Descriptor Closure

All six bridge runs selected:

- selector `0868`
  - base `0x00000000`
  - limit `0xFFFFFFFF`

No bridge run selected:

- `0008`
- `0870`
- `0824`

### File Diffs

Clone tree hash diffs:

- `d0p10-r1-clone` -> `0`
- `d0p10-r2-clone` -> `0`
- `d0p18-r1-clone` -> `0`
- `d0p18-r2-clone` -> `0`
- `d0p20-r1-clone` -> `0`
- `d0p20-r2-clone` -> `0`

So this pass again observed no writes into the mounted clone trees.

## Findings

### 1. Bridge-Local Delay Alone Did Not Reopen Object-`1`

This is the main closure.

Across all six runs:

- `bridge delay 0.10` -> `0868`, `0868`
- `bridge delay 0.18` -> `0868`, `0868`
- `bridge delay 0.20` -> `0868`, `0868`

So varying only the bridge slot did not reopen either:

- bridge-side `0870`
- bridge-side `0008`

### 2. Earlier Visible Object-`1` Still Was Not Sufficient

Even in this decoupled sweep:

- one run reached opening-slot object `1`
- one run reached second-slot object `1`

But both of those runs still collapsed to bridge-side `0868`.

So this pass did not rescue a simple “earlier visible object-`1` is enough” model.

### 3. The Current Live Control Is Not Purely Bridge-Local

This is the most useful interpretation.

The earlier mixed bridge-side object-`1` families at uniform `0.18` were real.

But once:

- opening = `0.18`
- second = `0.18`

were held fixed and only the bridge slot was varied, the bridge collapsed completely.

That suggests the remaining live lever is not a simple local bridge delay in isolation.

This is an inference from the six-run decoupled sweep.

### 4. The Remaining Explanation Has Shifted Toward Coupled Timing

The most plausible surviving model after this pass is:

- coupled lane timing across the first two slots and the bridge slot
- or another broader late-session state that the bridge-only timing change does not recreate by itself

## Practical Interpretation

This pass rules out a tempting shortcut.

If the bridge-side object-`1` families had been controlled mainly by the bridge slot’s own post-`VRT` delay, at least one of these decoupled runs should have reopened `0870` or `0008`.

None did.

So the next best move is to keep the bridge fixed at the working value and instead vary the opening and second slots together, looking for an early-slot timing seed that the bridge-only sweep could not reproduce.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Opening/Second Delay Seeding Pass**

Focus:

- reuse the new per-slot delay preset
- hold the bridge slot fixed at:
  - `0.18`
- vary the opening and second slots together across a small ladder such as:
  - `0.10`
  - `0.18`
  - `0.20`
- preserve numeric selector capture on the bridge slot

The success condition is not just another sparse bridge-side object-`1` hit, but evidence that coupled early-slot timing can seed the later bridge family even when bridge-local timing stays constant.

## Bottom Line

Changing only the bridge slot was not enough.

With the opening and second slots held at `0.18`, all six bridge-delay variants collapsed to flat `0868`. The next strongest move is to test whether coupled early-slot timing, not pure bridge-local timing, is the real seed for the later bridge-side object-`1` families.
