# Dirty Flush Queue Capacity Pass

Date: 2026-04-15

## Summary

This pass closes the remaining low-level uncertainty around `0x17719`.

Key result:

- `0x17719` is now executable-closed as a two-stage dirty flush backend with an exactly sized queue (`2400` cells), deterministic copy pattern, and consistent `flush -> present` caller pairing.

## 1) Two-Stage Backend Is Explicit

`0x17719` performs:

1. queue build over dirty map `0x1ad97`
2. planar VGA flush from that queue

Queue build stage details:

- clears count at `0x1ad8f`
- scans `40x60` dirty cells (`8x4` pixels each)
- for each non-zero dirty byte:
  - queue destination pointer (`dst`) and source pointer (`src`)
  - increment queue count
  - decrement dirty byte by `1`

Flush stage details:

- loops plane masks `1,2,4,8` through port `0x3c4`
- iterates queued cells in reverse order per plane
- writes four 2-byte destination rows (`+0x000,+0x050,+0x0a0,+0x0f0`)
- sources bytes from linear offsets matching `8x4` planar extraction:
  - `+0x000/+0x004`
  - `+0x140/+0x144`
  - `+0x280/+0x284`
  - `+0x3c0/+0x3c4`

## 2) Queue Capacity Is Exactly One Full Dirty Sweep

Queue region:

- base: `0x1b6f7`
- next known region base: `0x201f7`
- delta: `0x4b00` (`19200`) bytes

Queue entry size is `8` bytes, so capacity is:

- `19200 / 8 = 2400` entries

Dirty grid cell count is also:

- `40 * 60 = 2400`

So one complete dirty-map sweep fits exactly.
There is no overflow branch needed in the normal stage-1 design.

## 3) Direct Caller Closure

Full flat-image direct call-target scan (`E8 rel32`) to `0x17719` resolves exactly `14` callsites:

- `0x05b3`
- `0x3ebc`
- `0x3fad`
- `0x4553`
- `0x4974`
- `0x4f09`
- `0x5081`
- `0x5368`
- `0x55e2`
- `0x56ce`
- `0x5839`
- `0x5ec4`
- `0x6382`
- `0x6463`

Across these sites, the sequencing is stable:

- `0x17719` is immediately followed by `0x24d0`

So this helper remains the shared flush seam just before present pacing.

## Fidelity Impact

For the port, preserve:

- counted dirty decay semantics
- two-stage list build then flush behavior
- shared backend ownership across gameplay and frontend producers
- strict `flush -> present` ordering

This closes the prior “remaining low-level dirty-cell flush behavior around `0x17719`” item.

## New Machine-Readable Artifact

- `/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/dirty-flush-queue-model.json`
