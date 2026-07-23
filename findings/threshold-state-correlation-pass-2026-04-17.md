# Threshold-State Correlation Pass

Date: 2026-04-17

## Summary

This pass correlated the current late object-2 runtime landings against the statically resolved frontend text and pointfield frame family.

Main result:

- the current threshold lane is **not** landing in the early pointfield helper family
- it is consistently landing inside `0x178b0`, the frontend glyph draw helper used by the text renderer
- the current lane therefore reaches a frontend text-render phase, not an object-2 first-entry point

That sharpens the runtime interpretation significantly.

## New Owned Artifact

- [threshold-state-correlation.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/threshold-state-correlation.json)

This artifact records:

- representative runtime landings
- their offsets relative to `0x178b0`
- internal phase mapping inside the glyph helper
- caller-family implications from the already-owned frontend loop findings

## Key Artifacts Reused

- [raw-60cc-625f.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/frontend-menu-primitive-pass/raw-60cc-625f.asm)
- [raw-09c8-0ae0.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-09c8-0ae0.asm)
- [frontend-menu-present-order.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/frontend-menu-present-order.json)

## Findings

### 1. The Current Late Object-2 Lane Clusters Inside `0x178b0`

Representative late object-2 landings now map as:

- `0x178B1`
- `0x178BF`
- `0x178C1`
- `0x178CE`
- `0x178D8`
- `0x178E9`
- `0x1791F`

All of those offsets lie inside:

- `0x178b0`
  [draw_frontend_font_glyph_16x12_scaled_to_16](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json:1418)

They do **not** fall into:

- `0x175c5`
- `0x17613`
- `0x17719`

So the threshold lane is no longer best described as:

- "late somewhere in object 2"

It is more precise to say:

- "late inside the frontend glyph draw helper"

### 2. These Landings Are Inside The Non-Zero Reveal Body, Not The Clear-Only Path

The top of `0x178b0` is:

- `0x178b7`
  `test eax,eax`
- `0x178b9`
  `je 0x17935`

That is the zero-reveal fast exit.

Every current threshold landing is **after** that gate.

Examples:

- `0x178B1`
  still in entry/prologue, but already inside the active helper body
- `0x178E9`
  in the reveal-division setup
- `0x1791F`
  in the row zero-fill branch

So these landings correlate to active visible glyph drawing.
They are not the `reveal = 0` band-clear case from `0x60cc`.

### 3. The Internal `0x178b0` Phase Map Is Now Useful

The current landing family spans meaningful internal subphases:

- `0x178BB .. 0x178E8`
  non-zero reveal setup
- `0x178E9 .. 0x17900`
  per-row resample index calculation
- `0x17902 .. 0x1791D`
  visible glyph-row copy
- `0x1791F .. 0x17929`
  zero-fill branch for destination rows outside the 12-row source glyph

That means the threshold lane is not bouncing randomly across object 2.
It is converging inside one coherent glyph-render body.

### 4. `0x178b0` Tells Us We Are In Frontend Text Rendering

The caller-side evidence is already strong:

- `0x60cc` calls `0x178b0` for each visible glyph
- `0x60cc` is the shared frontend string renderer with reveal/pulse scaling

That renderer is used by:

- menu entry reveal `0x3df4`
- menu exit conceal `0x3ee4`
- steady selected-row pulse through `0x3fd8`
- high-score row pulse through `0x5868`
- name-entry and other row-local redraw paths that already route through the same text system

So the threshold lane has a concrete behavioral meaning now:

- it reaches frontend text rendering
- not gameplay tile work
- not alert-tile work
- not the chunk-7 pointfield helpers themselves

### 5. What Has Already Happened, And What May Still Be Ahead, Depends On Caller Family

This pass does **not** claim one single caller with full certainty.
But it does bound the frame position strongly.

For menu entry and exit transitions:

- `0x17613` clear pass is already earlier in the same frame
- `0x39c4 -> 0x175c5 -> 0x17719 -> 0x24d0` is later in the same frame

For steady menu-family loops:

- frame-start clear of previous object pixels is already earlier
- text/highlight redraw through `0x60cc` happens during the per-step logic phase
- later object redraw through `0x175c5` and later flush through `0x17719` are still ahead in the same frame tail

So the important bounded statement is:

- the current threshold lane reaches a frontend text phase that is **between** already-known pointfield phases

That is much more useful than calling it a generic late object-2 stop.

### 6. Why This Still Does Not Give Helper-Family Closure

Even though `0x175c5` or `0x17719` may still be ahead in some caller families, the current lane still does not give first-entry closure for the helper family.

Reasons:

- it is already downstream of object-2 entry by `+0x2EC` to `+0x35A`
- the clear helper `0x17613` is already behind the current instruction pointer in the strongest menu-family models
- the exact caller family is still not uniquely pinned to one frontend loop
- the runtime search goal remains "earlier phase boundary" rather than "more no-hits from inside `0x178b0`"

So this pass improves phase understanding, but not closure by itself.

## Practical Next Step

The best next move is now more specific:

1. target the object-1 or very-early object-2 phase boundary that leads into frontend text rendering before `0x178b0`
2. use that earlier boundary to distinguish entry transition vs steady loop vs row-pulse family if possible
3. only after that resume helper-family breakpoint closure attempts

## Bottom Line

The current threshold lane has a concrete identity now:

- it lands in active frontend glyph rendering inside `0x178b0`

That means we are no longer searching for "what is this late object-2 state?"
We now know it is a frontend text-render phase, and the next search should move earlier than that phase rather than repeating more probes inside it.
