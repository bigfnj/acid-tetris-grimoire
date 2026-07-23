# `0x26D000` State-4 Pre-Live-Piece Visibility Closure Pass

Date: 2026-04-21

## Summary

This pass closes the last visible-timing ambiguity on the state-`4` `New Game` path.

Main result:

- the fully staged state-`4` bootstrap image without a live piece is real
- but it lives only in the working screen and dirty map
- it is **not** a separately presented visible gameplay frame
- the visible sequence is now best modeled as:
  - early clear-only gameplay upload
  - palette-only reveal of that partial image
  - first gameplay-owned presented frame, already carrying the spawned live piece

That means the old wording:

- "we still need a frame-perfect capture of the completed bootstrap before the first live-piece draw"

can now be tightened to:

- there is no owned evidence for a separately visible frame in that interval because the interval is best modeled as offscreen-only

So this subsystem is now specific enough to treat as implementation-safe.

## New Owned Artifact

- [26d000-state4-pre-live-piece-visibility-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-pre-live-piece-visibility-closure.json)

This artifact records:

- the three-phase visible/offscreen model for state `4`
- why the completed bootstrap/no-piece image is not a separately presented frame
- why the first gameplay-owned present after state `4` still reaches the ordinary live-piece draw
- the resulting port-facing closure rule

## Key Artifacts Reused

- [26d000-state4-bootstrap-layout-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-layout-pass-2026-04-20.md)
- [26d000-state4-first-gameplay-flush-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-first-gameplay-flush-pass-2026-04-20.md)
- [26d000-state4-bootstrap-contributor-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-bootstrap-contributor-pass-2026-04-20.md)
- [startup-to-first-gameplay-present-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-to-first-gameplay-present-pass-2026-04-14.md)
- [new-game-visible-bootstrap-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/new-game-visible-bootstrap-pass-2026-04-14.md)
- [simple-palette-direction-correction-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/simple-palette-direction-correction-pass-2026-04-14.md)
- [piece-and-board-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/piece-and-board-pass-2026-04-13.md)
- [first-spawn-step-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/first-spawn-step-pass-2026-04-14.md)
- [gameplay-edge-paths-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-edge-paths-pass-2026-04-14.md)
- [gameplay-resume-early-return-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/gameplay-resume-early-return-closure-pass-2026-04-21.md)
- [06-newgame.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/06-newgame.png)
- [gameplay-t15.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t15.png)

## Findings

### 1. `0x05E0` Finishes Two Different Phases, But Only The Earlier One Is Presented

The owned `0x05e0` export now gives the split directly.

Early visible phase:

- clear playfield interior through `0x6274`
- clear preview interior through `0x6274`
- upload the working screen into all three VGA pages through `0x2938 x3`

Later staged-but-unpresented phase inside the same function:

- zero and redraw piece statistics through `0x2c58`
- redraw `LINES`, `SCORE`, `LEVEL`, and `HIGH-SCORE` through `0x2c58`
- clear logical board state through `0x1330`
- reset repeat timers through `0x09c8(EAX = 1)`
- stage preview / current-piece state through `0x1348`
- reset alert state through `0x206c(EAX = 1)`
- set live-game state and gravity fields

The important part is what does **not** happen after those later writes:

- no later `0x2938`
- no `0x17719`
- no `0x24d0`

So the completed bootstrap image exists after the early upload, but that completed form is not presented by `0x05e0` itself.

### 2. The Shared State-`4` Tail After `0x05E0` Is Palette-Only, Not A New Visible Screen-Content Present

The owned dispatcher tail already closes the post-`0x05e0` order:

- `0x6498`
- `0x2574`
- return to the session loop

And the simple-palette pass already closes what that pair does:

- `0x6498` is a palette-only fade-in
- `0x2574` uploads palette state
- neither helper is a new screen-content redraw or dirty flush

So the player can visibly receive:

- the earlier partially reset gameplay image already uploaded through `0x2938`

but not:

- a newly presented completed-bootstrap/no-piece image after the later `0x05e0` writes

That is the key closure on this branch.

### 3. The First Post-State-`4` Gameplay Present Cannot Take The Resume-Only Early-Return Exceptions

