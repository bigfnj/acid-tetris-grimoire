# Startup Init Pass

Date: 2026-04-13

## Summary

This pass followed the startup path centered around `0x00000018` and the continuation Ghidra split out at `0x0000023b`. The main goal was to understand what the executable actually does with the early `ATET.DAT` chunks during initialization and setup.

The biggest practical outcome is that `chunk 2` is no longer an "unknown maybe-table" candidate. The executable loads it through the same full-screen palette-plus-RLE path used by the known screen chunks, so it has now been promoted into the verified graphics extractor.

## Important Structural Correction

`FUN_0000023b` is best treated as a continuation of the larger startup/init routine, not as a fully independent subsystem. Ghidra created a function boundary there, but the code flow and state usage show it is still part of the same initialization chain that starts at `0x00000018`.

## What The Startup Path Clarified

### Audio slot registration is the WAV path

The early chunk loads through `0x00003370` plus `0x000067f8` are loading the twelve short sound-effect chunks, not the six tracker-based music chunks.

The chunk IDs loaded into slots `0..11` are:

- `15`
- `16`
- `23`
- `24`
- `25`
- `26`
- `19`
- `20`
- `21`
- `22`
- `17`
- `18`

These correspond to the verified WAV region range in `ATET.DAT`.

Practical consequence:

- `0x000067f8` should now be treated as an SFX-slot loader first.
- The music chunks are likely managed elsewhere.

### Chunk 2 is a full-screen bundle

The startup code calls `0x00002998` twice:

- once with `chunk 0`
- once with `chunk 2`

`0x00002998` is the screen-bundle loader that:

- opens a chunk through `0x00003300`
- reads `768` palette bytes
- reads a compressed payload size
- reads the payload into scratch memory
- decodes it through the verified four-mode RLE routine
- displays it through the video pipeline

Practical consequence:

- `chunk 2` uses the same local-palette compressed screen format as `chunks 0`, `1`, and `4`
- it is no longer reasonable to treat `chunk 2` as only a grayscale utility table

This pass updated the current-effort decoder and produced fresh outputs for `chunk 2`.

### Chunk 6 load path is now executable-confirmed

The startup path loads `chunk 6` as:

- `u32 decompressed_size`
- `u32 compressed_size`
- compressed payload

It then allocates the decompressed size and runs the verified RLE decoder into that buffer.

This matches the already verified `200x175` smiley-pattern interpretation.

### Chunk 1 is the gameplay base screen

The startup path loads `chunk 1` as:

- `768` palette bytes
- `u32 compressed_size`
- compressed `320x240` screen payload

It decodes the image into the working screen buffer, then copies it into all three page buffers used by the VGA presentation path.

Before those page copies, it calls `0x00002188`.

The most likely current interpretation of `0x00002188` is:

- copy a pre-zeroed `50x50` block into a fixed region of the gameplay base screen

Because:

- the source buffer is the separately allocated `0x9c4` block
- that block is zero-filled by the allocator wrapper
- the helper copies `50` rows of `50` bytes each

This looks more like clearing a UI/gameplay subregion than drawing art.

### Chunk 3 is much narrower now

The startup path loads `chunk 3` in this structure:

- `0x400` raw bytes into `0x2c6a3`
- `u32 value`
- `u32 value`
- compressed payload into scratch
- allocate decompressed output buffer
- RLE decode the payload into that new buffer

So `chunk 3` is not a single opaque blob anymore. It is a mixed-format resource:

- a fixed `1024`-byte raw block
- followed by a compressed secondary block

That strongly reduces the search space.

The raw `0x400` region still looks table-like and may be:

- a lookup table
- a remap table
- a font or sprite index table
- a per-piece or per-state descriptor table

This chunk remains unresolved, but it is now a much tighter target.

### Chunk 8 is the overlay path

Later in the same init chain, the executable loads `chunk 8` with:

- `u32 compressed_size`
- compressed payload

and decodes it into the preallocated `0x1200` buffer.

That exactly matches the already verified `128x36` `GAME OVER` overlay size.

## Setup-Oriented Behavior In The Same Chain

After the resource loads, the same startup path enters a UI loop that clearly manipulates:

- music volume state
- sound effects volume state
- frame presentation / redraw timing
- input-driven increment and decrement behavior

This strongly supports the idea that the init path is also responsible for bringing up the early setup or configuration flow before handing off to the rest of the menu system.

## Workspace Changes From This Pass

This pass updated the current-effort graphics decoder:

- `Decompilation.Effort/scripts/decode_verified_graphics.py`

New practical outputs:

- `chunk 2` is now emitted as a verified `320x240` screen bundle
- `chunk 7` is now emitted as eight `128x224` panels instead of guessed `320`-wide strips

Stale strip outputs from the older chunk-7 interpretation were removed so the folder matches the current manifest.

Update:

- the later executable-side follow-up in [chunk7-usage-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/chunk7-usage-pass-2026-04-13.md) shows that chunk `7` is more likely a bank of structured frontend-animation records than a literal bitmap panel set
- those `128x224` renders should now be treated as heuristic artifacts, not faithful final decodes

## Recommended Next Move

The next best executable-side targets are:

- `0x00003830`
- `0x00006544`
- `0x0000206c`
- `0x00002e18`
- `0x00002f24`

Those functions are close to the setup/init UI loop and are the most likely places to answer:

- what `chunk 2` actually depicts
- what the mixed-format `chunk 3` data controls
- which screen/state IDs are being selected during startup

## Bottom Line

This was a useful pass.

It converted one major uncertainty into a verified format:

- `chunk 2` is now a real screen resource

It also narrowed the hardest remaining early-format problem:

- `chunk 3` is now known to be a `1024`-byte raw table plus a separate compressed block

That is a much better place to continue from than the earlier "unknown chunk" bucket.
