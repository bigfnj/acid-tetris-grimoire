# State 9 Handoff And Render Boundary Pass

Date: 2026-04-14

## Summary

This pass tightens five neighboring fidelity questions together:

- how strong the current `state 9` trigger reading really is
- what the first visible `state 9` frontend frame must look like
- how `0x2938`, `0x17719`, and `0x24d0` differ in responsibility
- why transitions seed all three VGA pages before resuming incremental presentation
- what part of this is executable-proven versus only capture-supported

The biggest practical result is that the render/present boundary is now much more useful to the future port than a generic "copy and flip" model.

- `0x2938` is the immediate full-page seeding path
- `0x17719` is the incremental dirty-cell flush path
- `0x24d0` is the page-ring rotation, present, and pacing path

That in turn makes the state-`9` handoff clearer:

- current executable evidence still supports a release-driven handoff after finished game-over cleanup
- once that handoff happens, the first visible frontend frame must begin from the shared title/menu bootstrap rather than from a direct high-score table reveal

## 1. The Current `State 9` Trigger Reading Still Looks Release-Driven

The strongest executable-side evidence has not changed:

- the outer session loop compares the input-latch tail against inherited byte `0x01`
- that is the practical released-`Esc` check
- when that branch succeeds, the loop either enters frontend state `1` during live play or, if `0x184db == -2`, restores the `GAME OVER` underlay through `0x23ac` and enters frontend state `9`

What matters here is what we did **not** find:

- no separate confirmed direct gameplay-side call into `0x3830(9)`
- no separate confirmed direct call into the high-score handler from outside the dispatcher path
- no current capture artifact that proves an automatic no-input transition from finished game-over into high scores

So the best current reading remains:

- after the game-over dissolve finishes, the same release-driven handoff branch is reused
- when that branch sees finished game-over state, it enters `state 9` instead of `state 1`

That is a little unusual, but it is still the cleanest current executable reading.

It is important not to overstate this beyond the evidence, though.
The trigger model is still:

- executable-strong
- runtime-capture-light

So for now the most accurate wording is:

- **best current reading:** finished game-over reaches high-score qualification through the same release-driven handoff seam, specialized by `0x184db == -2`

## 2. The First Visible `State 9` Frame Must Begin With The Shared Frontend Bootstrap

The first visible frontend stage after the game-over handoff is now stronger than before.

Once `0x23ac` restores the `GAME OVER` underlay, `0x3830` performs its normal entry sequence:

1. save the current working gameplay screen into `0x2c69f`
2. fade out the gameplay palette through `0x64f0`
3. copy the preloaded title/menu base from `0x2c6af` into:
   - the linear working screen
   - all three VGA pages through three `0x2938` calls
4. run the chunk-7/object fade-in through `0x63b8`
5. only then dispatch the requested frontend state

This matters because the state-`9` path does **not** skip that entry bootstrap.

So the first visible state-`9` stage is best modeled as:

- title/logo base already copied into the visible page ring
- chunk-7 floating objects beginning their shared fade-in
- high-score-specific reveal only after that bootstrap is underway

That is a better fidelity model than either:

- "game over screen goes straight into the high-score table"
- "high-score table is the very first visible frontend frame"

The executable says the frontend bootstrap happens first.

## 3. `0x2938`, `0x17719`, And `0x24d0` Are Three Different Presentation Tools

This pass is really about not collapsing three distinct helpers into one fuzzy "render" idea.

### `0x2938` Full-Page Seeder

`0x2938` takes a linear `320x240` source and copies it directly into the caller-supplied planar VGA page.
Important negative facts:

- it does not consult the dirty-cell map
- it does not return a catch-up count
- it does not wait on runtime ticks

So `0x2938` is an **immediate full-page upload** helper, not a paced presentation boundary.

### `0x17719` Dirty-Cell Presenter

`0x17719` scans the `40x60` dirty-cell grid, builds a queue of changed `8x4` cells, decrements each cell's persistence counter, and flushes only those queued cells from the linear working screen into the VGA page rooted at `0x2c6ab`.

So `0x17719` is the **incremental changed-cell flush** helper.

### `0x24d0` Page-Ring Flip And Pacing

`0x24d0` rotates the three page-root pointers:

- `0x2c6ab`
- `0x2c6bb`
- `0x2c6bf`

