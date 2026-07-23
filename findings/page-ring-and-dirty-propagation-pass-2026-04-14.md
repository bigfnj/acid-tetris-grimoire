# Page Ring And Dirty Propagation Pass

Date: 2026-04-14

## Summary

This pass tightens four closely related engine/render questions:

- the startup ownership of the three VGA page pointers
- the exact rotation semantics of `0x24d0`
- why dirty cells are so often marked with value `3`
- when the executable chooses full-page seeding versus incremental flushes

The biggest result is a much cleaner renderer model:

- startup seeds a real three-page ring
- `0x24d0` rotates that ring in a stable order
- dirty value `3` very likely exists to propagate one changed cell across all three pages over three presented frames
- major transitions call `0x2938` three times because they want immediate full-ring synchronization, not because `0x2938` is the normal frame-present path

That is exactly the kind of engine-side detail that will matter when we build the Windows port.

## 1. Startup Seeds A Real Three-Page VGA Ring

The startup entry at `0x00000018` now gives us the initial page assignments directly:

- `0x2c6bf = 0x0a0000`
- `0x2c6bb = 0x0a4b00`
- `0x2c6ab = 0x0a9600`

That matters because it lets us stop speaking loosely about "some page pointers."

The current best role model is:

- `0x2c6bf`
  currently displayed/front page
- `0x2c6bb`
  next back page
- `0x2c6ab`
  current back page / current `0x17719` flush target

This lines up well with later behavior:

- `0x17719` always flushes into the page rooted at `0x2c6ab`
- `0x24d0` later promotes old `0x2c6ab` into the displayed slot at `0x2c6bf`

So the game is not just double-buffered with a spare page.
It is running a proper three-page ring.

## 2. `0x24d0` Rotates The Ring In A Stable Direction

`0x24d0` rotates the page roots like this:

before:

- `0x2c6ab`
  current back
- `0x2c6bb`
  next back
- `0x2c6bf`
  displayed

after one `0x24d0(0)`:

- new `0x2c6ab` = old `0x2c6bb`
- new `0x2c6bb` = old `0x2c6bf`
- new `0x2c6bf` = old `0x2c6ab`

and the CRTC is programmed from the new `0x2c6bf`.

So the visible result is:

- whatever `0x17719` most recently flushed into old `0x2c6ab` becomes the displayed page

while the engine now continues drawing future incremental updates into old `0x2c6bb`.

That gives us a precise ring model:

- flush into current back page
- flip
- former back page becomes displayed
- next back page becomes current back

This is cleaner and more deterministic than the earlier softer wording.

## 3. Dirty Value `3` Very Likely Exists To Propagate Across The Three-Page Ring

This is the strongest new explanatory finding of the pass.

We already knew:

- `0x17719` decrements each non-zero dirty byte by `1`
- many helpers mark dirty cells with value `3`

What now makes sense structurally is **why** that value is `3`.

Because the game is using a three-page ring:

1. frame `N`
   `0x17719` copies the changed cell into current back page `A`, then decrements dirty from `3 -> 2`
2. `0x24d0`
   page `A` becomes displayed, current back becomes page `B`
3. frame `N+1`
   dirty is still non-zero, so `0x17719` copies the same cell into page `B`, then decrements `2 -> 1`
4. `0x24d0`
   page `B` becomes displayed, current back becomes page `C`
5. frame `N+2`
   dirty is still non-zero, so `0x17719` copies the same cell into page `C`, then decrements `1 -> 0`

At that point, all three pages have received the update.

So the best current reading is:

- dirty value `3` is the natural propagation count for a three-page ring

That explains a lot elegantly:

- why dirty bytes are not just booleans
- why `0x17719` decrements instead of clearing immediately
- why localized edits can remain coherent across later flips without forcing an immediate whole-ring reseed

This is still an inference from multiple strong facts rather than a single explicit constant-name in the binary, so I would word it as:

- **high-confidence structural reading**

But it is a very good one.

## 4. Full-Page Seeding And Incremental Flushes Serve Different Purposes

With the page-ring model in hand, the executable's two screen-update strategies now separate much more cleanly.

### Incremental Path

Ordinary gameplay and frontend steady-state presentation use:

- dirty markers
- `0x17719`
- `0x24d0`

That is the normal incremental frame path.

### Full-Seed Path

Major scene boundaries use repeated `0x2938` calls across all three page roots:

- frontend entry in `0x3830`
- frontend exit in `0x3830`
- early `New Game` bootstrap in `0x05e0`
- gameplay-base load in the startup path

The best current reading is:

- those three `0x2938` calls exist to synchronize the whole page ring immediately with a new scene base

So the executable uses full-page seeding when it wants:

- immediate ring-wide agreement

and incremental dirty flushing when it wants:

- ordinary per-frame presentation updates

This also explains why the splash presenter `0x2998` is different.

`0x2998` uploads only to `0x2c6bf`, the displayed page, because:

- it is presenting a self-contained fullscreen splash directly
- it does not continue into the shared `0x17719 -> 0x24d0` frame rhythm for that same visual

That is a very healthy architectural distinction.

## 5. What This Means For `State 9`

This pass did not overturn the current state-`9` trigger reading.
The best current model remains:

- executable-side evidence still points to the release-driven handoff seam
- when `0x184db == -2`, that seam restores the `GAME OVER` underlay and enters frontend state `9`

What this pass **does** strengthen is the render-side part of state `9`:

- once state `9` enters `0x3830`, the title/menu base is seeded into all three pages before the shared frontend fade-in and later high-score reveal proceed

So the state-`9` path is not only a shared frontend bootstrap in principle.
It is a **full page-ring reseed** in practice.

That makes the first visible high-score transition behavior more coherent than before.

## Porting Impact

For the future C++23/SDL3 port, the safest faithful guidance is:

- preserve the distinction between:
  - full-scene reseed
  - incremental frame flush/present
- if we keep a dirty-region renderer, carry changed cells long enough to synchronize every presentable back surface, not just the one current target
- if we choose full-frame redraw instead, we still need to preserve the scene-seeding semantics conceptually so transitions do not expose stale back-buffer state
- preserve the three-surface mental model even if the SDL implementation uses textures or render targets instead of literal VGA pages

This is one of those details that can easily disappear in a modern renderer while still leaving the game "playable."
But if we want the port to feel crisp and original, it is worth preserving.

## Bottom Line

This pass turns the renderer from "triple-buffered somehow" into something much more concrete:

- startup seeds a real three-page ring
- `0x24d0` rotates that ring predictably
- dirty value `3` likely exists to propagate localized edits across all three pages
- full-page `0x2938` seeding is a scene-bootstrap tool, not the ordinary present path

That is a strong engine-structure win for the project.

## 5 Next Strongest Moves

1. Tighten the state-`9` trigger one more step from the raw outer-loop branch so we can either confirm or deliberately downgrade the current release-driven reading.
2. Resolve whether any helpers ever mark dirty cells with values other than `3` in ways that materially change the propagation model.
3. Tighten the startup/runtime ownership of `0x2c6ab`, `0x2c6bb`, and `0x2c6bf` around video-mode init so we can say whether the initial displayed page is intentionally `0x0a0000` or just the hardware default.
4. Cross-check the splash presenter and any other direct-to-`0x2c6bf` paths to map exactly which visuals bypass the three-page incremental model.
5. Keep turning these page-ring and dirty-propagation rules into implementation constraints for the future SDL renderer.
