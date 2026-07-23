## Gameplay Present Order And Resume Delta Pass - 2026-04-14

This pass stayed on the gameplay-side presentation seam and tightened two related questions:

1. what the gameplay-side helper region around `0x2e18 .. 0x3033` is really doing
2. what kinds of visible changes can land on the first resumed gameplay frame **before** the transient particle redraw

### New Owned Artifacts

- `research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-2e18-3033.asm`
- `research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-09c8-0ae0.asm`
- `research/ghidra/exports/decompilations/gameplay-present-order-pass/raw-206c-21b0.asm`
- `research/ghidra/exports/decompilations/gameplay-present-order-pass/gameplay-present-order-pass.index.json`
- `research/ghidra/gameplay-present-order.json`

These raw artifacts are meant to pair with the earlier outer session-loop raw artifact rather than replace it.

### Main Result

The first resumed gameplay frame is now tighter than:

- restored snapshot
- then particles

The stronger current reading is:

1. tracked transient cleanup can alter the restored snapshot first
2. `0x2e18` advances or recycles object state but does not itself redraw
3. `0x09c8` can already perform direct gameplay block blits
4. `0x206c` can already perform alert-region restore or reveal work
5. `0x2f24` then draws tracked transient particles over that staged gameplay image
6. `0x17719 -> 0x24d0` presents the result

That is a better fidelity model for the port.

### `0x2e18` Region

The raw helper export confirms `0x2e18` is an update/recycle pass, not a draw pass.

What it directly does:

- walks the active object list
- advances object age by `[object + 0x18]`
- removes expired records
- advances positions by velocity
- removes out-of-bounds records

What it does **not** do:

- it does not directly call the tracked pixel plotter
- it does not directly touch the working screen

That keeps simulation and drawing cleanly separated.

### `0x2f24` Region

The raw export confirms `0x2f24` is the actual tracked transient draw pass.

What it directly does:

- walks the active object list
- samples the chunk-3 color-ramp table through `0x2c6a3`
- computes the live palette byte
- calls `0x17821`

So the tracked transient overlay is still best modeled as the last gameplay-local writer before the shared flush boundary.

### `0x09c8` And `0x206c` Together

The important new thing is not just their individual roles, but their ordering relative to `0x2f24`.

The owned raw session-loop artifact already showed:

- `0x2e18`
- `0x09c8`
- `0x206c`
- `0x2f24`
- `0x17719`
- `0x24d0`

This pass tightened what that means visually.

#### `0x09c8`

Its owned raw helper export confirms that gameplay block-blit work can happen inside the gameplay step itself.

Direct evidence includes:

- `0x11e0` near the top of the step to erase the current piece
- `0x10ec` later to redraw the piece
- `0x2c58` and `0x1348` on the spawn/update path

So the gameplay step can already change the visible working image before transient particles are redrawn.

#### `0x206c`

Its owned raw helper export confirms that alert-tile work is also a real screen-writing stage, not just state maintenance.

Direct evidence includes:

- reset path: `0x2138` then `0x2d30`
- reveal path: `0x17983` then `0x2d30`

So an active or resetting alert tile can also change the working image before transient particles are redrawn.

### Practical Fidelity Consequence

The first resumed gameplay frame can already include:

- restored tracked pixels
- newly redrawn piece or board-adjacent HUD state
- alert tile restore or reveal changes
- then tracked transient particles over the top

That is more precise than the older wording and is a better target for the future port.

### `0x1765a`

This pass did not materially improve `0x1765a`.

That is a useful negative result:

- we got stronger on the gameplay present seam
- without pretending the unresolved helper is more closed than it is

### Updated Guidance

The preservation model should now treat gameplay presentation as:

- direct block writes and alert-region work first
- tracked transient overlay second
- shared dirty flush and page flip last

Even on a modern single-backbuffer renderer, keeping that ordering will matter for crisp parity.

### Next 5 Strongest Moves

1. Tighten the remaining ownership question around `0x1765a`, especially whether it is dead, indirect-only, or reached from a not-yet-exported gameplay or frontend helper.
2. Do the same low-level primitive-separation pass on the steady frontend menu loop so object pixels, text reveal, and highlight pulses are described at the same fidelity as gameplay now is.
3. Tighten the first resumed gameplay frame in specific edge paths: active alert tile, active tracked particles, and spawn/next-piece transition.
4. Keep expanding the owned raw artifact set around the gameplay session loop so first-frame and game-over-to-frontend edge cases remain grounded in direct evidence.
5. Continue converting these low-level results into direct porting constraints so the future C++23/SDL3 implementation preserves layering and update order instead of only reproducing final images.
