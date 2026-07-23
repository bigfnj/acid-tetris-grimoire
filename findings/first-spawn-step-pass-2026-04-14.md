# First Spawn Step Pass

Date: 2026-04-14

## Summary

This pass tightens one subtle but very useful point in the `New Game` handoff:

- the first outer gameplay step is definitely live
- but on a fresh spawn it normally does **not** move the piece yet

That means the first gameplay-side present is "live" in the engine sense while still being visually dominated by the staged startup redraw.

## Why The First Step Does Not Naturally Drop The Spawned Piece

The relevant startup facts are now all pinned down:

- `0x05e0` computes gravity as `(current_level << 9) + 0x200`
- `0x1348` resets the spawned piece to:
  - `X = 4`
  - `Y = 0`
  - `rotation = 0`
  - `gravity accumulator = 0`
- the live gameplay loop at `0x09c8` compares the accumulator against a descent threshold of `0x10000`

So on the first live gameplay step after spawn:

- natural gravity contributes only `0x200 * (level + 1)`
- the accumulator starts at `0`
- one step is therefore far below the `0x10000` row-drop threshold

Even the held-Down override does not change that conclusion for a fresh spawn:

- held Down overrides the per-step increment to `0x8000`
- the accumulator still starts at `0`
- so one step still does not reach `0x10000`

So the first gameplay step after `0x1348` cannot cause a gravity descent by itself.

## What That Means For The First Gameplay Present

This sharpens the earlier "first present includes at least one live step" result.

The first outer gameplay frame still does run:

1. particle/object update
2. `0x09c8`
3. alert update
4. dirty flush
5. present

But on a fresh run, the `0x09c8` step is usually not contributing a visible gravity drop.

So the first presented gameplay image is best understood as:

- the staged startup HUD / piece / alert state finally being flushed
- plus one live logic step that normally keeps the spawned piece at its initial coordinates

That is more precise than simply saying "the first frame already advanced gameplay."

## Important Caveat

This is a fresh-spawn conclusion, not a blanket claim about all first visible frames.

Held movement or rotation inputs can still matter, because:

- `0x05e0` only resets the repeat timers through `0x09c8(EAX = 1)`
- it does not clear the live pressed-state table itself

So if the player is already holding a gameplay action when the outer loop begins, the first live step can still apply movement or rotation immediately.

What this pass closes is the gravity side specifically:

- fresh-spawn gravity and soft-drop do not move the piece on the first live step

## Porting Impact

For the future source port, the faithful model is:

- preserve the live fixed-step call ordering exactly
- but do not assume the first live step should visibly descend the new piece

If we recreate the original sequencing correctly, the first gameplay-side present after `New Game` should usually feel like:

- the scene finishes appearing
- then the piece begins its real fall cadence afterward

That is a small detail, but it is exactly the kind of small detail that makes a faithful port feel "right."

## Bottom Line

The first gameplay-side present after `New Game` is:

- live
- fixed-step correct
- but usually not yet gravity-advanced

So the first presented frame is primarily the completion of the staged new-run bootstrap, not the first visible row descent.
