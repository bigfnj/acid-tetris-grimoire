# `0x26D000` State-4 Surface vs Flat-`0868` Pass

Date: 2026-04-20

## Summary

This pass compared the unique state-`4` new-game presentation surfaces against the dominant late flat-`0868` runtime lane.

Main result:

- the late flat-`0868` lane does **not** line up with the unique state-`4` early all-pages upload
- state `4` has two distinct presentation phases:
  - an immediate visible full-page gameplay upload through `0x2938`
  - later deferred gameplay redraws that wait for the next gameplay-side `0x17719 -> 0x24d0` present
- the late flat-`0868` lane belongs only to the second broad category of work:
  - deferred incremental redraw before a later present
  - but specifically in the **frontend text** family through `0x60cc -> 0x178b0`

So the dominant late lane is not a direct proxy for the state-`4`-only upload. If it is related at all, it is only through the shared dirty-flush / present model, not through the state-`4`-unique visible surface itself.

## New Owned Artifact

- [26d000-state4-surface-vs-flat-0868.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/26d000-state4-surface-vs-flat-0868.json)

This artifact records:

- the exact immediate and deferred presentation phases inside state `4`
- the owned late flat-`0868` lane identity
- the shared and non-shared presentation boundaries between them

## Key Artifacts Reused

- [26d000-state4-fork-first-frame-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-fork-first-frame-pass-2026-04-20.md)
- [startup-to-first-gameplay-present-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-to-first-gameplay-present-pass-2026-04-14.md)
- [threshold-state-correlation-pass-2026-04-17.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/threshold-state-correlation-pass-2026-04-17.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-entry-pass/ATET.EXE.flat-relocated.bin.000005e0.FUN_000005e0.c)
- [ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.00002c58.FUN_00002c58.c)
- [ATET.EXE.flat-relocated.bin.00006274.FUN_00006274.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-helper-pass/ATET.EXE.flat-relocated.bin.00006274.FUN_00006274.c)

## Findings

### 1. State `4` Has One Immediate Visible Surface And One Deferred Surface

The state-`4` fork remains:

- shared restore tail through `0x3830`
- `0x2d88`
- `0x05e0`
- shared gameplay palette reveal
- return to the session loop

Inside `0x05e0`, the presentation work is split cleanly.

Immediate visible surface:

- clear the playfield rectangle through `0x6274`
- clear the preview rectangle through `0x6274`
- upload the working screen into:
  - `0x2c6ab`
  - `0x2c6bb`
  - `0x2c6bf`
  through three calls to `0x2938`

That is an immediate all-pages gameplay upload.
It does **not** wait for `0x17719`.

Deferred surface after that upload:

- redraw gameplay counters through `0x2c58`
- refill the random table through `0x32b0`
- clear the logical board through `0x1330`
- reset repeat state through `0x09c8(EAX = 1)`
- stage preview / current piece state through `0x1348`
- reset the alert tile through `0x206c(EAX = 1)`

Those later writes stay in the working screen and dirty map until the next gameplay-side presented frame.

So state `4` is not one single presentation event.
It is:

- an immediate `0x2938` page-seeding phase
- followed by deferred gameplay-side staging

### 2. The Late Flat-`0868` Lane Is A Frontend Deferred Redraw Phase

The owned threshold correlation already closed the late lane:

- it lands inside `0x178b0`
- `0x178b0` is the frontend glyph draw helper
- that helper is reached through `0x60cc`

And `0x60cc` behaves like a deferred frontend producer:

- it draws visible glyphs into the working screen
- after each glyph it marks dirty rectangles through `0x2d30`
- it does **not** present directly
- later callers still rely on `0x17719 -> 0x24d0`

So the flat late lane belongs to:

- frontend text rendering
- dirty-map staging
- later back-page flush
- later page flip / throttle

It is not an all-pages gameplay upload.

### 3. The Unique State-`4` Upload And The Flat `0868` Lane Use Different Presentation Models

The contrast is now concrete:

- state-`4` immediate surface
  - gameplay-side
  - full-page seed
  - all three VGA pages
  - direct `0x2938` upload
  - visible before later run-start staging completes
- flat late `0868` lane
  - frontend-side
  - glyph-local text draw
  - working-screen only
  - dirty rectangles only
  - visible only after a later `0x17719 -> 0x24d0` frame tail

That means the strongest unique state-`4` discriminator is **structurally absent** from the late flat-`0868` lane.

The late lane never needed to explain:

- the `0x2938` all-pages upload
- the board / preview clear visible immediately through that upload

So the state-`4`-only upload is no longer the best direct explanation target for the dominant late lane.

### 4. The Only Meaningful Overlap Is The Broader Deferred-Redraw Model

There is still one important overlap, but it is weaker and more general.

After the early state-`4` upload, the rest of `0x05e0` stages gameplay changes that wait for a later present.
That broad structure matches the late frontend lane only at this level:

- producer writes into the working screen
- producer marks dirty regions
- later `0x17719` flushes only the back page
- later `0x24d0` promotes that page

But even there the producer families differ sharply:

- state-`4` deferred producers:
  - `0x2c58`
  - `0x1330`
  - `0x1348`
  - `0x206c`
  - then the first gameplay loop adds `0x2e18`, `0x09c8`, and `0x2f24`
- late flat-`0868` deferred producers:
  - `0x60cc`
  - `0x178b0`
  - later menu/object redraw family

So the overlap is:

- shared flush / present infrastructure

not:

- shared producer identity
- shared scene
- shared visible asset family

### 5. The Strongest Remaining Comparison Surface Is Now The First Gameplay-Side Flush After `0x05e0`

This pass narrows the next static target.

Because the late flat-`0868` lane is not the state-`4` immediate upload, the better comparison is now:

- the first gameplay-side deferred flush after `0x05e0`

That is the point where state `4` stops being uniquely "upload now" and starts looking more like every other dirty-map producer family:

- staged working-screen writes
- later localized flush
- later page promotion

So the next strong question is no longer:

- "does the late flat lane match the state-`4` upload?"

It is:

- "which staged `0x05e0` and first-loop gameplay writes survive into the first gameplay-side `0x17719` flush, and how is that flush sequenced against the state-`4` fork?"

## Practical Porting Impact

This keeps the port model honest:

- state `4` is not just "new game"
- it has an immediate visible gameplay upload before later run-start staging completes
- later redraw completion still follows the shared dirty-flush / present model

A faithful port should preserve both phases rather than collapsing them into one atomic restart screen.

## Next Strongest Move

Do a focused static pass on the first gameplay-side localized flush after the state-`4` fork:

1. recover which `0x05e0` producers are still only staged when control returns to the session loop
2. recover the first gameplay-loop writes that join them before the next `0x17719`
3. isolate the exact first gameplay-side flush surface that follows the unique state-`4` upload

## Bottom Line

The important closure is:

- the dominant late flat-`0868` lane is **not** the unique state-`4` all-pages upload surface

The only real overlap is the broader deferred redraw model behind the shared `0x17719 -> 0x24d0` present boundary.
