# EXE Loader Function Map

Date: 2026-04-13

## What This Pass Did

This pass extended the headless Ghidra export work around the PMODE/W-relocated flat image and focused on the executable-side loader path. The goal was to turn the earlier "interesting function" list into a more usable subsystem map for source-port work.

The strongest new result is that the `ATET.DAT` chunk-offset table is not embedded as initialized data in the executable image. It is loaded at runtime from the start of `ATET.DAT`, which explains why the flat relocated image shows zeroes in the `0x2cab7` region until initialization runs.

## Strong Corrections To Earlier Assumptions

- The runtime chunk table is loaded by code, not stored statically in the relocated image.
- `FUN_00003370` is an `ATET.DAT` stdio-style chunk loader, not a `SETUP.DAT` loader.
  Evidence:
  it opens `atet.dat` with the `"rb"` string at `0x17b07`.
- `FUN_000033e8` reads the `ATET.DAT` header count dword, multiplies it by 4, and reads the chunk-offset table into `0x2cab7`.
- `FUN_00005ef4` confirms that the decoded `chunk 7` data is treated as eight `128x224` panels.
  Evidence:
  it copies one `0x7000` byte slice from the decoded `1024x224` resource.
- `FUN_000062e0` and `FUN_000063b8` are palette fade helpers, not generic palette loaders.

## High-Confidence Function Hypotheses

- `0x00000770`
  Proposed role:
  allocation wrapper with failure handling.
  Evidence:
  returns allocator result, and on failure switches to text mode and prints an error path.
- `0x00002188`
  Proposed role:
  copy a `50x50` block into the active screen buffer.
  Evidence:
  copies `0x32` bytes per row for `50` rows with a `320`-byte stride.
- `0x000024d0`
  Proposed role:
  page-flip plus frame-throttle helper.
  Evidence:
  swaps page pointers, writes VGA CRTC start address registers, then waits on the runtime tick source.
- `0x00002438`
  Proposed role:
  set the game's `320x240` VGA planar video state.
  Evidence:
  uses BIOS video services, clears and loads a `768`-byte palette when needed, and programs VGA registers through `0x3c4`, `0x3c2`, and `0x3d4`.
- `0x00002574`
  Proposed role:
  load a `768`-byte VGA palette and mirror it into a cached copy.
  Evidence:
  writes to ports `0x3c8/0x3c9` for exactly `0x300` bytes.
- `0x000025c8`
  Proposed role:
  disk/file error retry screen.
  Evidence:
  contains strings for "Error during disk operation", retry, and quit; it is used by file open/read retry loops.
- `0x000027b8`
  Proposed role:
  build the `512`-entry sine table used by visual effects.
  Evidence:
  allocates `0x2000` bytes, computes `sin()` values, and mirrors positive/negative halves.
- `0x0000284c`
  Proposed role:
  scale a `768`-byte palette by a brightness factor.
  Evidence:
  processes exactly `0x300` bytes and multiplies each entry by a `(64 - factor) / 64` style term.
- `0x00002880`
  Proposed role:
  the verified four-mode chunk RLE decoder.
  Evidence:
  matches the independently verified control-byte structure and is used by all known compressed graphics resources.
- `0x00002938`
  Proposed role:
  copy a linear `320x240` buffer into VGA planar memory.
  Evidence:
  iterates four planes, writes to sequencer register `0x3c4`, and walks the source buffer in planar order.
- `0x00002998`
  Proposed role:
  load a `320x240` screen chunk, apply its palette, then reveal/display it with a timed transition.
  Evidence:
  reads palette and packed payload, decompresses with `0x2880`, blits to video memory, and uses timed loops plus palette work.
- `0x00003280`
  Proposed role:
  return the next value from the prebuilt random table.
  Evidence:
  increments a rolling index and reads from the table at `0x2cb43`.
- `0x000032b0`
  Proposed role:
  fill the random table with generated `u32` values.
  Evidence:
  writes `0x8000` bytes as `0x2000` dwords into the table allocated by `0x32f0`.
- `0x000032f0`
  Proposed role:
  allocate the random table buffer.
  Evidence:
  allocates `0x8000` bytes and stores the pointer at `0x2cb43`.
- `0x00003300`
  Proposed role:
  open `ATET.DAT` and seek to a chunk offset using low-level file APIs.
  Evidence:
  opens `atet.dat`, indexes the runtime chunk-offset table at `0x2cab7`, seeks to that offset, and stores the offset in `0x2cb47`.
- `0x00003370`
  Proposed role:
  open `ATET.DAT` in `"rb"` mode and seek to a chunk offset using stdio-style APIs.
  Evidence:
  opens `atet.dat` with `"rb"`, uses the same runtime chunk-offset table, verifies the final file position, and returns the file handle.
  This is the path used before `0x67f8` registers audio resources.
- `0x000033e8`
  Proposed role:
  load the `ATET.DAT` chunk-offset table at startup.
  Evidence:
  opens `atet.dat`, reads the first dword, multiplies by 4, then reads that many bytes into `0x2cab7`.
  This matches the observed archive structure:
  one count dword plus `27` explicit offsets.