The `Return to Game` closure added one important comparison point:

- resume can preserve `0x184df > 0` collapse-only first frames
- resume can preserve `0x184db >= 0` dissolve-only first frames

But state `4` runs `0x05e0`, and the owned `0x05e0` export writes:

- `0x184db = -1`
- `0x184df = 0`

before returning to the session loop.

So the first post-state-`4` `0x09c8` does **not** enter:

- `0x1d04` collapse-only early return
- or `0x1f8c` dissolve-only early return

Those are real resume exceptions, but they are not real fresh-`New Game` exceptions.

### 4. The First Gameplay-Owned Presented Frame On An Ordinary Fresh Run Already Includes The Live Piece Draw

The owned `0x09c8` slice closes the normal spawn band tightly:

- `0x1011`
  call `0x1348`
- `0x102d`
  call `0x12ac`
- `0x10a1`
  call `0x10ec`
  draw the live piece when `0x184df == 0`

The supporting helpers are already closed too:

- `0x1330` clears the logical `10x20` board
- `0x1348` resets spawn state to:
  - `X = 4`
  - `Y = 0`
  - `rotation = 0`
  - gravity accumulator = `0`
- `0x12ac` returns collision only for invalid placement or occupied-board overlap and explicitly allows negative-Y spawn space

Inference from those owned helpers:

- on an ordinary fresh run with the board just cleared by `0x1330`, the post-`0x1348` spawn check is the ordinary non-collision branch
- so the first normal post-state-`4` gameplay step falls through to `0x10a1`

That does **not** mean the first visible piece has already descended.
[first-spawn-step-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/first-spawn-step-pass-2026-04-14.md) already closed that one-step gravity does not reach the descent threshold.

So the first gameplay-owned presented frame is best modeled as:

- full staged bootstrap now flushed
- live piece newly visible at spawn coordinates
- but not yet gravity-dropped

### 5. The Owned Capture Family Matches The Two Visible Phases, Not A Third Visible Pre-Live-Piece Frame

[06-newgame.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/06-newgame.png) and [gameplay-t15.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t15.png) both show the same family:

- full left HUD counters present
- preview present
- right piece-stat panel present with one promoted-piece counter at `1`
- clean alert region
- one live piece already visible at the top of the well

That capture family matches the stronger executable-led model:

- by the time the completed bootstrap is visibly present, the first gameplay-owned frame has already joined it with the live piece draw

No owned capture currently contradicts that by showing:

- a fully completed bootstrap layout
- with no live piece

And after this pass, that absence is no longer best treated as a simple missing screenshot.
It matches the stronger call-order model that the interval is offscreen-only.

## Closure

The remaining ambiguity from the older layout pass is now best closed as:

- the completed state-`4` bootstrap/no-piece image is a **working-screen interval**
- not a separately presented gameplay frame

So the visible state-`4` sequence is now:

1. early visible clear-only upload
2. palette-only reveal of that partial image
3. first gameplay-owned present with:
   - staged HUD / preview / piece-stat / clean-alert bootstrap
   - plus the newly visible live piece

## Practical Porting Impact

For a faithful port:

1. preserve the early clear-only reveal as a real visible state
2. do **not** invent a synthetic visible pause where the full bootstrap is shown without the live piece
3. treat the completed-bootstrap/no-piece image as an internal staging interval that exists only before the first gameplay-owned flush
4. treat the first fully readable new-game frame as already carrying the spawned piece at its reset coordinates

That is a stronger preservation rule than leaving this seam open as a generic capture-timing caveat.

## Remaining Nuance

One nuance still stays lighter than the call-order closure:

- the exact user-perceived feel of the short palette reveal between the early clear-only upload and the first gameplay-owned flush is still capture-light

But that no longer blocks implementation-safe behavior because it does **not** change:

- what the visible phases are
- what belongs in each phase
- or whether a separate visible pre-live-piece completed-bootstrap frame should exist

## Bottom Line

The important closure is:

- there is no longer a meaningful visible-frame gap around the completed state-`4` bootstrap
- the "full bootstrap but no live piece yet" interval is best modeled as offscreen-only
- the state-`4` new-game seam is now implementation-safe
