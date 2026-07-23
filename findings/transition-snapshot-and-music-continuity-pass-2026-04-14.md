# Transition Snapshot And Music Continuity Pass

Date: 2026-04-14

## Summary

This pass tightens three closely related transition questions:

- what exactly gets snapshotted when gameplay enters the frontend
- what the first post-frontend gameplay frame is actually allowed to do
- how music continuity behaves across `New Game`, `Return to Game`, and post-game-over handoff

The biggest result is that the transition model is now cleaner than before:

- menu entry snapshots a restored gameplay image, not one with frame-local tracked particle overlays still painted on top
- the first post-frontend gameplay frame gets a deliberately reset timing baseline and effectively one fresh logic step, not a catch-up burst for however long the menu was open
- current music continuity is stronger than before: the track keeps playing across `New Game`, `Return to Game`, and the high-score handoff unless the player explicitly changes tracks or exits the game

## 1. Menu Entry Snapshots The Clean Gameplay Image, Not Frame-Local Tracked Overlays

The outer gameplay loop order on the `Esc` handoff is now very useful:

1. `0x17875`
   restore tracked transient pixels
2. `0x4a3`
   compare the input-latch tail against released `Esc`
3. if matched:
   - `0x94c`
   - optional `0x23ac` for finished game-over handoff
   - `0x3830`
4. only later, on ordinary non-menu frames:
   - `0x2f24`
   - `0x17719`
   - `0x24d0`

The important negative fact is:

- there is no `0x2f24` call between `0x17875` and `0x3830`

That means the dispatcher entry snapshots the working screen only after:

- the previous frame's tracked particle pixels have already been restored

and before:

- any new frame-local tracked particle pixels are drawn

So the saved gameplay snapshot at `0x2c69f` is best modeled as:

- the clean gameplay image for that moment
- not a one-frame particle-overlay composite

This is a strong fidelity detail for both:

- menu entry from live gameplay
- later `Return to Game`

because it means the saved image and the first resumed image are anchored to the same cleaned gameplay layer.

## 2. The First Post-Frontend Gameplay Frame Gets A Fresh Baseline And One New Step

The post-frontend return path in the outer gameplay loop is tighter than the earlier "at least one step" reading.

After `0x3830` returns, the loop does this:

1. `mov ecx, ebp`
2. `mov eax, ebp`
3. `mov [0x184cb], ebp`
4. `call 0x24d0`
5. jump back to the inner-step compare at `0x5a2 -> 0x4a3`

And `0x24d0` has a special `AL == 1` path:

- store current tick into `0x2ca93`
- return immediately

So this is not an ordinary present/catch-up call.
It is a timing-baseline reset.

That means menu time does **not** accumulate into a large post-menu catch-up burst.

In practice, the resumed control flow becomes:

- reset timing baseline
- re-enter the gameplay inner-step loop
- run the current step budget
- then reach `0x2f24`, `0x17719`, and `0x24d0(0)` for the first gameplay-side present

Because the `Esc` handoff is only meaningful on the first inner step, the practical step budget here is:

- exactly one fresh gameplay step

So the earlier broader wording can now be sharpened:

- the first gameplay present after leaving the frontend is not "some unknown positive catch-up"
- it is effectively one clean post-menu gameplay step

This holds for both:

- `New Game`
- `Return to Game`

The difference is not the step count.
The difference is what state that one step operates on.

### On `New Game`

That one step runs after:

- snapshot restore
- `0x2d88`
- `0x05e0`
- the staged new-run bootstrap

So the first gameplay present is:

- staged new-run screen state
- plus one fresh gameplay step

### On `Return to Game`

That one step runs after:

- snapshot restore only

So the first gameplay present is:

- restored live gameplay image
- optional transient tracked-pixel cleanup at frame start
- plus one fresh gameplay step on preserved live run state

This is a very healthy fidelity checkpoint for the port.

