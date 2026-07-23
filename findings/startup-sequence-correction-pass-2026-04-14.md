# Startup Sequence Correction Pass

Date: 2026-04-14

## Summary

This pass corrects the earlier same-day "startup direct gameplay" interpretation.

The important result is:

- the early startup gate at `0x0172` only controls an optional **first frontend entry** into state `5` `sound setup`
- it does **not** prove that normal startup skips the frontend entirely
- later in the same top-level runtime path, startup **always** enters `0x3830` again with state `1` after splash/resource/music setup

So the shipped cold-boot model is not "gameplay first."
It is a two-stage startup:

- optional early `sound setup`
- then later normal `main menu`

## The Correct Top-Level Startup Order

The raw flat-relocated startup path rooted at `0x00000018` now reads cleanly in this order:

1. `0x5fd8`
   load frontend animation resources from chunks `5` and `7`
2. `0x5f78`
   preload the title/logo base screen from chunk `4`, clear dirty state, seed selected row `0`
3. `0x36dc`
   load or initialize `SETUP.DAT`
4. `0x68d8`
   arm the configured audio service layer
5. optional early frontend detour:
   - if explicit `setup` mode is requested, or `SETUP.DAT` had to be initialized
   - call `0x3830(5)`
   - state `5` is `0x5a94` `sound setup`
6. `0x668c`
   initialize the selected audio backend, with fallback to device `4` `None` on failure
7. `0x3370` + `0x67f8`
   load and register the twelve WAV SFX slots
8. `0x2998(0)`
   show the `DDD Dungeon Dweller Designs` splash
9. `0x2998(2)`
   show the warning splash
10. load chunk `6`
    alert-face bank
11. load chunk `1`
    gameplay palette plus compressed gameplay base screen, then upload/save its alert-region backing data
12. load chunk `3`
    gameplay auxiliary art and lookup data
13. load chunk `8`
    `GAME OVER` overlay resource
14. `0x6544`
    start the selected music track
15. `0x3830(1)`
    enter the normal frontend dispatcher path in state `1`
16. inside `0x3830`, the `New Game` exit state `4` runs:
    - `0x2d88`
    - `0x05e0`
17. only after that does control return to the outer gameplay/session loop rooted at `0x023b`

## What A Player Actually Sees

This produces a much more faithful visible model.

Normal launch with valid setup:

- DDD splash
- warning splash
- title/main menu
- `New Game`
- gameplay

Explicit `setup` launch or missing/invalid setup:

- title-backed sound setup screen first
- then, if the user continues instead of exiting
- DDD splash
- warning splash
- title/main menu

So the earlier "normal launch goes straight into gameplay" reading was too aggressive.
The early state-`5` gate is real, but it is only an optional prelude, not the whole startup story.

## Why The Earlier Reading Broke

The mistaken interpretation came from looking at the early conditional call:

- `cmp esi, 1`
- `jnz 0x0187`
- `mov eax, 5`
- `call 0x3830`

That part is real.
What it does **not** mean is "no later frontend."

Once the raw disassembly is followed past the splash/resource load region, the same top-level routine later does:

- `mov eax, [0x2c6c3]`
- `call 0x6544`
- `mov eax, 1`
- `call 0x3830`

That later call is unconditional in the currently recovered startup path.

## Gameplay Handoff

This also sharpens the gameplay start model.

The `New Game` menu exit does **not** return to the caller and then ask some outer startup code to initialize gameplay later.
Instead:

- `0x3830`
  handles the frontend fade-out tail
- state `4`
  calls `0x2d88`
- then calls `0x05e0`
  `start_new_game_session`
- and only then returns to the outer session loop

So by the time startup leaves the frontend and enters the live session loop, the gameplay state is already seeded:

- board clear state
- HUD redraw
- first-piece selection
- gravity setup
- alert state reset
- live-game flag set

## Remaining Presentation Caution

One nuance still deserves mild caution:

- how prominent the `0x64f0` blackout stage feels to the player in each caller context

Its direction is now resolved:

- `0x64f0` is the simple palette fade-out helper

What remains slightly open is whether that blackout is visually prominent on every startup path or partly redundant on paths that are already near black before `0x3830` begins.

That does not affect the startup-order correction above.

## Bottom Line

The corrected startup model is now:

- frontend resources are preloaded very early
- sound setup is an optional early frontend path
- splashes still occur afterward
- normal cold startup still enters the main menu before gameplay
- gameplay begins only after the frontend `New Game` path has already called `0x05e0`
