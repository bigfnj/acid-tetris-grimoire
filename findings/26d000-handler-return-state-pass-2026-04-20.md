# `0x26D000` Handler-Return State Pass

Date: 2026-04-20

## Summary

This pass recovered the concrete `EAX` return states emitted by the steady frontend handlers.

Main result:

- the steady handlers now have a closed return-state map
- only `run_main_menu` and `run_sound_setup_menu` can emit gameplay-facing exit states `2` or `4`
- all other recovered steady handlers return to another frontend state, most often state `1`

That sharply narrows the best remaining static search surface for the late flat-`0868` `0x26D000` lane.

## New Owned Artifact

- [26d000-handler-return-state.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-handler-return-state.json)

This artifact records:

- the recovered handler return states
- the local row or action that stages each return
- which handlers can feed dispatcher exits `2` or `4`

## Key Artifacts Reused

- [26d000-dispatch-state-return-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-dispatch-state-return-pass-2026-04-20.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin)
- [ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000040c0.FUN_000040c0.c)
- [ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00004584.FUN_00004584.c)
- [ATET.EXE.flat-relocated.bin.000049bc.FUN_000049bc.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000049bc.FUN_000049bc.c)
- [ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00005a94.FUN_00005a94.c)

## Findings

### 1. `run_main_menu` Is The Only Recovered Steady Handler That Emits State `4`

The main-menu row jump table at `0x40a0` now closes as:

- row `0`
  New Game
  stages return state `4`
- row `1`
  Options
  stages return state `6`
- row `2`
  Music
  no exit; cycles the track and rewrites the visible row
- row `3`
  Level
  no exit; increments the starting level and rewrites the visible row
- row `4`
  High Scores
  stages return state `8`
- row `5`
  Credits
  stages return state `10`
- row `6`
  Exit Game
  stages return state `3`
- row `7`
  Return to Game
  stages return state `2` only when the live-game flag at `0x2c72b` is `1`

So the main menu is the only currently recovered steady handler that can emit:

- state `4`
  start-new-game exit

It can also emit:

- state `2`
  resume-gameplay exit
- state `3`
  exit to DOS
- states `6`, `8`, and `10`
  frontend redispatches

### 2. `run_sound_setup_menu` Is The Other Handler That Can Emit State `2`

The sound-setup Enter jump table at `0x5a7c` closes as:

- row `0`
  cycle output device
  no exit
- row `1`
  cycle mixing rate
  no exit
- row `2`
  toggle stereo / mono
  no exit
- row `3`
  toggle 16-bit / 8-bit
  no exit
- row `4`
  Play Game
  stages return state `2`
- row `5`
  Exit to Dos
  stages return state `3`

Its Esc path also stages:

- state `2`

So the sound-setup handler can feed the shared dispatcher exit tail through:

- state `2`
- state `3`

but never state `4`.

### 3. `run_options_menu` Only Returns To Other Frontend States

The options handler stages only two concrete return states:

- Esc
  return state `1`
  main menu
- Enter on row `2`
  Keyboard Setup
  return state `7`
- Enter on row `3`
  Back To Main Menu
  return state `1`

Rows `0` and `1` only mutate values in place:

- Music Volume
- Sound FX Volume

So options is not a candidate producer for dispatcher exits `2` or `4`.

### 4. `run_keyboard_setup_menu` Mostly Returns To Frontend State `6`

The keyboard-setup handler uses two distinct local exit behaviors:

- Esc
  stages return state `1`
- Enter on rows `0..4`
  enters the dedicated capture substate
  no final dispatcher return yet
- Enter on row `5`
  Back to Options Menu
  stages return state `6`

So keyboard setup also does **not** feed dispatcher exits `2` or `4`.

One small but useful clarification:

- "Back to Options Menu" is the only normal steady exit to state `6`
- Esc is a different path and stages state `1`

### 5. `run_credits_sequence` And `run_high_scores_screen` Return To State `1`

The remaining recovered steady handlers now close cleanly:

- `run_credits_sequence`
  Esc stages return state `1`
  otherwise the page sequence auto-advances and eventually continues the same cycle
- `run_high_scores_screen`
  plain table display exits to state `1`
  qualifying-name-entry also ends at state `1` after commit and the footer-exit loop

So neither handler feeds the gameplay-facing dispatcher exits.

### 6. The Remaining Gameplay-Facing Search Surface Is Now Tiny

Across the recovered steady handlers:

- state `4` comes from:
  - `run_main_menu` only
- state `2` comes from:
  - `run_main_menu`
  - `run_sound_setup_menu`

Everything else returns only to:

- state `1`
- state `6`
- state `7`
- state `8`
- state `10`
- or local non-exit substates

That means the next late-lane static target should now concentrate on:

- main-menu return-state staging
- sound-setup return-state staging
- the shared dispatcher tail they feed afterward

Those are the only recovered steady handlers that can currently hand control into gameplay-facing dispatcher exits.

## Practical Porting Impact

The frontend state graph is now concrete enough to preserve directly:

- main menu owns the New Game / Resume / Exit / submenu graph
- sound setup owns the early "continue or exit" graph
- other steady handlers are frontend-only and rejoin menu flow rather than gameplay flow

That is a much stronger behavior model than a generic "all menus return a next state."

## Next Strongest Move

Do a focused pass on the gameplay-facing return edges only:

1. isolate the exact main-menu code paths that stage states `2` and `4`
2. isolate the exact sound-setup code paths that stage state `2`
3. correlate those exit edges against the late flat-`0868` lane and shared dispatcher tail

## Bottom Line

The important closure is:

- only two recovered steady handlers can emit gameplay-facing exits at all

So the `0x26D000` late-lane static chase should now focus on `run_main_menu` and `run_sound_setup_menu`, not the rest of the frontend handler family.
