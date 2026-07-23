# Sound Setup Return And Resume Fidelity Pass

Date: 2026-04-14

## Summary

This pass tightens two neighboring seams:

- the optional early sound-setup detour on startup
- the real `Return to Game` resume path

The important result is that both use frontend state `2`, but they do **not** mean the same thing experientially.

- during early startup, state `2` returns to the pre-resource startup path
- during live gameplay, state `2` resumes the exact saved gameplay state without reinitialization

That distinction matters a lot for a faithful port.

## Early Sound Setup Is A Prelude, Not A Gameplay Launch

The startup entry at `0x00000018` does this before the optional `0x3830(5)` detour:

- allocate the major working buffers
- zero-fill them through `0x0770`
- preload the chunk-7 frontend object resources
- preload the chunk-4 title/logo base through `0x5f78`
- load or seed `SETUP.DAT`
- arm the audio service layer through `0x68d8`

Then, if explicit `setup` mode is requested or `SETUP.DAT` had to be initialized:

- `0x3830(5)` enters the sound-setup screen

So the sound-setup screen is shown on top of a real frontend presentation base:

- title/logo backdrop
- chunk-7 object fade
- six sound-setup rows

but it still happens **before**:

- concrete audio backend init through `0x668c`
- SFX-slot loading
- the two startup splashes
- gameplay-base resource loading

## What State `2` Means In Early Sound Setup

Inside `0x5a94`:

- `Play Game` stages return state `2`
- `Esc` also stages return state `2`

The dispatcher tail for state `2` then:

1. clears the release latch
2. fades the frontend out
3. saves setup
4. restores the previously saved screen snapshot from `0x2c69f`
5. runs the simple palette reveal path
6. returns to the caller

At this early startup point, that restored snapshot is **not** a live gameplay screen.

Why:

- `0x2c727` was only allocated and zero-filled before the sound-setup detour
- chunk `1` gameplay base has not been loaded yet
- the gameplay palette path associated with chunk `1` has not been populated yet either

So the best current reading is:

- leaving early sound setup returns to a blank or near-black startup screen
- then the startup entry continues into normal backend init, splash presentation, gameplay-base loading, music start, and later `0x3830(1)` main-menu entry

That is a very different meaning from gameplay resume.

## Startup Explicitly Re-Seeds Main-Menu Selection After The Detour

Right after `0x3830(5)` returns, the startup entry does:

- `mov [0x1886f], 0`

So even though dispatcher state `2` normally preserves row `7` as the next main-menu selection for a future resume context, the early startup caller immediately overwrites that and re-seeds the main menu to row `0`.

That confirms the intended behavior:

- early sound setup is a startup prelude
- not a persistent gameplay/menu session

## `Return to Game` Is A True Live-State Resume

The meaning of state `2` is much stronger when reached from the real main menu during live gameplay.

In that path, `0x3830`:

1. restores the saved gameplay snapshot from `0x2c69f`
2. mirrors it back to the visible VGA pages
3. does **not** call `0x2d88`
4. does **not** call `0x05e0`
5. runs the simple palette reveal
6. returns to the outer gameplay loop

That means `Return to Game` is not a partial rebootstrap.
It is a direct restoration of the previously saved playfield image and live run state.

## What Is Preserved On Resume

Because state `2` never calls `0x05e0`, it does **not** reset any of the gameplay-side transient run fields that `New Game` resets.

So resume preserves:

- current piece ID
- next piece ID
- piece X / Y / rotation
- gravity accumulator
- left/right/rotate repeat timers
- Down lockout state
- score / lines / level
- alert-tile state
- board contents

That is one of the biggest behavioral differences between:

- `New Game`
- `Return to Game`

`New Game` creates a staged visible restart.
`Return to Game` restores a coherent live snapshot and keeps going.

## What The First Resumed Gameplay Frame Looks Like

This now reads more cleanly than before.

Unlike `New Game`, the resume path has:

- no early partial board clear
- no HUD redraw bootstrap
- no `0x05e0`
- no dirty-map reset through `0x2d88`

So the first resumed gameplay present is **not** completing a staged scene.

It is best modeled as:

- fully restored saved gameplay image already visible
- then at least one real outer gameplay step
- then the next dirty flush / present for whatever that step actually changes

That is a stronger and cleaner resume model than the startup `New Game` path.

## Resume Input Semantics

The input side also splits cleanly.

### Resume Via `Return to Game`

Main-menu row `7` activation is:

- release-driven through `0x964`
- gated through `0x3fd8`

So resuming through `Enter` on `Return to Game` does **not** carry the Enter event into gameplay:

- the menu saw it on release
- the release latch is cleared again in the dispatcher tail

### Resume Via `Esc`

The main-menu fast path for `Esc` is different:

- it checks the live pressed-state table directly
- it only works when the live-game flag is set

But the gameplay-side menu-entry path for `Esc` is release-driven through the latch.

So on resume:

- held `Esc` can leave the frontend
- but it does **not** immediately bounce back into the menu
- because the dispatcher tail clears the release latch, and gameplay re-entry to the menu watches for released `Esc`, not held `Esc`

That is a subtle but very useful fidelity detail.

### Other Held Gameplay Keys

Because the dispatcher tail clears only the release latch and does not zero the live pressed-state table:

- held gameplay-bound keys can still carry into the first resumed gameplay step

And because resume does **not** reset gameplay repeat timers or gravity state:

- those carried inputs operate on the preserved live run state, not a fresh bootstrap state

That is different from `New Game`, where `0x05e0` resets the gameplay repeat fields before the first active piece is played.

## Porting Impact

For the future source port, the safest faithful split is:

- early sound setup:
  - treat as a startup-only frontend prelude
  - return to startup continuation, not gameplay
  - reseed main-menu selection to row `0` afterward
- return to game:
  - restore the saved gameplay snapshot directly
  - preserve all live gameplay transient state
  - allow physically held gameplay keys to continue affecting the resumed run
  - do not accidentally sanitize the gameplay state as if it were a `New Game`

If we later add a modernization mode that clears all held input or soft-resets repeat timers on resume, that should be explicit and optional.

## Bottom Line

This pass closes an important ambiguity:

- state `2` is a generic frontend exit code
- but its caller context matters

On startup, it means:

- leave sound setup and continue boot

During live play, it means:

- restore the exact saved run and resume it

That is a strong fidelity checkpoint for the eventual port.
