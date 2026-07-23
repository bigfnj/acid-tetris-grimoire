# Executable Structure Pass

Date: 2026-04-13

## Scope

This pass formalized two things:

- the currently resolved graphics chunk formats
- the top-level executable structure of `ATET.EXE`

The goal was to turn recent discoveries into reproducible artifacts owned by the current workspace before moving deeper into loader and decoder analysis.

## Graphics State After Verification

The verified graphics decode outputs were spot-checked visually and line up with the current evidence set.

Resolved or mostly resolved graphics chunks:

- `chunk 0`
  `320x240` DDD splash screen with local palette
- `chunk 1`
  `320x240` gameplay HUD / empty board screen with local palette
- `chunk 4`
  `320x240` ACiD Tetris title screen with local palette
- `chunk 5`
  `192x66` font or glyph-sheet style image plus `66` bytes of metadata
- `chunk 6`
  `200x175` repeating smiley background pattern
- `chunk 7`
  structurally resolved frontend animation bank set: `8` banks of `1024` records of `0x1c` bytes each after decompression, with runtime projection and bank-morph behavior
- `chunk 8`
  `128x36` `GAME OVER` overlay

The newly rendered outputs under `extracted/converted/graphics/verified/` are now a better reference point than the earlier heuristic palette/chunk guesses.

Primary unresolved early chunks remain:

- `chunk 2`
- `chunk 3`

## Executable Structure Findings

`ATET.EXE` is not a single flat target from a reverse-engineering standpoint.

The DOS MZ header declares a file size of:

- `0x2CB0` bytes (`11440`)

The actual file size is:

- `0x173C5` bytes (`95173`)

That leaves:

- `0x14715` bytes (`83733`)

of appended data after the MZ-declared executable image.

This exactly matches Ghidra's headless import message:

- `File contains 0x14715 extra bytes starting at file offset 0x2cb0`

The appended region begins with:

- `PMW1`

This is a strong indication that the executable is organized as:

- DOS real-mode launcher / stub
- appended PMODE/W-managed protected-mode payload

## What Ghidra Did

Headless Ghidra import succeeded and created a project at:

- `Decompilation.Effort/research/ghidra/projects/atet-ghidra`

Import details from the log:

- loader: `Old-style DOS Executable (MZ)`
- language/compiler: `x86:LE:16:Real Mode:default`

This means the default import path is analyzing the DOS stub correctly, but it is not automatically giving us a full semantic view of the appended PMODE/W payload as a first-class 32-bit program image.

The import log is at:

- `Decompilation.Effort/logs/ghidra-atet-import.log`

## Payload Import Follow-Up

After splitting the executable layers, the extracted payload was also imported into the same Ghidra project as a separate raw-binary target.

Import details from the payload log:

- loader: `Raw Binary`
- language/compiler: `x86:LE:32:default:windows`

This is useful because it gives the current effort a practical 32-bit analysis foothold for the PMW1 payload, even though it is not yet a fully interpreted PMODE/W-aware load.

Important caution:

- this import should be treated as a provisional analysis view
- the load address, memory map, and compiler-spec assumptions may still need refinement

The payload import log is at:

- `Decompilation.Effort/logs/ghidra-pmw1-import.log`

## Watcom Disassembler Note

Open Watcom `wdis` does not directly disassemble `ATET.EXE` in its shipped form.

The tool reports:

- `The object file is not in OMF, ELF or COFF format`

That is expected and reinforces the current approach:

- use Ghidra for executable analysis
- keep Watcom tools as secondary support tooling, not as the primary loader/disassembler for this binary

## New Reproducible Artifacts

The current effort now owns a dedicated executable-layer splitter:

- `Decompilation.Effort/scripts/extract_exe_layers.py`

Running it writes:

- `Decompilation.Effort/research/disassembly/pmodew/ATET.EXE.mz-stub.bin`
- `Decompilation.Effort/research/disassembly/pmodew/ATET.EXE.pmw1-payload.bin`
- `Decompilation.Effort/research/disassembly/pmodew/ATET.EXE.layers.json`

This gives us a clean and reproducible separation between:

- the MZ stub, which Ghidra already understands
- the appended PMW1 payload, which is now the primary executable-side reverse-engineering target

## Notable String Hits

Within the appended payload, quick ASCII hits include:

- `PMW1`
- `PMODE/W`
- `WATCOM C/C++32 Run-Time`
- `setup`
- `AciD`
- `Tetris`
- `exit`

This is consistent with the earlier conclusion that the interesting game logic and menu functionality are living in the protected-mode payload rather than only in the DOS stub.

## Implications For Next Work

The next executable-side work should treat the program as two layers, not one:

1. keep the MZ stub as a known launcher layer
2. focus active reverse engineering on the extracted `PMW1` payload
3. correlate the verified asset formats with the code that consumes them
4. use chunk `2` and chunk `3` as anchor targets for loader-table and decode-path identification

The most promising immediate next technical move is:

- start working from the imported PMW1 payload inside Ghidra and correlate chunk-loading code with the now-resolved graphics formats
