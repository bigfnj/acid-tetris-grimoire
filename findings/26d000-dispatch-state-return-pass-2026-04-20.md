# `0x26D000` Dispatch-State Return Pass

Date: 2026-04-20

## Summary

This pass recovered the missing end of `0x3ee4` and the concrete `0x3830` jump-table state map.

Main result:

- `0x3ee4` is now closed as a complete 48-frame exit transition that returns to its caller only after the frame counter reaches `0x30`
- `0x3830` now has a concrete state map for entries `0..10`
- handler-family returns are no longer anonymous: the dispatcher copies the handler result from `EAX` into `ECX`, then redispatches until a direct exit state sets `ESI = 1`

That matters because the late `0x26D000` static target is now a named handoff problem:

- steady handlers return a next frontend state
- direct exit states `2` and `4` set the shared exit condition
- `0x3830` then runs the already-recovered shared fade-out / restore tail

## New Owned Artifact

- [26d000-dispatch-state-return.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-dispatch-state-return.json)

This artifact records:

- the full recovered `0x3ee4` tail
- the `0x3804` jump-table entries
- the now-concrete frontend state meanings that sit after the steady post-present tails

## Key Artifacts Reused

- [26d000-post-present-caller-tail-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-post-present-caller-tail-pass-2026-04-20.md)
- [26d000-caller-seam-static-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-caller-seam-static-pass-2026-04-20.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin)
- [ATET.EXE.flat-relocated.bin.00003830.FUN_00003830.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/startup-followup-pass/ATET.EXE.flat-relocated.bin.00003830.FUN_00003830.c)
- [ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c)
- [ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c)
- [ATET.EXE.flat-relocated.bin.000049bc.FUN_000049bc.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000049bc.FUN_000049bc.c)

## Findings

### 1. `0x3ee4` Is A Complete 48-Frame Exit Transition, Not Just A Shared Helper Prefix

The recovered flat disassembly now closes the function boundary directly:

- `0x3ee4 .. 0x3f17`
  clear previously drawn chunk-7 points through `0x17613`
- `0x3f2b .. 0x3f64`
  redraw the eight menu rows through `0x60cc` using `max(0x2f - frame_index, 0)`
- `0x3f66`
  advance chunk-7 records through `0x39c4`
- `0x3f85 .. 0x3fab`
  redraw the pointfield through `0x175c5`
- `0x3fad`
  flush through `0x17719`
- `0x3fb8`
  present through `0x24d0`
- `0x3fbd`
  store the returned pacing value into `0x184cb`
- `0x3fc2`
  compare frame counter against `0x30`
- `0x3fcb .. 0x3fd4`
  return only after the transition completes

So the local caller-side meaning is now explicit:

- a steady menu does **not** directly hand back to `0x3830` on the same frame that it arms exit
- it first runs a full presented exit transition through `0x3ee4`

### 2. The `0x3804` Jump Table Now Has A Concrete State Map

The dispatcher entries read cleanly as:

1. state `0`
   `0x3938`
   no-op redispatch slot
2. state `1`
   `0x38bf`
   call `0x40c0`
   `run_main_menu`
3. state `2`
   `0x38c9`
   set row index `0x1886f = 7`, set `ESI = 1`, then fall into the shared dispatcher tail
   `resume_gameplay_exit`
4. state `3`
   `0x38d8`
   music fade-out and DOS exit path
5. state `4`
   `0x3906`
   set `ESI = 1`, then fall into the shared dispatcher tail
   `start_new_game_exit`
6. state `5`
   `0x390a`
   call `0x5a94`
   `run_sound_setup_menu`
7. state `6`
   `0x3911`
   call `0x4584`
   `run_options_menu`
8. state `7`
   `0x3918`
   call `0x49bc`
   `run_keyboard_setup_menu`
9. state `8`
   `0x391f`
   call `0x50b0` with `EAX = EBX`
   plain high-scores display
10. state `9`
    `0x3928`
    call `0x50b0` with `EAX = EDI`
    qualifying-name-entry high-scores flow
11. state `10`
    `0x3931`
    call `0x4f38`
    `run_credits_sequence`

That replaces the last fuzzy part of the frontend return path with a concrete state table.

### 3. Handler Returns Now Have A Closed Dispatcher Contract

The dispatcher-side contract is now explicit:

- handler-style states return their next state in `EAX`
- `0x3936` copies `EAX` into `ECX`
- `0x3938` checks `ESI`
- if `ESI != 1`, dispatch continues through the jump table
- if `ESI == 1`, the shared exit tail at `0x3941 .. 0x39bd` runs

That means the remaining late-lane static question is no longer "what does the dispatcher generally do?"
It is narrower:

- which handler return values and direct exit-state stubs lead into the late flat lane before or during the shared exit tail?

### 4. This Narrows The Next Static Target Again

The best remaining code surfaces are now:

- the exact return values produced by:
  - `run_main_menu`
  - `run_options_menu`
  - `run_keyboard_setup_menu`
  - `run_high_scores_screen`
  - `run_credits_sequence`
  - `run_sound_setup_menu`
- the post-`0x3ee4` return edges inside the steady caller families
- the direct exit-state stubs at:
  - `state 2`
  - `state 4`

What is now much lower value:

- generic frontend dispatcher archaeology
- any renewed attempt to treat the `0x26D000` branch as a mystery hidden inside the pointfield helper family

## Practical Porting Impact

The frontend state model is now strong enough to structure a future port around:

- steady handler states that return a next state
- direct exit states for resume-game and new-game
- a shared dispatcher tail for fade-out, snapshot restore, and gameplay handoff

That is a faithful behavioral split, not just an implementation convenience.

## Next Strongest Move

Do a focused handler-return pass:

1. map the concrete `EAX` return values emitted by the steady frontend handlers
2. identify which ones feed states `2` or `4` most directly
3. correlate that return path against the closed late flat-`0868` lane

## Bottom Line

The important closure is:

- `0x3ee4` and `0x3830` are now concrete enough that the late `0x26D000` branch can be chased as a named state-return path

That is a much tighter target than the earlier generic "post-present caller tail" framing.
