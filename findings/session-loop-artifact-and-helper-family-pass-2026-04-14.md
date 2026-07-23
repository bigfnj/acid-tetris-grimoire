## Session Loop Artifact And Helper Family Pass - 2026-04-14

This pass did two things:

1. Created a dedicated owned raw-disassembly artifact for the full outer session loop so we are no longer leaning on ad hoc or truncated slices when describing the `Esc` and `state 9` handoff.
2. Resolved the nearby low-level helper family around `0x1765a .. 0x17700` enough to describe the gameplay blit/erase primitives directly instead of only through higher-level behavior.

### New Owned Raw Artifact

- Raw session-loop disassembly:
  - `research/ghidra/exports/decompilations/session-loop-raw-pass/raw-023b-05c4.asm`
- Artifact index:
  - `research/ghidra/exports/decompilations/session-loop-raw-pass/session-loop-raw-pass.index.json`

The raw artifact covers:

- startup handoff into the live outer loop
- the released-`Esc` compare path
- the finished-game-over `state 9` handoff
- the per-step gameplay update path
- the present boundary through the jump back to `0x048f`

That gives this project a clean owned source artifact for one of the most important control-flow regions in the executable.

### Helper Family Resolution

#### `0x17688` -> `blit_5x5_block_to_screen`

This helper is now high-confidence.

What it does:

- copies one dword plus one byte from `ESI` to `EDI`
- repeats that copy over five rows
- advances each row by the gameplay screen stride `0x140`

Why that matters:

- `0x2c58` (`draw_decimal_counter`) calls it directly
- the chunk-3 digit atlas is already resolved as `10` sequential `5x5` glyphs
- this makes `0x17688` the real low-level decimal-digit blitter

#### `0x176df` -> `blit_8byte_rows_to_screen`

This helper is now high-confidence.

What it does:

- copies two dwords per row from `ESI` to `EDI`
- advances `ESI` by `8`
- advances `EDI` by `0x140`
- repeats for the caller-supplied row count in `EDX`

Why that matters:

- the piece-draw path at `0x10ec` calls it with `EDX=8`
- the source points into the chunk-3 gameplay tile atlas
- this makes `0x176df` the real low-level `8x8` gameplay tile blitter

#### `0x17700` -> `clear_8byte_rows_to_zero`

This helper is now high-confidence.

What it does:

- zero-fills two dwords per row at `EDI`
- advances `EDI` by `0x140`
- repeats for the caller-supplied row count in `EDX`

Why that matters:

- the piece-erase path at `0x11e0` calls it with `EDX=8`
- it matches the exact geometry used by `0x176df`
- this makes it the erase partner for the gameplay tile blitter

#### `0x1765a` -> `swap_pixel_and_mark_dirty_untracked`

This helper is useful, but still only medium-confidence on its exact subsystem role.

What it definitely does:

- computes a working-screen byte from `(x,y)` using the same address math family as `0x17821`
- reads the previous byte
- writes the new color from `CL`
- marks the corresponding dirty cell as `3`

Why it is not yet fully locked:

- unlike `0x17821`, it does **not** append to the tracked restore queue at `0x1ad8b`
- current whole-binary call and jump sweeps did not isolate a clean direct caller or a separate entry at `0x17660`
- so the behavior is clear, but the exact owner path is not yet direct enough to call fully closed

The best current reading is that this is an **untracked pixel-swap primitive** used by some transient or special-case presenter that wants dirty propagation without queue-backed restoration.

### Why This Improves Fidelity

These low-level helpers matter for the future port because they tell us the original renderer is not one generic sprite system. It has distinct primitives for:

- direct `5x5` HUD digit blits
- direct `8x8` gameplay tile blits
- direct `8x8` zero erases
- tracked transient pixel plotting/restoration
- untracked one-pixel dirty updates

That separation is exactly the kind of detail that keeps a preservation port crisp instead of "close enough."

### Map Updates

This pass also tightened the higher-level callers in the function map:

- `0x10ec` now explicitly records its dependency on `0x176df`
- `0x11e0` now explicitly records its dependency on `0x17700`
- `0x2c58` now explicitly records its dependency on `0x17688`

### Current Confidence Summary

- `0x17688`: high
- `0x176df`: high
- `0x17700`: high
- `0x1765a`: medium
- dedicated raw session-loop artifact: high-value and complete enough for future passes

### Next 5 Strongest Moves

1. Tighten the caller set for `0x1765a` so we can either close it as a specific transient presenter primitive or keep it deliberately generic.
2. Resolve the nearby `0x17660 .. 0x17684` neighborhood as a family, not just as one helper, so we know whether there is a second entry or alias pattern we should preserve.
3. Keep expanding the owned raw session-loop artifact set around `0x023b` so first-frame resume and post-game-over paths can be quoted from direct raw evidence instead of scattered slices.
4. Tighten the remaining low-level dirty/present seam around `0x17719` with the new helper-family context, especially how tracked and untracked pixel changes coexist in one presented frame.
5. Reflect these low-level renderer primitives into the transition preservation spec so the eventual SDL port keeps the original layering model instead of collapsing everything into one generic blit path.
