# `0x26D000` Shape-B Carry-Decay Isolation Pass

Date: 2026-04-17

## Summary

This pass followed the Shape B stabilization result:

- the fixed Shape B bridge had looked like it decayed from `0008` to `0870` to flat `0868`

The narrow question here was:

- can that apparent decay be reset by interrupting the Shape B sequence with a different short lane

The batch shape was:

- two Shape B repeats
- one short Shape A interrupt lane
- one resumed Shape B repeat

Main result:

- the Shape B baseline was already fully collapsed to flat `0868`
- the Shape A interrupt still reproduced both object-`1` families:
  - `0008`
  - `0870`
- the resumed Shape B bridge stayed flat `0868`

So the strongest new closure is:

- the interruption did not reset Shape B
- and the earlier “carry-decay ladder” is probably not just an in-process runtime effect

## New Owned Artifacts

- [26d000-shape-b-carry-decay-isolation-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-shape-b-carry-decay-isolation/26d000-shape-b-carry-decay-isolation-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T222322Z/batch-summary.json)

## Harness Preset

This pass added and exercised:

- `selector-26d000-shape-b-carry-decay-interrupt-shape-a`

## Results

### Shape B Baseline Repeat 1

- `26D000_decay_b1_no_inspect`
  - `0868:0002B87C`
  - object `3`
- `26D000_decay_b1_no_inspect_again`
  - `0868:0002B826`
  - object `3`
- `26D000_decay_b1_bridge`
  - `0868:000238F7`
  - object `3`

### Shape B Baseline Repeat 2

- `26D000_decay_b2_no_inspect`
  - `0868:0002B87F`
  - object `3`
- `26D000_decay_b2_no_inspect_again`
  - `0868:0002B880`
  - object `3`
- `26D000_decay_b2_bridge`
  - `0868:00023A00`
  - object `3`

### Shape A Interrupt Lane

- `26D000_decay_interrupt_a_no_inspect`
  - `0868:0002B821`
  - object `3`
- `26D000_decay_interrupt_a_inspect_after_no_inspect`
  - `0008:00000C6C`
  - object `1`
- `26D000_decay_interrupt_a_bridge`
  - `0870:00000425`
  - object `1`

### Resumed Shape B After Interrupt

- `26D000_decay_resume_b_no_inspect`
  - `0868:000253F4`
  - object `3`
- `26D000_decay_resume_b_no_inspect_again`
  - `0868:000253F2`
  - object `3`
- `26D000_decay_resume_b_bridge`
  - `0868:0002515B`
  - object `3`

## Findings

### 1. The Shape B Interruption Did Not Reset The Bridge

This is the first thing to say clearly.

The resumed Shape B bridge landed at:

- `0868:0002515B`

So the inserted Shape A lane did not restore:

- rare `0008`
- or bounded `0870`

At least under this tested interruption shape, Shape B stayed collapsed.

### 2. Shape B Was Already Collapsed Before The Interrupt

This matters even more than the resumed result.

Both baseline Shape B bridges were already flat:

- `b1_bridge` -> `0868:000238F7`
- `b2_bridge` -> `0868:00023A00`

So this batch did not replay the earlier `0008 -> 0870` ladder at all.

That means the prior ladder is not a guaranteed baseline for every fresh interruption experiment.

### 3. Shape A Stayed Fully Live Inside The Same Batch

This is the strongest positive control.

Inside the same batch where Shape B was fully collapsed, Shape A still produced:

- `0008:00000C6C`
- `0870:00000425`

That tells us the flat Shape B outcome is not just:

- “the whole environment can no longer reach object `1`”

It is more specific than that.

### 4. The Best Explanation Is No Longer Pure In-Process Carry

This is an inference from the harness design.

Each batch item is run by a fresh child invocation of:

- `run_dosbox_protected_mode_breakpoint_probe.py`

So the items in the batch do not share one long-running DOSBox process.

That means the difference between:

- the earlier ordered Shape B ladder
- and this fully collapsed Shape B batch

is more consistent with:

- shared persisted state
- or another cross-run external effect

than with a purely in-memory runtime carry alone.

### 5. The Frontier Has Shifted Toward Game-Directory Freshness

After this pass, the best question is no longer:

- “what interruption resets Shape B?”

We do not have evidence that this kind of interruption resets it.

The sharper question is now:

- “does a fresh disposable copy of the game directory restore the early Shape B bridge families?”

That is the most direct way to test the persisted-state explanation.

## Practical Interpretation

This pass ruled out one promising reset idea.

We now know:

- Shape A interruption does not restore Shape B
- Shape B can be fully collapsed even when Shape A remains live
- the observed run-family differences are likely crossing process boundaries

So the next pass should move from lane-order experiments to freshness and persistence experiments.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Shape-B Fresh-Clone Reset Pass**

Focus:

- create disposable fresh copies of `Original.Game`
- run the short Shape B bridge lane against each fresh clone
- compare whether the bridge returns to:
  - `0008`
  - `0870`
  - or stays flat `0868`
- record any file mutations or hash changes before and after the runs

## Bottom Line

The Shape A interruption did not reset Shape B.

In this batch, Shape B was already fully collapsed to flat `0868`, while Shape A still reproduced both `0008` and `0870`. The strongest current model is now a cross-run persisted-state effect, not a simple in-process carry ladder.
