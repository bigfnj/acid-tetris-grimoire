# Frontend Dispatcher Transition Pass

Date: 2026-04-14

## Summary

This pass tightens `0x3830`, the frontend dispatcher, using the raw relocated disassembly together with the now-cleaner handler models for:

- `0x40c0` main menu
- `0x4584` options
- `0x49bc` keyboard setup
- `0x4f38` credits
- `0x50b0` high scores
- `0x5a94` sound setup

The important result is that the dispatcher is no longer just "call a handler from a jump table."
It is a real frontend state loop with:

- a frontend fade-in/bootstrap phase
- repeated state dispatch through `ECX`
- handler-returned next states
- two direct exit states that break back into gameplay
- a shared frontend fade-out / persistence / page-restore tail

## Shared Dispatcher Frame

Before dispatch begins, `0x3830`:

1. saves the current working screen into the gameplay snapshot buffer
2. runs the simple palette fade-out path through `0x64f0`
3. copies the title/menu base screen into the working and visible pages
4. runs the chunk-7 frontend fade-in through `0x63b8`

After that, the dispatcher enters a loop using:

- `ECX`
  current frontend state
- `ESI`
  break-out flag
- `EAX`
  next state returned by handlers

Core rule:

- handlers return their next frontend state in `EAX`
- the dispatcher copies that into `ECX`
- dispatch continues until `ESI == 1`

## Direct Exit States

Two states do not call a separate handler and instead break out of the frontend loop directly.

### State `2` Resume Gameplay

The dispatcher:

- stores `7` into `0x1886f`
- sets `ESI = 1`
- leaves `ECX = 2`

Practical meaning:

- state `2` is the "leave frontend and continue gameplay" state
- setting `0x1886f = 7` preserves `Return to Game` as the selected main-menu row for the next time the menu opens

### State `4` Start New Game

The dispatcher:

- sets `ESI = 1`
- leaves `ECX = 4`

Then, after the shared frontend fade-out tail, it performs:

- `0x2d88`
- `0x05e0`

So state `4` means:

- leave frontend
- clear the gameplay dirty/region map
- start a fresh run

This is an important architectural distinction from state `2`.

## Handler-Returned States

The currently resolved handlers fit the dispatcher cleanly:

- main menu `0x40c0`
  returns menu-side states such as `6`, `8`, `10`, `3`, `2`, or `4`
- options `0x4584`
  returns `7` or `1`
- keyboard setup `0x49bc`
  returns `6` on Back to Options and `1` on Esc
- sound setup `0x5a94`
  returns `2` for Play Game or Esc, and `3` for Exit to Dos
- high scores `0x50b0`
  returns `1` after its footer-exit flow
- credits `0x4f38`
  returns `1` on Esc

That means the dispatcher model for the source port can be written in a straightforward way:

- call the current state's handler
- accept its returned next state
- redispatch until the handler chain reaches state `2` or `4`

## State `3` Is A Hard Exit Path

The direct state-`3` branch does not behave like the other frontend states.

It:

- waits for the lower audio path to settle
- issues an audio-side command sequence through `0x699e`
- runs the frontend animation fade-out through `0x62e0`
- saves `SETUP.DAT`
- calls `0x05cc`

This is the executable-side termination path, not just another menu return.

## Shared Dispatcher Tail

Once `ESI == 1`, the dispatcher always runs the same tail:

1. clear the input latch through `0x94c`
2. run frontend fade-out through `0x62e0`
3. save setup state through `0x3604`
4. restore the saved gameplay snapshot back into the working and visible pages
5. if `ECX == 4`, call `0x2d88` then `0x05e0`
6. run simple gameplay-palette fade-in through `0x6498`, then finish with a direct full-palette upload through `0x2574`

So both gameplay-return and new-game-start leave the frontend through the same visual tail, then diverge only after the page restore.

## Porting Impact

This is a useful source-port milestone because it gives us the actual dispatcher contract:

- frontend handlers return next-state IDs
- only specific states break out to gameplay
- new-game and resume-game are intentionally separate exit states
- the visual tail after frontend exit is shared and should remain shared in a faithful port

## Recommended Next Move

The strongest next frontend-side target is to promote this into a formal transition table for the modern codebase, including:

- state ID
- handler function
- returned next states
- whether the state redispatches or exits to gameplay
- any side effects that happen in the dispatcher tail
