# New Game Visible Bootstrap Pass

Date: 2026-04-14

## Summary

This pass closes the last important visual gap in the `New Game` handoff.

The useful result is:

- `New Game` is a real two-stage visible bootstrap
- the palette reveal through `0x6498` shows an intentionally incomplete gameplay screen
- the first outer gameplay flush through `0x17719` is what completes the scene

That makes the restart path more faithful and more interesting than a generic "switch to gameplay and draw everything" model.

## What The Visible Pages Actually Hold Before `0x05e0`

On the shared frontend exit tail inside `0x3830`, the game does this in order:

1. fade the frontend out through `0x62e0`
2. save setup through `0x3604`
3. restore the saved gameplay snapshot from `0x2c69f` into the working screen `0x2c727`
4. mirror that restored working screen into all three visible VGA pages through `0x2938`

So by the time `0x05e0` starts on the state-`4` path, the visible pages already contain:

- the cold-boot gameplay base snapshot on a startup `New Game`
- or the previously saved live gameplay screen on an in-game `New Game`

depending on how the frontend was entered.

## What `0x05e0` Makes Visible Immediately

The start-new-game initializer does not redraw the whole run state before showing anything.

Its earliest visible work is:

1. clear the playfield interior through `0x6274`
2. clear the next-piece preview area through `0x6274`
3. copy the working screen into all three visible VGA pages through three `0x2938` calls

That means the visible pages are updated early with:

- the restored gameplay snapshot
- but with the board interior and preview box blanked/reset

At that point, the newly visible gameplay image is only partially initialized.

## What Stays In The Working Screen Until The First Outer Flush

After the early `0x2938` uploads, `0x05e0` keeps doing important run-start work:

- zero the per-piece counters at `0x2c6f3`
- redraw decimal HUD counters through `0x2c58`
- clear the logical `10x20` board through `0x1330`
- reset gameplay repeat state through `0x09c8(EAX = 1)`
- promote and redraw current/next pieces through `0x1348`
- reset the alert-tile subsystem through `0x206c(EAX = 1)`

The important shared pattern is that these helpers write into the linear working screen and dirty map, but they do **not** present directly:

- `0x2c58` draws digits and only calls `0x2d30`
- `0x1348` uses `0x011e0`, `0x010ec`, and `0x2c58`, all of which only touch the working screen plus dirty map
- `0x206c` uses `0x2138` / `0x17983` and also only marks dirty cells

There is no later `0x2938`, `0x17719`, or `0x24d0` inside the tail of `0x05e0`.

So the fully initialized new-run scene is **not** on the visible pages yet when `0x05e0` returns.

## What The Player Actually Sees During `0x6498`

`0x6498` is now resolved as a simple palette fade-in:

- it repeatedly scales the gameplay palette through `0x284c`
- uploads it through `0x2574`
- does **not** redraw or flush screen content

So the palette reveal is applied to whatever was already sitting on the visible VGA pages after the early `0x05e0` upload.

That gives us the most faithful current model:

- the board interior and preview are already reset
- later HUD counters, piece displays, and alert tile are still only staged in the working screen

## Why Restarting From The In-Game Menu Is Especially Interesting

This is the strongest visible-behavior detail from the pass.

If `New Game` is chosen after entering the frontend from live gameplay, the snapshot buffer `0x2c69f` contains the previous run's live gameplay screen.

Because `0x05e0` uploads the cleared-board version of that snapshot **before** it redraws the counters and piece-related HUD, the fade-in can temporarily reveal:

- old score / lines / level digits
- old per-piece counters
- old surrounding HUD state

around:

- a newly cleared board interior
- a newly cleared preview box

Then the first outer gameplay frame flushes the staged new-run HUD and piece state on top.

At cold boot this effect is milder, because the saved snapshot is the gameplay-base screen loaded earlier from chunk `1`.

## Why The First Outer Gameplay Flush Matters Even More Now

`0x2d88` clears the dirty map before `0x05e0`, but the early clears and all of the later HUD/piece/alert work mark dirty cells again.

Because the early direct page uploads do not clear that map, the first outer gameplay pass still reaches `0x17719` with a full set of startup dirty cells.

So the first gameplay-side flush is doing two things at once:

- redundantly reconfirming the already uploaded cleared board/preview cells
- finally presenting the staged counters, first piece state, and alert tile state

That makes the first gameplay-side present the true completion point of the `New Game` bootstrap.

## Porting Impact

For the source port, the faithful model should be:

1. restore the saved gameplay snapshot
2. blank the board interior and preview and present that reset image
3. stage the rest of the new-run HUD / piece / alert state offscreen
4. run the gameplay palette reveal over the partially initialized visible image
5. let the first outer gameplay flush complete the scene

The important preservation detail is sequencing, not literal VGA mechanics.

If we later choose to "clean up" the stale-HUD restart artifact for a modernization mode, that should be a conscious opt-in change rather than an accidental side effect of a simplified renderer.

## Bottom Line

`New Game` is now best understood as a two-stage visual restart:

- an early visible reset of the restored gameplay snapshot
- then a first gameplay flush that finishes the real new-run presentation

That is a stronger and more faithful target for the future Windows port than either:

- immediate fully-drawn gameplay
- or a single opaque fade between menu and game
