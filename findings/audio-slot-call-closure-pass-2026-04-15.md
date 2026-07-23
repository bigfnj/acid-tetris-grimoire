# Audio Slot Call Closure Pass

Date: 2026-04-15

## Summary

This pass closes the direct-call coverage around runtime audio-slot usage so slot `2` can be treated with higher confidence.

Result:

- `slot 2` is startup-loaded but unplayed in the shipped executable
- all direct runtime SFX playback still funnels through `0x6817`
- the full-image call-target scan agrees with our existing function-level disassembly

## Method

We scanned the full flat-relocated payload:

- `/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin`

using byte-level direct near-call matching:

- opcode `E8 rel32`

and resolved destinations for key audio helpers:

- `0x67f8` (slot loader)
- `0x680b` (slot release by index)
- `0x6817` (slot playback)
- `0x6544` (music track load or switch)

## Direct Call Results

- Calls to `0x67f8`: `12` sites (`0x01e3..0x02d5` startup cluster)
- Calls to `0x680b`: `1` site (`0x67d5`, cleanup loop)
- Calls to `0x6817`: `13` sites
  - `0x0d4e`, `0x0f4f`, `0x1065`, `0x10dc`, `0x2244`, `0x2294`, `0x22e3`, `0x4445`, `0x447c`, `0x487f`, `0x48ba`, `0x4c9e`, `0x4cd7`
- Calls to `0x6544`: `2` sites (`0x0474` startup and `0x42d8` menu music-row switch)

## Slot Coverage Resolution

Resolved slot usage from the `0x6817` callset:

- observed played slots: `{0,1,3,4,5,6,7,8,9,10,11}`
- startup-loaded slots: `{0,1,2,3,4,5,6,7,8,9,10,11}`
- loaded but unplayed: `{2}`

Special note on cleanup:

- `0x67cf` uses `0x680b` to release slots `0..15` during shutdown
- this path consumes slot pointers for teardown only, not playback

## Fidelity Impact

For faithful first-pass behavior:

- preserve slot `2` in extraction and manifests
- do not map slot `2` to any required gameplay or frontend event yet
- keep it labeled as unused or cut-content candidate until contradictory evidence appears

## New Machine-Readable Artifact

- `/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/audio-slot-call-closure.json`
