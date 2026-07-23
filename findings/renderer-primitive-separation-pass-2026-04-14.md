## Renderer Primitive Separation Pass - 2026-04-14

This pass stayed focused on the low-level presentation seam.

The goal was not just to name more helpers, but to answer a fidelity question that matters to the future port:

- are all localized screen updates really one generic writer family?

The current direct-call evidence says **no**.

### New Owned Artifact

- Raw helper-family disassembly:
  - `research/ghidra/exports/decompilations/renderer-primitive-pass/raw-175c5-178af.asm`
- Machine-readable primitive map:
  - `research/ghidra/renderer-primitive-map.json`

### Main Result

The executable currently supports a cleaner renderer model than "everything draws pixels and `0x17719` presents them."

There are at least four distinct writer families:

1. frontend object pixels
2. gameplay tracked transient pixels
3. gameplay direct block blits
4. one unresolved untracked pixel-swap helper

All of them ultimately feed the same dirty-cell flush backend at `0x17719`, but they are not interchangeable and should not be collapsed together in the preservation port.

### Frontend Object Pixel Family

Helpers:

- `0x175c5` -> `plot_frontend_pixel_if_empty`
- `0x17613` -> `clear_frontend_pixel`

Current direct-call sweep result:

- all direct callers are in the frontend object / transition family
- no current direct gameplay caller was found

That is a useful fidelity boundary.
These helpers belong to the chunk-7 floating menu-object system and its transitions, not to gameplay particles.

### Gameplay Tracked Transient Family

Helpers:

- `0x17821` -> `plot_tracked_particle_pixel`
- `0x17875` -> `restore_tracked_particle_pixels`

Current direct-call sweep result:

- `0x17821` is currently reached through `0x2f24`
- `0x17875` is currently reached at the top of the outer gameplay loop
- no current direct frontend caller was found

That means the tracked restore queue is still best modeled as a gameplay-local transient system, not a shared frontend/gameplay overlay mechanism.

### Gameplay Block-Blit Family

Helpers:

- `0x17688` -> `blit_5x5_block_to_screen`
- `0x176df` -> `blit_8byte_rows_to_screen`
- `0x17700` -> `clear_8byte_rows_to_zero`

These are now clearly the direct low-level building blocks for:

- HUD digits
- gameplay tile drawing
- gameplay tile erasure

They write directly into the linear working screen and depend on higher-level callers to mark the affected region dirty.

### `0x1765a` Status

`0x1765a` is still best kept provisional.

What is firm:

- it reads the old byte
- writes the new color from `CL`
- marks the dirty cell as `3`
- does **not** queue tracked restoration

What is not firm:

- no direct call or jump reference to `0x1765a`
- no direct call or jump reference to a separate entry at `0x17660`

So the current best reading remains:

- useful helper
- behavior understood
- owning subsystem not yet closed

### Shared Present Boundary

This pass also strengthened the shared present ordering on the gameplay side.

The owned raw session-loop artifact already showed the key order:

1. `0x17875`
2. `0x2e18`
3. `0x09c8`
4. `0x206c`
5. `0x2f24`
6. `0x17719`
7. `0x24d0`

That means:

- tracked transient cleanup happens before gameplay simulation
- fresh tracked transient drawing happens after gameplay simulation
- `0x17719` is the common present boundary, not the producer

### Why This Matters For The Port

The future `C++23 + SDL3` port should preserve the separation between:

- persistent frontend floating objects
- gameplay tracked transient particles
- direct gameplay tile/HUD block writes

Even if all of them target one modern backbuffer, the update timing and lifetime model should stay distinct.

That will make a faithful port much crisper than a design that routes every visible change through one generalized sprite/effect layer.

### Spec And Map Updates

This pass updated:

- `function-hypotheses.json`
- `transition-preservation-spec.md`
- `renderer-primitive-map.json`

### Next 5 Strongest Moves

1. Tighten the remaining ownership question around `0x1765a`, including whether it is dead, indirect-only, or reached through a not-yet-exported caller.
2. Build one more owned raw artifact around the gameplay-side transient/object helpers so `0x2e18`, `0x2f24`, and `0x206c` are grounded as directly as the session loop now is.
3. Tighten the first resumed gameplay frame one more step with this primitive split in mind, especially whether direct block blits can land before or after the first transient redraw in any edge path.
4. Do the same primitive-separation pass on the frontend steady-state menu loop so the title/menu object cycle and text/hilite rendering are described at the same low level.
5. Keep converting these low-level renderer findings into direct implementation constraints for the future port so we preserve behavior without reproducing DOS VGA literally.
