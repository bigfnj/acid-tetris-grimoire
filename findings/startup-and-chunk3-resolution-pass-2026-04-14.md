# Startup And Chunk 3 Resolution Pass

Date: 2026-04-14

## Summary

This pass closes two stubborn gaps in the early executable map:

- the exact identity and order of the startup splash screens
- the practical contents of mixed-format `chunk 3`

The biggest result is that `chunk 3` is no longer an unresolved auxiliary blob. It now reads cleanly as a gameplay-side resource pack:

- a raw `0x400` lookup table used as four-stage transient particle/object color ramps
- seven `8x8` piece tiles used by the gameplay piece renderer
- ten `5x5` decimal glyphs used by the HUD counter renderer

## Startup Screen Order Is Now Clear

The startup chain at `0x000002da` and `0x000002e6` calls the full-screen local-palette screen loader `0x2998` in this order:

1. `chunk 0`
2. `chunk 2`

Current owned renders confirm those screens are:

1. `DDD Dungeon Dweller Designs`
2. `WARNING: This game has been known to cause severe brain damage.`

That matches the capture set and explains the two-splash startup sequence directly from the shipped data and executable flow.

Important consequence:

- `chunk 2` is the warning splash screen
- it is not a table, palette utility, or hidden menu asset

## What Happens After The Two Splash Screens

The startup path then pivots into gameplay-side resource initialization before entering the frontend:

- loads and decompresses `chunk 6`
- loads the gameplay base screen from `chunk 1`
- copies the gameplay base screen into the page buffers
- loads and decompresses `chunk 3`
- loads and decompresses the `GAME OVER` overlay from `chunk 8`
- loads the selected music track through `0x6544`
- conditionally enters the frontend dispatcher through `0x3830`, with the first startup frontend state later resolved as state `5` `sound setup` when setup mode is requested or `SETUP.DAT` is missing/invalid

Practical reading:

- the title/logo screen from `chunk 4` belongs to the frontend presentation path
- it is not one of the two pre-frontend startup splashes
- `chunk 3` is loaded after the gameplay screen is established, which strongly fits a gameplay-auxiliary role rather than an intro/splash role

## `chunk 3` Raw Block: Particle And Transient Object Color Ramps

The raw `0x400` bytes loaded into `0x2c6a3` are now much better explained by the transient object renderer at `0x2f24`.

`0x2f24`:

- walks the active transient object list
- computes `stage = [object + 0x14] >> 8`
- adds that stage to the object effect base at `[object + 0x1c]`
- looks up one byte from `0x2c6a3 + stage_offset`
- passes that byte to `0x17821` as the plotted color value

The object spawners at `0x2f78` and `0x3034` store the caller-supplied effect byte as:

- `[object + 0x1c] = effect_id << 2`

That makes the raw lookup block a natural fit for:

- `256` effect IDs
- `4` color stages per effect

So the raw block is now best described as a `256 x 4` particle/object color-ramp table.

## `chunk 3` Decompressed Block: Piece Tiles Plus Decimal Digits

The decompressed block at `0x2c60b` is `698` bytes long. Executable-side consumers now explain that layout cleanly.

### First Region: Seven `8x8` Piece Tiles

`0x10ec` `draw_piece_tiles` uses:

- source pointer `0x2c60b + piece_id * 64`
- destination blits of `8x8`

That means the first `7 * 64 = 448` bytes are the gameplay piece-tile bank.

Current owned renders confirm this region exports cleanly as:

- seven `8x8` piece tiles
- one per tetromino type / style index used by the renderer

`0x11e0` then uses the same geometry as the paired erase helper.

### Second Region: Ten `5x5` Decimal Glyphs

`0x2c58` `draw_decimal_counter` uses:

- source pointer `0x2c60b + 0x1c0 + digit * 25`
- copy helper `0x17688`, which copies `5` rows of `5` bytes

This exactly matches:

- offset `0x1c0` after the seven `8x8` piece tiles
- `10 * 25 = 250` bytes remaining

So the tail of the decompressed block is the decimal HUD font:

- digits `0` through `9`
- each as a `5x5` glyph

That fully explains the decompressed size:

- `7 * 64 = 448`
- `10 * 25 = 250`
- total = `698`

No unexplained decompressed tail remains after applying this layout.

## Practical Chunk 3 Layout

Current best layout:

- raw block at `chunk3 + 0x000`
  - `0x400` bytes
  - `256 x 4` particle/object color-ramp table
- compressed header at `chunk3 + 0x400`
  - decompressed size
  - compressed size
- decompressed payload at runtime to `0x2c60b`
  - `0x000 .. 0x1bf`
    seven `8x8` piece tiles
  - `0x1c0 .. 0x2b9`
    ten `5x5` decimal digit glyphs

## Extraction Impact

The current-effort graphics extractor should now treat `chunk 3` as resolved enough to export deliberately, not heuristically.

Useful owned outputs are now expected to include:

- raw lookup table
- grouped ramp JSON
- piece-tile renders and atlas
- decimal-digit renders and atlas
- raw decompressed auxiliary blob for preservation

## Porting Impact

This is a good quality-of-life improvement for the future Windows port:

- startup sequence can be reproduced exactly as `DDD -> warning splash -> frontend title/menu`
- piece-tile graphics no longer need to be inferred from screenshots
- decimal HUD glyphs no longer need to be reconstructed from captures
- transient particle/object effects now have a concrete byte-ramp source instead of a vague lookup-table guess

## Bottom Line

This pass resolves the remaining early-startup confusion well enough to move on cleanly.

What was previously called "mixed-format chunk 3" is now a practical gameplay resource pack, and the startup splash sequence is now grounded in both the shipped data and the executable path:

- `chunk 0` -> DDD splash
- `chunk 2` -> warning splash
- `chunk 4` -> frontend title/logo screen

That is a much better base for the eventual faithful source port.
