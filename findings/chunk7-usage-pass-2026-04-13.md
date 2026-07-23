# Chunk 7 Usage Pass

Date: 2026-04-13

## Summary

This pass corrected one of the biggest remaining format assumptions.

`chunk 7` is very likely **not** a direct bitmap panel set.

The executable-side startup path shows that:

- the full decompressed `chunk 7` payload is `0x38000` bytes
- it is divided into `8` slices of `0x7000` bytes each
- one slice is selected randomly at startup
- the active slice is then consumed as `1024` records of `0x1c` bytes

That makes the current best interpretation:

- `chunk 7` is a bank of structured frontend animation or decoration records
- not a set of literal `128x224` pixel images

This explains why the earlier heuristic PNG renders from chunk 7 looked like noise even though the dimensions seemed mathematically plausible.

## Startup Load Path

The startup routine at [FUN_00005fd8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.00005fd8.FUN_00005fd8.c) does the following:

1. loads chunk `5`
2. loads chunk `7`
3. allocates several working buffers:
   - `0x3000`
   - `0x7000`
   - `0x38000`
4. decompresses chunk `7` into the `0x38000` buffer at `0x2d22f`
5. calls `0x5ef4`

The key point is the `0x38000` destination size:

- `0x38000 = 229376`
- this had previously been treated as `1024 x 224`
- but the consumers do not treat it like a pixel framebuffer

## `0x5ef4` Randomly Selects One Of Eight Slice Banks

[FUN_00005ef4.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/loader-deeper-pass/ATET.EXE.flat-relocated.bin.00005ef4.FUN_00005ef4.c) shows:

- random value modulo `8`
- selected bank index stored at `0x2d233`
- active copy size `0x7000`
- source address:
  `chunk7_base + bank_index * 0x7000`
- destination buffer:
  `0x2d223`

So the decompressed chunk-7 payload is best modeled as:

- `8` structured banks
- each bank exactly `0x7000` bytes

## `0x7000` Is Treated As `1024` Records Of `0x1c` Bytes

The consumers [FUN_000062e0.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000062e0.FUN_000062e0.c) and [FUN_000063b8.c](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/first-pass/ATET.EXE.flat-relocated.bin.000063b8.FUN_000063b8.c) both:

- iterate the active bank buffer at `0x2d223`
- step by `0x1c` bytes each iteration
- stop at `0x7000`

That means:

- `0x7000 / 0x1c = 1024`

So one active bank contains:

- `1024` records
- `28` bytes each

That is structured object data, not image pixels.

## Frontend Consumers Use The Active Chunk-7 Bank

The active bank pointer at `0x2d223` is referenced by multiple menu-side handlers:

- main menu `0x40c0`
- options `0x4584`
- keyboard setup `0x49bc`
- sound setup `0x5a94`
- frontend fade or transition helpers `0x62e0` and `0x63b8`

That strongly suggests chunk `7` belongs to the frontend decorative system rather than the gameplay playfield itself.

This also lines up with another recent correction:

- the actual level-0 gameplay base screen is already provided directly by chunk `1`

So chunk `7` does not need to explain the visible gameplay backdrop in the captures.

## What The Consumer Functions Suggest

`0x62e0` and `0x63b8` do two things in parallel:

- iterate and update or render the `1024` active records
- apply brightness-scaled palette animation over time

That makes them look more like:

- animated frontend decoration loops
- particle or spritefield style effects
- menu-side motion systems with fade-in or fade-out behavior

not static bitmap blits.

## Practical Correction To Earlier Assumptions

Older working interpretation:

- chunk `7` was probably eight `128x224` background panels

Current better interpretation:

- chunk `7` is eight banks of `1024` structured frontend-animation records

The earlier `128x224` panel render outputs should now be treated as heuristic artifacts, not faithful decodes.

## Decompilation Impact

This is a helpful narrowing step.

It means:

- we should stop spending time trying to visually decode chunk `7` as raw gameplay art
- we should instead map its record format and the animation helpers that consume it
- gameplay backdrop reconstruction should continue to rely on chunk `1` until contrary evidence appears

## Recommended Next Move

The strongest next move is to characterize one chunk-7 record:

- map the fields inside a single `0x1c` record
- identify which fields are x/y, velocity, color, lifetime, and rendered glyph or sprite index
- connect `0x62e0`, `0x63b8`, `0x17613`, `0x175c5`, and `0x17719` into one coherent frontend-animation path
