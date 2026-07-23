# Dirty Flush Backend Pass

Date: 2026-04-14

## Summary

This pass revisits `0x17719` with the newer gameplay-driver context in place.

The main correction is:

- `0x17719` is not just a frontend pointfield helper
- it is the shared dirty-cell flush backend used by both gameplay and frontend presentation paths

That makes it one of the most important rendering helpers in the executable.

## `0x17719` Scans A `40x60` Dirty-Cell Grid

The function starts by clearing a queue count at `0x1ad8f`, then scans the dirty map at:

- `0x1ad97`

The scan geometry is:

- `0x28` columns
- `0x3c` rows

That is a `40x60` grid, which matches:

- `320 / 8 = 40`
- `240 / 4 = 60`

So each dirty entry corresponds to an `8x4` cell of the `320x240` working screen.

## What Gets Queued

For each non-zero dirty cell, `0x17719`:

- computes the source pointer from the linear working screen at `0x2c727`
- computes the destination pointer from the active VGA page rooted at `0x2c6ab`
- stores the source/destination pair into the temporary queue rooted at `0x1b6f7`
- increments the queued-cell count at `0x1ad8f`
- decrements the dirty byte itself by `1`

That decrement explains why callers often write the dirty map with the value `3`:

- cells can persist across a few flush passes without needing every caller to rewrite them immediately

## How The Copy Actually Works

After queue construction, `0x17719` loops across the four VGA planes by programming:

- port `0x3c4`

For each queued cell and each plane, it copies the `8x4` cell from the linear buffer into planar VGA memory as four 2-byte writes:

- row `0` -> destination `+0x000`
- row `1` -> destination `+0x050`
- row `2` -> destination `+0x0a0`
- row `3` -> destination `+0x0f0`

And for each row it reads the matching two bytes for the current plane from the linear source using offsets:

- `+0x000`
- `+0x004`
- `+0x140`
- `+0x144`
- `+0x280`
- `+0x284`
- `+0x3c0`
- `+0x3c4`

That is exactly the pattern expected for converting an `8x4` block from the linear `320x240` working screen into four planar VGA writes.

## Shared Call Graph Evidence

The new caller review matters here.

`0x17719` is reached from:

- the outer gameplay session loop at `0x05b3`
- menu/frontend transition loops at `0x3ebc` and `0x3fad`
- palette-fade and presentation loops at `0x6382` and `0x6463`
- multiple other frontend handlers already identified in the menu and score paths

That means the function is not specialized to chunk-7 pointfield animation.

It is the common backend flush that presents changed `8x4` cells from the working screen to the active VGA page.

## Relationship To Other Known Helpers

This now gives us a cleaner renderer model:

- `0x2c727`
  linear `320x240` working screen
- `0x2d30`
  coarse dirty-rectangle marker for larger rectangular edits
- `0x175c5`
  plot one frontend point if the destination is empty, then mark the dirty cell
- `0x17613`
  clear one frontend point, then mark the dirty cell
- `0x17821`
  plot one transient gameplay particle/object pixel and queue its restoration
- `0x17875`
  restore those transient particle/object pixels at the top of the next outer frame
- `0x17719`
  flush dirty `8x4` cells from the working screen into planar VGA memory

That is now a coherent presentation stack rather than a pile of isolated helpers.

## Porting Impact

This is a useful architectural checkpoint for the future SDL port.

We do not need to reproduce planar VGA writes literally, but we should preserve the logical separation:

- render into a linear working buffer
- track changed regions or cells
- present the changed result after logic and transient effects are resolved

In a modern port, we can choose between:

- preserving the dirty-cell update model directly
- or rendering the full frame every time while keeping the original logical layering order

Either choice is now informed rather than guessed.

## Recommended Next Move

The strongest remaining unknown in this render/presentation band is:

- `0x3ebc` / `0x3fad` caller context around `0x17719`

Those frontend loops already use the shared flush backend, and tightening their exact sequencing with `0x39c4`, `0x175c5`, and `0x24d0` would finish the presentation story for the menu-side animation path.