- `0x0000349c`
  Proposed role:
  exact-read helper for the currently opened `ATET.DAT` low-level handle.
  Evidence:
  retries until the requested byte count is read or the disk error screen is shown.
- `0x00003520`
  Proposed role:
  one-time engine handler or interrupt/timer installation.
  Evidence:
  guarded by a once flag at `0x2cb37` and uses PMODE/W-era low-level service calls.
- `0x00003df4`
  Proposed role:
  run a short menu or transition frame pump over the active frontend animation records.
  Evidence:
  iterates the active chunk-7 record bank at `0x2d223`, draws repeated text lines, calls the frame-present helper at `0x24d0`, and loops until roughly `0x30` ticks have elapsed.
- `0x00005ef4`
  Proposed role:
  initialize the frontend animation bank and projection state.
  Evidence:
  picks a random bank index modulo `8`, copies one `0x7000`-byte slice from the decoded chunk-7 bank set into the active bank buffer, seeds the projection state, and initializes the next-bank field to `0`.
- `0x00005fd8`
  Proposed role:
  load the frontend animation resources and allocate their working buffers.
  Evidence:
  loads chunk metadata, allocates the active bank, delta-table, and decompressed bank-set buffers, expands chunk `7`, then calls `0x5ef4`.
- `0x000061c8`
  Proposed role:
  menu text width measurement plus positioned draw helper.
  Evidence:
  walks a string through font metadata, computes centered width when requested, then calls two text-rendering helpers.
- `0x000062e0`
  Proposed role:
  run the frontend animation fade-out loop.
  Evidence:
  clears previously drawn points, projects the active chunk-7 bank, redraws it, flushes dirty cells, and scales the palette with elapsed time over roughly `64` ticks.
- `0x000063b8`
  Proposed role:
  run the frontend animation fade-in loop.
  Evidence:
  uses the same chunk-7 erase/project/draw/flush loop as `0x62e0`, but with the inverse palette-brightness ramp.
- `0x000067f8`
  Proposed role:
  load or register an audio resource into a numbered slot.
  Evidence:
  hands the positioned file handle to another loader and stores the returned object pointer in a slot array at `0x2d257`.
- `0x00006817`
  Proposed role:
  play a loaded audio slot with volume and positional adjustment.
  Evidence:
  converts percent-like values into the engine's `0-64` scale and uses the object pointer loaded from the slot table.
- `0x000068bb`
  Proposed role:
  convert user-facing music volume to the engine's internal scale.
  Evidence:
  computes `value * 7 / 20` and stores a single-byte result.
- `0x000068d8`
  Proposed role:
  configure or reconfigure the audio engine.
  Evidence:
  used during startup after config is loaded and calls a lower-level audio setup chain.

## Updated Executable-Side Asset Pipeline

The current best reading of the startup pipeline is:

1. `0x00000018` performs top-level engine init and allocates the major working buffers.
2. `0x000032f0` allocates the random table and `0x000032b0` fills it.
3. `0x00002438` sets the VGA mode/state.
4. `0x000033e8` opens `ATET.DAT` and loads the runtime chunk-offset table.
5. `0x00003300` plus `0x0000349c` are the low-level chunk read path for graphics-like resources.
6. `0x00002880` decodes compressed chunk payloads.
7. `0x00002574`, `0x0000284c`, `0x000062e0`, and `0x000063b8` handle palette load and fade behavior.
8. `0x00002938` copies linear frame buffers into VGA planar memory.
9. `0x00003370` plus `0x000067f8` are the audio chunk load/register path.
10. `0x00005fd8` plus `0x00005ef4` load and initialize the chunk-7 frontend animation bank set and its projection state.

## What This Means For The Port

The source-port side is getting clearer. We now have enough confidence to implement the following modern equivalents without waiting for deeper decompilation:

- A clean `ATET.DAT` header parser:
  count dword plus offset table.
- A reusable four-mode RLE decoder for graphics chunks.
- Palette upload, palette scaling, and fade helpers.
- `320x240` linear buffer presentation logic.
- A `chunk 7` decoder that treats the payload as eight banks of `1024` records of `0x1c` bytes, plus the runtime projection and morph logic that turns those banks into the frontend pointfield effect.
- An audio-slot registry layer that mirrors the executable's "load handle into slot, then play by slot ID" behavior.

## Remaining Unknowns

- The exact semantics of `chunk 2` and `chunk 3` are still unresolved.
- The `SETUP.DAT` access path inside the executable still needs to be isolated cleanly.
- Some function boundaries in Ghidra are still undercut and need better recovery or manual reseeding.
  The most obvious example is the helper around `0x00003df4`.
- The text/font helpers below `0x000061c8` and the lower-level audio functions below `0x00006817` still need a deeper pass.

## Recommended Next Move

The next highest-value executable step is to recover and label the startup code around `0x0000023b` in more detail, especially the portion that loads:

- the gameplay base screen
- the font or glyph resources
- the `50x50` overlay block used by `0x00002188`
- the currently unresolved `chunk 2` and `chunk 3` data

That should close the gap between "we know the archive formats" and "we know how the original engine composes the final screens."
