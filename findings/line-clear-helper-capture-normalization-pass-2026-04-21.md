# Line-Clear Helper Capture Normalization Pass

Date: 2026-04-21

## Summary

This pass reused the new normalized gameplay-capture alignment artifacts to test whether the line-clear helper branch gets materially more specific once the owned stills are compared in gameplay-space coordinates.

Main result:

- the branch is better preserved
- it is still not helper-certifying

So the problem remains:

- capture separation

not:

- missing executable understanding

## Artifacts Used

- [gameplay-alignment-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/gameplay-alignment-2026-04-21/manifest.json)
- [gameplay-delta-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/gameplay-delta-2026-04-21/manifest.json)
- [line-clear-helper-capture-coverage-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-helper-capture-coverage-pass-2026-04-20.md)
- [line-clear-style-source-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-style-source-closure-pass-2026-04-20.md)

Most relevant normalized stills:

- [gameplay-t15.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t15.png)
- [gameplay-t45.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t45.png)
- [gameplay-t90.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t90.png)

## Findings

### 1. `gameplay-t15` And `gameplay-t45` Normalize Into Almost The Same Capture Family

After alignment:

- `gameplay-t15`
  - luma fit: `30.5275`
  - aligned-vs-base RGB diff: `31.439891`
- `gameplay-t45`
  - luma fit: `30.61`
  - aligned-vs-base RGB diff: `31.469944`

Those are effectively the same capture-quality tier.

Their direct previous-frame delta is also small:

- `gameplay-t15 -> gameplay-t45`: `569` changed pixels
- delta bbox: `x = 0..98`, `y = 79..185`

So the normalized branch still does **not** expose a large clean helper-specific difference between those two preserved stills.

### 2. The Base-Difference Region Counts Still Do Not Select One Of The Ten Helpers

Normalized base-delta counts:

- `gameplay-t15`
  - well interior: `1147`
  - alert tile: `1964`
  - outside well interior: `41212`
- `gameplay-t45`
  - well interior: `1148`
  - alert tile: `1992`
  - outside well interior: `41265`

Those counts are too close to support any honest helper-family claim.

They do confirm:

- the preserved stills are from a gameplay family with real transient activity

They do **not** support:

- naming one specific helper address out of the ten level-selected motion families

### 3. `gameplay-t90` Separates More Strongly, But Not In The Right Way

The `gameplay-t45 -> gameplay-t90` previous-frame delta is much larger:

- `8991` changed pixels

But that later change does not create a helper-certifying line-clear sequence.
It just shows that the wider gameplay presentation keeps evolving.

So the normalized branch still lacks:

- a preserved adjacent-frame event family tightly centered on one clear
- known level modulo context at that same moment

## Closure

This pass upgrades the branch from:

- raw reference stills

to:

- normalized aligned gameplay-space evidence

But the key answer stays the same:

- the owned captures are still not specific enough to assign a named helper family honestly

## Port Implication

For subsystem porting:

- preserve the ten executable-owned helper families as distinct implementation paths
- keep helper naming and capture-side visual matching provisional
- do not pretend that `gameplay-t45` now identifies one specific helper

## Next Ordered Step

If this branch is revisited later, the next useful input is:

- an adjacent-frame line-clear event sequence with visible level context

not:

- another interpretation-only pass over the current still set
