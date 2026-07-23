# Startup To First Gameplay Present Pass

Date: 2026-04-14

## Summary

This pass tightens the exact handoff from the frontend `New Game` exit to the first gameplay-side presented frame.

The important result is:

- the first gameplay image is **not** just the raw menu-exit restore
- it is **not** just the immediate `0x05e0` new-run snapshot either
- the first true gameplay present already includes at least one live outer-loop step

That gives us a much sharper preservation target for the future port.

## What `0x3830` Actually Does On `New Game`

When the frontend dispatcher leaves through state `4`, the tail at `0x3938` runs in this order:

1. clear the input latch through `0x094c`
2. run the animated frontend fade-out through `0x62e0`
3. save setup state through `0x3604`
4. restore the saved gameplay snapshot at `0x2c69f` into the working screen `0x2c727`
5. mirror that working screen into all visible VGA pages through three calls to `0x2938`
6. because `ECX == 4`, call:
   - `0x2d88`
   - `0x05e0`
7. run the simple gameplay-palette handoff:
   - `0x6498(0x2c303)`
   - `0x2574(0x2c303)`
8. return to the outer gameplay/session path

So the dispatcher does more than "leave the menu."
It restores the saved gameplay-side screen first, then seeds a fresh run, then hands the caller back a gameplay-side palette state.

## What `0x05e0` Makes Visible Immediately

`0x05e0` does include three direct `0x2938` uploads to the visible pages, but their placement matters.

The early portion of the function:

- clears the main playfield interior through `0x6274`
- clears the next-piece preview area through `0x6274`
- uploads the working screen into all three display pages through `0x2938`

That means the player can already be looking at a gameplay-side screen before the function finishes.

But that is not the whole initialized run.

## What `0x05e0` Stages After The Early Upload

After those early screen uploads, `0x05e0` continues with more run-start work:

- clears the per-piece statistics area
- redraws multiple numeric HUD counters through `0x2c58`
- refills the random table through `0x32b0`
- clears the logical `10x20` board through `0x1330`
- resets gameplay repeat state through `0x09c8(EAX = 1)`
- promotes the next piece to current and chooses the new next piece through `0x1348`
- computes gravity
- updates the alert tile through `0x206c`
- sets the live-game flag

Those later helpers render into the working screen and dirty-cell map.
There is no second full-screen `0x2938` upload after that later work inside `0x05e0`.

So the fully initialized run state is not completely on screen yet when `0x05e0` returns.

## Why The First Gameplay Present Includes A Live Logic Step

This is the most useful fidelity finding of the pass.

The frontend exit transition `0x3ee4` ends each frame with:

- `0x17719`
- `0x24d0(0)`
- store returned value to `0x184cb`

And `0x24d0` in normal mode:

- swaps the visible page
- waits for the global tick at `0x2d2a3` to advance
- enforces the minimum delay in `0x2c607`
- returns the elapsed tick count, clipped to `6`

Because it explicitly waits for tick progress, the normal return path cannot hand back a negative value or a same-tick stale value.
In practice, that means the inherited `0x184cb` entering the first outer gameplay loop is at least one live step.

At the top of the gameplay loop:

- `ESI` starts at `1`
- the inner catch-up loop runs while `ESI <= 0x184cb`

So the first gameplay-side presented frame necessarily performs at least one iteration of:

1. `0x2e18`
   particle/object update
2. `0x09c8`
   live gameplay update
3. `0x206c`
   alert-tile update

before it reaches:

- `0x2f24`
- `0x17719`
- `0x24d0`

for the first gameplay-side present.

## Practical Visible Reading

The most faithful current model is:

1. frontend exit transition finishes
2. the saved gameplay-side snapshot is restored
3. `0x05e0` seeds the new run, with some changes uploaded immediately and later changes left dirty in the working screen
4. the first outer gameplay frame runs at least one real simulation step
5. `0x17719` flushes the dirty gameplay cells
6. the first gameplay-side present shows the initialized run plus one live step of gameplay-side state

That is more precise than either of the simpler guesses:

- "menu exits straight to a finished gameplay screen"
- "menu exits to gameplay before any live logic runs"

Neither of those is as accurate as the call order we now have.

## What Still Stays Slightly Open

One nuance still deserves a little caution:

- the exact visible feel of the short gameplay-palette reveal through `0x6498` then `0x2574`

Its direction is now clear:

- `0x6498` is the simple palette fade-in helper

But without frame-perfect runtime tracing we should still avoid overstating whether the player perceives it as a distinct reveal beat or as a subtle part of the broader menu-to-game transition.

That uncertainty does not change the first-gameplay-step conclusion above.

## Porting Impact

For the source port, the safest faithful model is:

- separate `New Game` into:
  - frontend exit
  - gameplay base restore
  - run-state initialization
  - first outer gameplay step batch
  - first gameplay present
- do **not** collapse that into a single instantaneous "switch scene and start game" abstraction

This is one of those places where preserving sequencing will help the port feel like the DOS original rather than just behave like it.

## Bottom Line

The first gameplay frame is now much better understood:

- `0x05e0` seeds the run during the dispatcher tail
- the first fully meaningful gameplay present happens in the outer gameplay loop
- that first present already includes at least one real gameplay update step
