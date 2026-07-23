# `0x26D000` Post-Present Caller-Tail Pass

Date: 2026-04-20

## Summary

This pass followed the newly closed `0x17719 -> 0x24d0` frontend seam one layer later into the caller tails and the shared frontend dispatcher.

Main result:

- after `0x24d0`, the recovered frontend tails split cleanly into two control families
- steady menu/fade callers first store the returned pacing value into `0x184cb`, then either loop locally or hand off into a shared exit transition
- after handlers finish, `0x3830` owns the common fade-out, snapshot restore, and gameplay-return / new-game handoff tail

That moves the next static target from "anything after the helper family" to a smaller concrete set:

- the local post-`0x24d0` loop/exit blocks in the steady callers
- the end of `0x3ee4`
- the redispatch and shared exit tail in `0x3830`

## New Owned Artifact

- [26d000-post-present-caller-tail.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-post-present-caller-tail.json)

This artifact records:

- the exact post-present blocks recovered in the steady frontend callers
- the shared dispatcher bootstrap and exit tail
- the mechanical narrowing of the next static target set

## Key Artifacts Reused

- [26d000-caller-seam-static-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-caller-seam-static-pass-2026-04-20.md)
- [26d000-investigation-scope.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/26d000-investigation-scope.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin.00003830.FUN_00003830.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/startup-followup-pass/ATET.EXE.flat-relocated.bin.00003830.FUN_00003830.c)
- [ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c)
- [ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c)
- [ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c)
- [ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c)
- [ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c)

## Findings

### 1. The Steady Frontend Callers Use `0x24d0` As A Local Pacing And Branch Point

The already-closed redraw tail is immediately followed by caller-local control logic.

Recovered steady-menu examples:

- `0x40c0` `run_main_menu`
  - `0x4553 -> 0x17719`
  - `0x455d -> 0x24d0`
  - `0x4562` stores the returned value into `0x184cb`
  - `0x4567` tests the local exit flag
  - `0x456f` calls `0x3ee4` only when exit is armed; otherwise control loops back to `0x41fa`
- `0x4584` `run_options_menu`
  - `0x4974 -> 0x17719`
  - `0x497f -> 0x24d0`
  - `0x4984` stores the returned value into `0x184cb`
  - `0x4989` tests the local exit flag
  - `0x4991` calls `0x3ee4` only when exit is armed; otherwise control loops back to `0x463b`
- `0x5a94` `run_sound_setup_menu`
  - `0x5ec4 -> 0x17719`
  - `0x5ecf -> 0x24d0`
  - `0x5ed4` stores the returned value into `0x184cb`
  - `0x5ed9` tests the local exit flag
  - `0x5ee1` calls `0x3ee4` only when exit is armed; otherwise control loops back to `0x5bc5`

So the shared meaning of the post-present block in the menu-family states is now clear:

- refresh the catch-up budget
- decide whether to keep running the same steady menu
- or enter the shared exit transition

### 2. The Time-Based Frontend Fades Use The Same Post-Present Pattern But Loop On Elapsed Ticks

The fade helpers are structurally parallel, but their local continuation rule is time-based rather than exit-flag based:

- `0x62e0` `run_frontend_animation_fade_out`
  - `0x6382 -> 0x17719`
  - `0x6389 -> 0x24d0`
  - `0x638e` stores the returned value into `0x184cb`
  - `0x6393..0x63a2` compares elapsed ticks against `0x40`
  - if still under budget, control loops back to `0x62fa`
- `0x63b8` `run_frontend_animation_fade_in`
  - `0x6463 -> 0x17719`
  - `0x646a -> 0x24d0`
  - `0x646f` stores the returned value into `0x184cb`
  - `0x6474..0x6483` compares elapsed ticks against `0x40`
  - if still under budget, control loops back to `0x63d2`

That closes the second post-present control family:

- menu-family callers loop on local state / exit flags
- fade-family callers loop on elapsed time

### 3. Exiting A Menu Does Not Bypass The Shared Frontend Dispatcher

The recovered local tails do not jump directly from a steady menu into gameplay.

Instead:

- the steady menu-family callers enter `0x3ee4` for the visible exit transition
- after handler completion, `0x3830` owns the shared frontend exit tail

The dispatcher-side structure is now concrete:

- `0x3830 .. 0x38a2`
  saves the gameplay snapshot, fades out the current palette, copies the title/menu base into the working and visible pages, then runs the frontend fade-in through `0x63b8`
- `0x38ae .. 0x3938`
  redispatches frontend states through the jump table at `0x3804`
- `0x3941 .. 0x39bd`
  is the shared post-handler exit tail:
  - clear release latch through `0x094c`
  - run frontend fade-out through `0x62e0`
  - save settings through `0x3604`
  - restore the gameplay snapshot into the working and visible pages
  - for state `4`, also call `0x2d88` and `0x05e0`
  - then run the gameplay palette reveal through `0x6498` and `0x2574`

So the next unresolved branch surface after the local menu tail is not an unknown direct jump.
It is a known shared dispatcher tail.

### 4. This Narrows The Late `0x26D000` Search Surface Further

The remaining static target set is now smaller and more specific than after the seam pass:

- local post-`0x24d0` loop/exit blocks inside:
  - `run_main_menu`
  - `run_options_menu`
  - `run_keyboard_setup_menu`
  - `run_sound_setup_menu`
- the return edge of `run_menu_exit_transition_frames`
- the redispatch and shared exit tail in `run_frontend_state_transition`

What is now lower value:

- revisiting the helper family as if the late flat lane still hid inside `0x175c5`, `0x17613`, or `0x17719`
- reopening the closed runtime delay ladder without a new static branch hypothesis

## Practical Porting Impact

The frontend structure is now better separated into three layers:

- shared pointfield producer / flush / present machinery
- caller-local steady-loop policy
- shared dispatcher bootstrap / fade-out / restore policy

That is a cleaner future port architecture than a single monolithic "frontend loop."

## Next Strongest Move

Do a focused static pass on the exact handoff path from steady caller to dispatcher:

1. recover the end of `0x3ee4`
2. map which handler return values feed which `0x3830` states
3. identify which state-local tail is the best static candidate for the dominant late flat-`0868` lane

## Bottom Line

The important new closure is:

- post-`0x24d0` control is already split into known local-loop tails and a known shared dispatcher tail

So the late `0x26D000` branch should now be chased in those tails, not anywhere earlier in the chunk-7 producer pipeline.