then programs the new displayed page through the CRTC start address registers, waits for tick progress, enforces the minimum frame delay, and returns the elapsed tick count clipped to `6`.

So `0x24d0` is the **page-ring present and pacing** helper.

### Why This Separation Matters

Putting those three together gives a much cleaner picture:

- `0x2938`
  seed whole pages immediately
- `0x17719`
  incrementally update the current back page from the working screen
- `0x24d0`
  promote that page to the display and advance timing

That is not an incidental implementation detail.
It is a real transition/render policy.

## 4. Why Transitions Seed All Three Pages First

This is the strongest render-side payoff from the pass.

At major scene boundaries, the executable repeatedly calls `0x2938` three times:

- once with `0x2c6ab`
- once with `0x2c6bb`
- once with `0x2c6bf`

That happens in places like:

- frontend entry after copying the title/menu base
- frontend exit after restoring the saved gameplay snapshot
- early `0x05e0` new-game bootstrap

The best current reading is:

- full-page seeding is used to synchronize the entire page ring before later incremental `0x17719 -> 0x24d0` presentation resumes

That prevents stale page contents from being exposed on later flips.

So the porting lesson is not "copy the framebuffer three times because DOS did."
It is:

- when a new scene base is established, all future presentable pages must start from the same base image before incremental updates resume

That is why these transitions feel stable instead of flickery or partially stale.

## 5. What This Means For `New Game`, `Return to Game`, And `State 9`

The three paths now separate more cleanly.

### `New Game`

- restore saved gameplay snapshot to all three pages
- stage an early cleared-board/preview image to all three pages
- later HUD/piece/alert work stays dirty in the working screen
- first outer `0x17719 -> 0x24d0` frame completes the scene

### `Return to Game`

- restore saved gameplay snapshot to all three pages
- do not run `0x05e0`
- first outer gameplay frame incrementally updates only what changed since the saved snapshot

### `State 9`

- restore cleaned gameplay underlay only long enough for snapshot/fade-out logic
- seed all three pages with title/menu base
- begin the shared frontend object fade-in
- then enter the high-score reveal path

So `state 9` is not just a special-case score table.
It still rides on the shared scene-seeding and present pipeline.

## 6. Capture Reality Versus Executable Certainty

This is a good place to be explicit about confidence boundaries.

The capture set currently gives us:

- `08-gameover.png`
- `04-hiscores.png`

Those are valuable endpoint confirmations.
But they do **not** directly capture the handoff sequence itself.

So:

- the steady endpoint look of game over is capture-supported
- the steady endpoint look of high scores is capture-supported
- the exact handoff trigger and first visible `state 9` frame are still primarily executable-derived

That is acceptable, but it is worth keeping visible in the notes so we do not confuse strong executable inference with direct transition footage.

## Porting Impact

For the future C++23 port, the safest faithful rules here are:

- preserve the conceptual difference between full-scene seeding and incremental per-frame presentation
- treat `0x2938`-style all-page seeding as a scene-bootstrap operation
- treat `0x17719 -> 0x24d0` as the steady incremental presentation boundary
- do not jump directly from finished game-over overlay to high-score table
- preserve the shared frontend bootstrap in the `state 9` path
- keep the current `state 9` trigger wording cautious: executable-strong, capture-light

## Bottom Line

This pass closes a very useful renderer seam.

- the page seeding model is now cleaner
- the shared flush/present boundary is better separated from direct uploads
- the first visible `state 9` frontend stage is better understood
- the finished game-over trigger still needs a little humility, but it is not vague anymore

That is exactly the kind of detail that will help us build a crisp port instead of a close-looking approximation.

## 5 Next Strongest Moves

1. Tighten the raw outer-loop branch semantics one more step so we can either confirm or downgrade the current release-driven `state 9` trigger model with less ambiguity.
2. Correlate the high-score still and any recoverable late gameplay capture frames against the title-base/bootstrap model to strengthen the first-visible-`state 9` presentation story.
3. Resolve the remaining low-level details of `0x17719` caller behavior on the gameplay side, especially how dirty persistence values interact with repeated flushes across consecutive frames.
4. Tighten the page-ring ownership model at startup so we can document exactly which page pointer is treated as current back page, next back page, and displayed page at each major transition.
5. Keep turning these transition/render rules into direct implementation guidance in the preservation spec so the eventual SDL port preserves scene cadence as well as game logic.
