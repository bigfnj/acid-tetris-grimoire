# Pre-Threshold Breakpoint Family Pass

Date: 2026-04-17

## Summary

This pass moved off the late object-2 helper lane and tried to catch the handoff earlier by:

- reusing the repeatable no-autoexec live-stop window that now lands in object `1`
- arming the two strongest closed object-1 seam callsites
- and checking whether either seam is reachable before the probe times out

Main result:

- the object-1 pre-threshold live-stop window is real and repeatable
- but neither `0x03DAC` nor `0x03DDC` fired from that window within the tested `LOGC` budgets
- so the current cold object-1 lane is still not the right direct closure path to the frontend seam

## New Owned Artifact

- [pre-threshold-breakpoint-family-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/pre-threshold-breakpoint-family/pre-threshold-breakpoint-family-results-2026-04-17.json)

## Probe Control

All runs used the native WSL protected-mode probe with the same cold startup lane:

- `DOSBOX_X_BIN=.tools/bin/dosbox-x-linux-debug`
- no `AUTOTYPE`
- no extra `ATET.EXE` args
- `--post-vrt-log-mode logc`
- `--absolute-timeout-seconds 35`

The only variables were:

- armed target `0x03DAC` vs `0x03DDC`
- staged `LOGC` budget `0x180000` vs `0x200000`

## Targeted Seam Family

This pass used the already-closed object-1 to object-2 seam from the earlier call-window pass:

- `0x03DAC -> 0x17613`
  - conditional clear side
- `0x03DDC -> 0x175C5`
  - unconditional draw side

Those are the strongest known object-1 frontend seam callsites before the later object-2 helper cluster.

## Results

- `budget180_clear_3dac`
  - live stop `0870:000003EB`
  - object `1` relative offset `0x3EB`
  - armed `0870:03DAC`
  - result: no hit before timeout
- `budget180_draw_3ddc`
  - live stop `0870:00000433`
  - object `1` relative offset `0x433`
  - armed `0870:03DDC`
  - result: no hit before timeout
- `budget200_clear_3dac`
  - live stop `0870:000003B7`
  - object `1` relative offset `0x3B7`
  - armed `0870:03DAC`
  - result: no hit before timeout
- `budget200_draw_3ddc`
  - live stop `0870:000003FA`
  - object `1` relative offset `0x3FA`
  - armed `0870:03DDC`
  - result: no hit before timeout

Across all four runs:

- the selected live stop stayed inside object `1`
- the live-stop window stayed bounded to `0x3B7..0x433`
- `prompt_return_within_timeout` stayed false
- `target_display_hits_after_arm` stayed empty

## Findings

### 1. A Useful Object-1 Pre-Threshold Window Now Exists

This pass confirmed that the cold no-autoexec lane can reliably re-enter protected-mode code in object `1`, not just late object `2` or object `3`.

That matters because it gives us an owned live-stop family earlier than the current late frontend glyph-draw floor.

### 2. The Strongest Closed Seam Calls Still Did Not Fire From That Window

Even with the live stop already in object `1`, arming:

- `0x03DAC`
- `0x03DDC`

did not produce a breakpoint hit in either staged budget.

So the current cold lane is still not traversing the known frontend seam quickly enough for direct closure.

### 3. This Is Path Evidence, Not Global Unreachability

These no-hits are still valuable, but they should be read carefully.

They do **not** prove:

- that `0x03DAC` or `0x03DDC` can never fire at runtime

They **do** prove:

- the specific no-autoexec object-1 lane used here does not reach those seam calls within the tested budgets

That is a tighter and more useful statement.

### 4. The Current Window Is Probably Too Early Or On The Wrong Branch Family

The live-stop range `0x3B7..0x433` is materially earlier than the seam callsites near `0x3DAC` and `0x3DDC`.

Given the repeated no-hit result, the strongest interpretation is now:

- this window is either still too early to be a practical direct-arm point
- or it belongs to a cold startup branch family that never reaches the relevant frontend transition loop under the tested conditions

Either way, it is not yet the closure lane we want.

## Practical Interpretation

This pass separated two things that were previously blurred together:

- late object-2 landings are too late to mean much
- but an earlier object-1 live-stop alone is still not enough if the lane itself is wrong

That is useful closure.
It means the next pass should extract more state from the early object-1 window or move to a nearby breakpoint family that can tell us which branch chain this lane actually belongs to.

## Recommended Next Move

The strongest next move is now:

- **Step 8: Runtime-State Snapshot Pass**

Reason:

- we now have a stable object-1 live-stop window
- but the seam calls do not fire from it under the cold lane
- so the highest-value next step is to capture more runtime state at that window before choosing the next breakpoint family

## Bottom Line

This pass successfully established a repeatable object-1 pre-threshold live-stop window.

It also showed that simply arming the strongest known seam callsites from that window is not enough: neither `0x03DAC` nor `0x03DDC` fired before timeout.

That closes the straightforward version of the early-arm idea and sets up runtime-state capture as the next strongest move.
