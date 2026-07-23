# RNG / Piece-Selection Port Parity Pass

Date: 2026-07-21

## Summary

This pass certifies the port's gameplay random-number generator and piece
selection against the exported decompilation. The port already reproduced the
original RNG; this pass confirms it is **bit-exact given a seed** and resolves
the long-open "exact startup seed source" question (`GAME-06`): the seed comes
from a shared entropy global with no recovered deterministic write, so it is
effectively timer-like and not a fixed value to match.

No port code change was required beyond an in-code provenance comment.

## Decompilation Evidence

From `research/ghidra/exports/decompilations/`:

- Table allocation `0x32f0`: `malloc(0x8000)` (32768 bytes = 8192 dwords),
  pointer stored at `0x2cb43`.
- Table fill `0x32b0`:
  1. `seed = [0x2d2a3]`, then `srand` via `0xed0f`
  2. loop filling the table: `high = rand()`, `low = rand()`,
     `entry = (high << 16) | low`, advancing 4 bytes until byte offset `0x8000`
     (i.e. all 8192 dwords)
  3. reset the consume index at `0x1886b` to 0
- `rand` `0xeceb`: the Watcom/ANSI C LCG `state = state*0x41C64E6D + 0x3039`
  returning bits 16..30 (`(state>>16) & 0x7FFF`). `0x41C64E6D = 1103515245`,
  `0x3039 = 12345`.
- Consumption `0x3280`: `++index; if (index >= 0x1FFE) index = 0;
  return table[index]`. Only 8190 (`0x1FFE`) of the 8192 filled entries are ever
  cycled; entries `0x1FFE`/`0x1FFF` are never read.
- Piece selection (`gameplay-entry-pass`, `0x5e0`): the returned table value
  modulo 7 chooses the tetromino, stored at `0x2c6ef`.

## Port Match

`MilestoneADemo` (`milestone_a_demo.cpp`) reproduces each stage exactly:

- `gameplay_random_table_` is `std::array<uint32_t, 8192>` — same size as the
  original allocation.
- `RefillGameplayRandomTable`: `state = seed`, then fills all 8192 entries with
  `(NextGameplayRandom15() << 16) | NextGameplayRandom15()`.
- `NextGameplayRandom15`: `state = state*0x41C64E6D + 0x3039;
  return (state>>16) & 0x7FFF` — matches `rand` `0xeceb`, and `state = seed`
  matches `srand`.
- `NextGameplayRandomU32`: `++index; if (index >= 0x1FFE) index = 0;
  return table[index]` — matches `0x3280` exactly.
- `RollNextPieceType`: `NextGameplayRandomU32() % 7`.

## Verification (Windows/MSVC, offscreen + dummy audio)

A standalone reference model of the decompiled RNG (Watcom `rand` + `0x32b0`
fill + `0x3280` consume + `%7`) was compared to the port's observed piece
sequence via `--piece-seed` and the `piece=`/`next=` `--debug-state` fields:

- Initial piece (`table[1] % 7`, since the index pre-increments from 0):
  seed 1 -> 4, seed 7 -> 2, seed 42 -> 6. The port matches all three.
- The port's `next=` run-collapsed spawn sequence for seed 1
  (`4 1 5 2 3 2 5 1 6 ...`) is an exact prefix of the reference model's
  collapsed sequence (`4 1 5 2 3 2 5 1 6 1 2 0 ...`).
- Determinism: the same seed produces the same sequence across runs (seed 42 ->
  `piece=6` on both).

## Seed Source Resolution

The srand seed is read from global `0x2d2a3`. Across the exported
decompilation there is no deterministic write to `0x2d2a3`; it is read as a
shared entropy value by several subsystems (the RNG refill and the chunk-7
frontend object animation among them). The best reading is a startup/timer-seeded
entropy global rather than a fixed constant, so there is no fixed original seed
to reproduce. The port models this with a settable seed (`--piece-seed`,
non-zero) and a fixed default for deterministic smoke runs.

## Checklist Impact

`GAME-06` is upgraded: the recovered random-table modulo-7 piece path is now
certified bit-exact against the decompilation (`0x32b0`/`0x3280`/`0xeceb`), and
the "exact startup seed source" is resolved as a non-deterministic shared
entropy global (`0x2d2a3`) rather than a recoverable fixed value.

## Not In Scope

- Row-clear helper velocities / per-helper random consumption (`0x14b4`..
  `0x1c40`) remain uncertified; those helper bodies are not decompiled and the
  branch is paused per `line-clear-style-source-closure-pass-2026-04-20`.
