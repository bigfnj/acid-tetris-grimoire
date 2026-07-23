# Alert ID Correlation Pass

Date: 2026-04-13

## Summary

This pass correlated the chunk-6 `50x50` alert-tile bank against:

- the executable-side alert ID logic
- the captured gameplay and game-over screenshots

The strongest result is that the alert IDs appear to map directly to chunk-6 tile indices.

That conclusion is supported by both code and capture evidence:

- `0x1793d` and `0x17983` index the alert art bank with `effect_id * 0x9c4`
- `0x9c4` is exactly one `50x50` tile
- the `game over` path triggers alert ID `6`
- the captured game-over face matches chunk-6 tile `06`
- a live gameplay alert crop matches chunk-6 tile `02`

So the simplest and strongest current reading is:

- alert effect ID `N` uses chunk-6 tile `N`

## Capture Artifacts Used

Generated correlation crops live in:

- [alert-crops](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/alert-crops)

Most useful files:

- [08-gameover-alert.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/alert-crops/08-gameover-alert.png)
- [gameplay-t45-alert.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/alert-crops/gameplay-t45-alert.png)
- [07-gameplay-alert.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/alert-crops/07-gameplay-alert.png)

The full source captures were:

- [08-gameover.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/screenshots/08-gameover.png)
- [gameplay-t45.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/captures/video/frames/gameplay-t45.png)

## Strong Mappings

### ID `6` -> chunk-6 tile `06`

Executable evidence:

- the spawn-failure / game-over path in `0x09c8` calls `0x2008` with `EAX = 6`

Capture evidence:

- the isolated game-over alert crop shows the worried open-mouth face
- that face matches chunk-6 tile `06`

Confidence:

- high

### ID `2` -> chunk-6 tile `02`

Executable evidence:

- the generic line-clear alert table in the gameplay loop contains `2, 0, 9, 7`
- the first entry is used for the generic one-line-clear case

Capture evidence:

- [gameplay-t45-alert.png](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/alert-crops/gameplay-t45-alert.png) shows the skeptical smirk face
- that face matches chunk-6 tile `02`

Interpretation:

- the captured frame likely represents a one-line-clear reward state

Confidence:

- medium_high

## Executable-Derived Effect Table

The alert-ID-to-tile mapping appears direct, so the remaining IDs can be interpreted from the executable paths even where capture confirmation is still limited.

Current best table:

- `0` -> chunk-6 tile `00` -> generic two-line clear
- `1` -> chunk-6 tile `01` -> fast follow-up one-line clear
- `2` -> chunk-6 tile `02` -> generic one-line clear
- `3` -> chunk-6 tile `03` -> low stack warning
- `4` -> chunk-6 tile `04` -> medium stack warning
- `5` -> chunk-6 tile `05` -> high stack warning
- `6` -> chunk-6 tile `06` -> game over / top-out
- `7` -> chunk-6 tile `07` -> generic four-line clear
- `8` -> chunk-6 tile `08` -> repeated four-line clear / back-to-back tetris reward
- `9` -> chunk-6 tile `09` -> generic three-line clear
- `10` -> chunk-6 tile `10` -> fast follow-up two- or three-line clear
- `11` -> chunk-6 tile `11` -> long no-clear timeout / sleepy state
- `12` -> chunk-6 tile `12` -> wake-up or recovery after clearing while tile `11` is active
- `13` -> chunk-6 tile `13` -> currently unused or still unresolved

## Why The Table Makes Sense Visually

The art progression lines up well with the known executable contexts:

- tiles `03`, `04`, `05` become progressively more distressed, which matches escalating stack-pressure warnings
- tile `06` is worried or shocked, which fits top-out/game-over
- tile `07` is a big surprised face, which fits a major event like a tetris
- tile `08` wears sunglasses, which fits a stronger "cool" or repeated-tetris reward
- tile `11` is sleepy, and the gameplay loop triggers ID `11` after a long idle period without a clear
- tile `12` is a calm satisfied smile, and the gameplay loop triggers it when a single-line clear happens while ID `11` is active

So the visual language of the extracted assets reinforces the code-side interpretation rather than fighting it.

## Limits

Not every ID has been directly confirmed in capture yet.

What is strong:

- ID-to-tile indexing is direct
- IDs `2` and `6` are capture-supported
- IDs `3`, `4`, `5`, `11`, and `12` have strong code-plus-art semantic support

What is still provisional:

- exact player-facing descriptions for IDs `0`, `1`, `7`, `8`, `9`, `10`, and `13`
- whether ID `13` is used in some mode or simply unused content

## Decompilation Impact

This is one of the best current asset-behavior correlations in the project.

It means the future port can already preserve:

- which gameplay events trigger the alert tile
- which exact face art belongs to each known event
- the alert-tile reveal / restore lifecycle

without waiting for every other asset format to be fully decoded.

## Recommended Next Move

The next best follow-up is to continue the same correlation workflow for the background/theme system:

- identify where chunk-7 panel selection happens in the executable
- determine whether level numbers map directly to panel indices or to a smaller reused set
- use captured gameplay states to confirm which panel or theme is active at each level
