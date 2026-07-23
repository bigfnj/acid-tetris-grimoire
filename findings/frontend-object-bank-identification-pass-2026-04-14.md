# Frontend Object Bank Identification Pass

Date: 2026-04-14

## Summary

This pass used the title/options video frames to revisit the chunk-7 frontend animation interpretation.

The important result is that the captured floating smiley and tetrimino are already explained by the current chunk-7 render outputs.

So the right content-level description of chunk 7 is now:

- banked dotted frontend object silhouettes

not just:

- an abstract pointfield

The implementation-level model of projected points is still correct.
What changes here is the content identity of those projected points.

## The Capture Cue Was Real

The title/options video frames clearly show small floating menu objects near the text:

- [title-options-t60.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/title-options-t60.png)
- [title-options-t135.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/title-options-t135.png)

Those frames show exactly the kind of object the user called out:

- a smiley/face-like floating object
- a tetromino-like floating object

That was an important clue because it pushed the chunk-7 work away from overly generic language and back toward recognizable authored shapes.

## Current Chunk-7 Renders Already Match The Shapes

The existing current-effort render outputs contain the same object families:

- [bank00.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank00.frame000.title-overlay.png)
- [bank01.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank01.frame000.title-overlay.png)
- [bank02.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank02.frame000.title-overlay.png)
- [bank03.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank03.frame000.title-overlay.png)
- [bank04.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank04.frame000.title-overlay.png)
- [bank05.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank05.frame000.title-overlay.png)
- [bank06.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank06.frame000.title-overlay.png)
- [bank07.frame000.title-overlay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/bank07.frame000.title-overlay.png)

These renders show two clear bank families.

## Bank Families

### Banks `0..4`

These are tetromino-like dotted silhouettes.

High-confidence visible identities:

- bank `01`
  line / `I`-like silhouette
- bank `02`
  square / `O`-like silhouette
- bank `03`
  `T`-like silhouette

The remaining two compact banks are also clearly tetromino-family shapes:

- bank `00`
  asymmetric hooked silhouette
- bank `04`
  asymmetric stepped silhouette

I would avoid overclaiming exact piece letters for those last two without more direct corroboration, but they are plainly in the tetromino family.

### Banks `5..7`

These are smiley/face silhouettes rather than tetromino shapes.

They differ slightly in expression/outline, but all three belong to the same face family.

That matches the captured floating smiley on the menu screens.

## Important Negative Finding

The menu smiley is **not** evidence that chunk 6 is being reused on the frontend.

That distinction matters.

We already know:

- chunk `6`
  gameplay-side smiley/face assets for the `50x50` alert tile system
- chunk `7`
  frontend projected dotted object bank

So there are two separate smiley-related systems in the game:

1. chunk-6 gameplay alert faces
2. chunk-7 frontend floating face silhouettes

That is a useful correction for the preservation model.

## Why The Older Chunk-7 Language Was Incomplete

Calling chunk 7 a "pointfield" was not wrong at the rendering level.
The executable really does:

- project records to screen-space points
- draw only onto zero-valued pixels
- erase prior plotted points next frame

But at the authored-content level, those points are not random.
They form recognizable banked menu objects.

So the more accurate combined description is:

- projected dotted-object silhouettes

or:

- a banked frontend object cloud

That is a much better phrase for the future port docs than a bare "pointfield."

## Practical Porting Impact

This helps in two ways.

First, it sharpens the preservation goal:

- the menu frontend should preserve the floating tetromino/face object behavior, not just a generic field of dots

Second, it reduces later ambiguity:

- chunk 7 is a real authored menu-object asset system
- chunk 6 remains gameplay alert art

That should keep us from mixing those two systems during implementation.

## Bottom Line

The title/options video cue was useful and materially improved the frontend model.

We can now say with much more confidence:

- the floating smiley and tetrimino on the menu screens are explained by chunk 7
- chunk 7 banks resolve into tetromino-like and smiley-like dotted object silhouettes
- chunk 6 is a different smiley system used for gameplay alert tiles

That is a good cleanup of one of the more visually distinctive parts of the original frontend.
