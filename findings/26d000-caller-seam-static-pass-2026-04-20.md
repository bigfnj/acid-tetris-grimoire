# `0x26D000` Caller-Seam Static Pass

Date: 2026-04-20

## Summary

This pass moved the `0x26D000` investigation from runtime timing into static caller analysis around the already-closed chunk-7 frontend helper family.

Main result:

- the shared frontend producer seam is now statically closed through the present boundary
- every recovered frontend caller family ends its chunk-7 redraw tail with `0x17719 -> 0x24d0`
- the dominant late flat-`0868` `0x26D000` lane therefore sits **downstream** of the helper family, not inside `0x175c5`, `0x17613`, or `0x17719`

That matters because the runtime branch is already closed by scope rule:

- the final authorized runtime pass labeled the lane `flat-0868 dominant with bounded outliers below reproducibility threshold`
- so the next useful question is no longer "which post-`VRT` delay hits `0008` or `0870`?"
- it is "which caller tail after the shared present boundary reaches the later flat lane?"

## New Owned Artifact

- [26d000-caller-seam-static.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-caller-seam-static.json)

This artifact ties together:

- the closed runtime label from the `0x26D000` scope branch
- the shared chunk-7 producer contract
- the recovered frontend caller families that all converge on `0x17719 -> 0x24d0`
- the next static target after runtime-ladder closure

## Key Artifacts Reused

- [26d000-investigation-scope.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/26d000-investigation-scope.md)
- [26d000-opening-second-delay-seeding-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-opening-second-delay-seeding-pass-2026-04-20.md)
- [object2-entry-call-window-pass-2026-04-17.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/object2-entry-call-window-pass-2026-04-17.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin.000039c4.FUN_000039c4.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-update-followup/ATET.EXE.flat-relocated.bin.000039c4.FUN_000039c4.c)
- [ATET.EXE.flat-relocated.bin.000175c5.FUN_000175c5.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.000175c5.FUN_000175c5.c)
- [ATET.EXE.flat-relocated.bin.00017613.FUN_00017613.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.00017613.FUN_00017613.c)
- [ATET.EXE.flat-relocated.bin.00017719.FUN_00017719.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/chunk7-record-followup/ATET.EXE.flat-relocated.bin.00017719.FUN_00017719.c)
- [ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c)
- [ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c)
- [ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c)
- [ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c)
- [ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c)

## Findings

### 1. The Shared Frontend Producer Contract Is Now Closed Through Present

The helper-side contract is no longer ambiguous:

- `0x39c4` projects the active chunk-7 bank into record-local `x`, `y`, and shade fields
- `0x17613` conditionally clears the old point when the prior-frame draw-success latch at record `+0x18` was `1`
- `0x175c5` attempts the new draw, writes the pixel only if the target is still zero, and returns a fresh success bit
- the draw pass stores that return value back into record `+0x18`
- `0x17719` flushes the dirty `8x4` cell queue into the current VGA back page
- `0x24d0` immediately flips and throttles that page

That is the exact reusable frontend pointfield cadence now owned statically:

- clear previous points
- project current records
- draw new points
- flush dirty cells
- present one page

### 2. Every Recovered Frontend Caller Family Ends The Pointfield Tail With `0x17719 -> 0x24d0`

The convergence is broader than one menu:

- `0x3df4`
  `run_menu_entry_transition_frames`
  ends the transition frame with draw through `0x175c5`, flush through `0x3ebc -> 0x17719`, then present through `0x24d0`
- `0x3ee4`
  `run_menu_exit_transition_frames`
  matches the same tail at `0x3fad -> 0x17719 -> 0x24d0`
- `0x40c0`
  `run_main_menu`
  uses `0x4553 -> 0x17719`, then zeroes `EAX`, calls `0x24d0`, stores the returned catch-up count into `0x184cb`, and only then either loops back or exits through `0x3ee4`
- `0x4584`
  `run_options_menu`
  uses `0x4974 -> 0x17719`, then calls `0x24d0`, stores the returned catch-up count, and only then either loops back or exits through `0x3ee4`
- `0x49bc`
  `run_keyboard_setup_menu`
  is already closed by function hypothesis as using the same entry / steady / exit structure with `0x39c4`, `0x175c5`, `0x17719`, `0x24d0`, and `0x3ee4`, even though the current old export truncates after the entry-clear side
- `0x5a94`
  `run_sound_setup_menu`
  uses `0x5ec4 -> 0x17719`, then calls `0x24d0`, stores the returned catch-up count, and only then either loops back or exits through `0x3ee4`
- `0x62e0`
  `run_frontend_animation_fade_out`
  uses `0x6382 -> 0x17719`, then calls `0x24d0`, stores the returned catch-up count, and only then decides whether the time-based fade should continue
- `0x63b8`
  `run_frontend_animation_fade_in`
  uses `0x6463 -> 0x17719`, then calls `0x24d0`, stores the returned catch-up count, and only then decides whether the time-based fade should continue

So the currently owned frontend producer-side answer is:

- there is no recovered caller family where chunk-7 point redraw continues past `0x17719` without an immediate `0x24d0` present

### 3. This Moves The `0x26D000` Seam Out Of The Helper Bodies

The closed runtime branch now says:

- dominant bridge family: flat `0868`
- bounded outliers: `0008`, `0870`
- reproducibility threshold: not met

Combined with the static caller map above, the useful interpretation is sharper:

- the dominant `0x26D000` flat lane is not evidence of a hidden branch *inside* `0x175c5`, `0x17613`, or `0x17719`
- it is downstream of the shared `0x17719 -> 0x24d0` seam, or upstream in caller-family selection before the redraw tail even begins

That is exactly why further delay-ladder densification stopped paying off:

- the helper family was already the wrong search surface
- the real unresolved branch sits at the caller tail or later frontend dispatch path

### 4. The Next Static Move Is Now Mechanical

The next question should be:

- which post-`0x24d0` caller tail or later frontend dispatch path reaches the dominant late flat-`0868` lane?

That means the strongest next static targets are now:

- the post-present tails in:
  - `run_main_menu`
  - `run_options_menu`
  - `run_keyboard_setup_menu`
  - `run_sound_setup_menu`
  - `run_frontend_animation_fade_out`
  - `run_frontend_animation_fade_in`
- the shared exit chain through `0x3ee4`
- the frontend state dispatcher and its caller-family selection around `0x3830`

What should *not* be the next move:

- reopening the `0x26D000` timing ladder
- re-probing `0x175c5`, `0x17613`, or `0x17719` as if they still hid the dominant branch

## Practical Porting Impact

The frontend preservation model is tighter now:

- chunk-7 pointfield behavior is a producer pipeline ending at a shared present boundary
- menu-to-menu differences live in caller-local text, selection, setup, and exit logic
- the pointfield helpers themselves are shared backend-style presentation machinery, not per-menu policy

That supports a cleaner future port split between:

- shared chunk-7 projection / draw / flush code
- caller-local menu-state logic that decides when to continue, exit, or hand off

## Next Strongest Move

Do a focused post-present static pass on the caller tails:

1. recover the exact code after `0x24d0` in the dominant steady frontend loops
2. map how those tails loop, exit, or hand off into later frontend dispatch
3. only reopen runtime work if that static result isolates a materially different branch than the closed `0x26D000` timing ladder

## Bottom Line

The important closure is:

- the last statically recovered chunk-7 producer seam is `0x17719 -> 0x24d0`

So the dominant late `0x26D000` flat lane should now be chased in post-present caller logic, not inside the pointfield helper family.
