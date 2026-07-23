# Restart-Scene Capture Normalization Pass

Date: 2026-04-21

## Summary

This pass did not discover new executable behavior.
It normalized the owned gameplay captures against the recovered gameplay base so the remaining restart-scene visual questions can be discussed in gameplay-space coordinates instead of raw desktop pixels.

Main result:

- we now have a durable aligned evidence set for `06-newgame`, the early gameplay video stills, and the steady gameplay screenshot
- the screenshot captures align cleanly
- the video-frame captures align much more weakly and remain too noisy for frame-tight restart certification

So this branch advanced from:

- ad hoc desktop still references

to:

- normalized gameplay-space evidence

But it still does **not** fully close the finer restart reveal as a capture-certified sequence.

## New Artifacts

- [gameplay-alignment-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/gameplay-alignment-2026-04-21/manifest.json)
- [gameplay-delta-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/gameplay-delta-2026-04-21/manifest.json)

These artifacts normalize:

- [06-newgame.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/06-newgame.png)
- [gameplay-01.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-01.png)
- [gameplay-02.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-02.png)
- [gameplay-03.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-03.png)
- [gameplay-t15.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t15.png)
- [gameplay-t45.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t45.png)
- [gameplay-t90.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t90.png)
- [07-gameplay.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/07-gameplay.png)

against:

- [atet-dat.chunk-0001.off-00003391.len-0000AD86.gameplay-screen-320x240.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/screens/atet-dat.chunk-0001.off-00003391.len-0000AD86.gameplay-screen-320x240.png)

## Findings

### 1. The Screenshot Pair Is Still The High-Trust Gameplay Reference

The two desktop screenshots align cleanly to the recovered gameplay base:

- `06-newgame.png`
  - luma fit: `9.7575`
  - aligned-vs-base RGB diff: `13.074757`
- `07-gameplay.png`
  - luma fit: `10.1475`
  - aligned-vs-base RGB diff: `14.349918`

That makes them the best current preserved gameplay-side visual anchors.

### 2. The Early Gameplay Video Stills Remain Low-Certification

All six gameplay-video stills align much worse:

- luma fits around `30.5` to `32.5`
- aligned-vs-base RGB diffs around `31.4` to `33.4`

Those are not catastrophic if the goal is rough contextual normalization.
But they are much too weak to treat as frame-tight certification evidence for the exact visible restart reveal.

### 3. The Ordered Delta Sequence Preserves Some Timing Shape, But Not A Clean Visual Contract

The normalized previous-frame delta counts are:

- `06-newgame -> gameplay-01`: `46427`
- `gameplay-01 -> gameplay-02`: `5039`
- `gameplay-02 -> gameplay-03`: `588`
- `gameplay-03 -> gameplay-t15`: `9569`

That does preserve a real ordered sequence signal.
But every gameplay-video frame still departs from the authored gameplay base across nearly the whole screen bounding box.

So the branch now has:

- ordered normalized evidence

but still lacks:

- capture cleanliness strong enough to certify the exact user-visible restart reveal frame by frame

## Closure

This branch now closes one smaller but important gap:

- the restart-scene evidence is preserved in a normalized gameplay-space form

What remains open is narrower:

- not "we have no useful restart capture"
- but "the owned gameplay-video stills are too noisy to certify the fine visible reveal sequence beyond the executable-led model"

## Port Implication

For subsystem porting:

- keep using the executable-led restart contract from [26d000-state4-pre-live-piece-visibility-closure-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/26d000-state4-pre-live-piece-visibility-closure-pass-2026-04-21.md)
- treat the normalized screenshot pair as the stronger visual reference
- treat the gameplay-video stills as low-certification support material, not frame-exact proof

## Next Ordered Step

If this branch is revisited later, the useful new input is no longer "another interpretation pass."
It is:

- cleaner gameplay-frame extraction
- or a genuinely better-preserved new-game reveal clip