## 3. Music Continuity Is Stronger Than Before

A direct flat-binary call scan now gives a strong continuity result.

Direct `0x6544` call sites found:

- startup at `0x0474`
- main-menu music row at `0x42d8`

Direct `0x699e` / `0x69b0` call sites found outside `0x6544`:

- dispatcher state `3` hard exit path in `0x3830`

Direct `0x67aa` call sites found:

- inside `0x676a`
- inside full audio cleanup `0x67cf`

What that means behaviorally:

- startup chooses and starts the initial track
- the main-menu music row is the normal track-change UI
- hard exit stages a fade-out before DOS shutdown
- `New Game` does not load a new track
- `Return to Game` does not load a new track
- the game-over -> high-score handoff does not load a new track

So current music continuity is best modeled as:

- the active track persists across gameplay/menu transitions
- track changes are explicit user actions, not implicit scene changes

This now also has direct runtime confirmation:

- pressing `Esc` from live gameplay leaves the current track playing in the menu
- changing the `Music` row in the main menu switches tracks while the game is paused
- choosing `Return to Game` resumes gameplay under the newly selected track

This is more specific than "music keeps playing in menus."
It means the transition model itself is stable:

- `New Game` continues the current music track
- `Return to Game` continues the current music track
- state `9` high-score qualification / name entry continues the current music track
- only the `Music` main-menu row or cold startup selects a different track
- only `Exit Game` explicitly schedules a fade-out for termination

## 4. What The First Resumed Frame Can Actually Show

With these pieces combined, the first resumed gameplay frame is now easier to talk about accurately.

It can include, in this order:

1. restored clean gameplay snapshot already on screen
2. tracked-pixel cleanup if the restore queue was non-empty
3. one fresh gameplay step
4. one particle draw pass
5. one dirty flush
6. one normal gameplay present

So the first resumed frame is **not**:

- a huge catch-up burst
- a stale menu-time accumulation
- a snapshot that still includes tracked frame-local overlays

And it is also **not guaranteed** to show a dramatic visible change beyond the restored screen.

Depending on live state, the visible delta may be:

- only transient tracked-pixel cleanup
- cleanup plus one movement/rotation/alert update
- one live-step result without any visible cleanup if the restore queue was empty

That is a much tighter and more honest description than earlier.

## Porting Impact

For the future C++23 port, the safest faithful transition model is:

- save the gameplay snapshot only after frame-local tracked overlays have been restored
- do not let menu dwell time explode into a catch-up burst on resume
- give the first post-frontend gameplay frame one fresh fixed gameplay step
- preserve current music across `New Game`, `Return to Game`, and post-game-over high-score flow
- restrict automatic track changes to startup and explicit music-row actions

That should help the port feel stable and "native to the original" instead of overreactive or overly modernized during transitions.

## Bottom Line

This pass closes an important transition seam:

- the saved gameplay image is cleaner than a naive "whatever was on screen" interpretation
- the first resumed frame is more controlled than a naive "run catch-up for all menu time" interpretation
- the soundtrack is more continuous than a naive "scene changes may restart music" interpretation

Those are the kinds of details that will matter once we start implementing the port.

## 5 Next Strongest Moves

1. Tighten the exact first visible resumed-frame delta sources inside `0x2e18`, `0x206c`, and any HUD dirty-mark helpers so we know what most often changes on that one-step resume frame.
2. Resolve whether any non-tracked transient systems can still diverge between the saved gameplay snapshot and the first resumed frame.
3. Tighten post-game-over audio behavior beyond music continuity, especially whether any warning/game-over SFX tails survive into state `9`.
4. Resolve the first visible frame of the game-over -> high-score handoff with the same precision we now have for `New Game` and `Return to Game`.
5. Start a formal transition-preservation spec for snapshot timing, first-frame sequencing, and music continuity while we keep reverse-engineering the few remaining frame-boundary helpers.
