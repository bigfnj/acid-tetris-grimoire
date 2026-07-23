# Gameplay Escape And Transient Render Pass

Date: 2026-04-14

## Summary

This pass resolves three helpers around the outer gameplay loop:

- the transient-pixel restore flush at `0x17875`
- the game-over overlay restore helper at `0x23ac`
- the byte-compare helper at `0xe6c8`

The most useful practical result is that the live session loop now has a clear `Esc` handoff model:

- `Esc` during live gameplay returns to frontend state `1` `Main menu`
- `Esc` after the game-over animation fully completes restores the overlay background and enters frontend state `9` `High-score qualification / name entry`

## `0x023b` Is A Loop Region Inside The Main Runtime Entry

One important structural correction:

- `0x023b` is a logical game-session loop region inside the main runtime path that starts at `0x0018`
- it is not a clean standalone function boundary with its own prologue

That matters because the odd `esp + 0x10` compare inside the session loop is using inherited stack-local data from the main entry path.

## `0xe6c8` Is A Straight Byte-Compare Helper

`0xe6c8` is a compact byte-compare helper.

Recovered behavior:

- `EAX` -> first byte pointer
- `EDX` -> second byte pointer
- `EBX` -> byte count
- returns `0` when the ranges match
- returns nonzero when they differ

It is effectively a local `memcmp`.

## Why The Session Loop Compare Really Means `Esc`

Inside the main runtime entry at `0x0018`, the code copies bytes beginning at `0x11` and also stores the byte at `0x10` into `[esp + 0x10]`.

That inherited local block begins with:

- `0x01 73 65 74 75 70 ...`

The important first byte is `0x01`, which is the Set-1 keyboard scancode for `Esc`.

In the game-session loop, the branch at:

- `0x04a3 -> 0x04b1 -> CALL 0x0000e6c8`

compares the current input-latch tail at `0x2c227 + 0x1f` against that inherited local block, using the current catch-up step index in `EBP` as the byte count.

In the normal case, the first inner step runs with `EBP = 1`, so this reduces to:

- compare the most recent released scan code against `0x01`

That makes the branch a practical `Esc`-release check.

## Session-Loop `Esc` Handoff

Once that compare succeeds, the outer gameplay loop does this:

1. clears the release latch through `0x94c`
2. checks `0x184db`
3. if `0x184db == -2`, calls `0x23ac`, then enters frontend state `9`
4. otherwise enters frontend state `EBP`, which is normally `1` on the first inner step

From the already resolved frontend dispatcher:

- state `1` -> main menu
- state `9` -> high-score qualification and name-entry path

Practical reading:

- `Esc` during ordinary live gameplay returns to the main menu
- `Esc` after the finished game-over sequence hands off to the high-score entry flow

That is a useful preservation detail for the future Windows port.

## `0x23ac` Restores The GAME OVER Background Before State `9`

`0x23ac` was already identified as the overlay-background restore helper, but its role in the outer loop is now clearer.

Recovered behavior:

- copies the saved `128x36` playfield region from `0x2c603` back into the live screen
- marks the restored region dirty through `0x2d30`

The session loop calls it only from the `Esc` handoff path when:

- `0x184db == -2`

That matches the `0x1f8c` game-over progression:

- `0x1f8c` advances the game-over animation
- when it completes, it sets `0x184db = -2` and clears the live-game flag at `0x2c72b`
- then the session loop uses `0x23ac` to remove the GAME OVER overlay before transferring control to state `9`

## `0x17821` And `0x17875` Form A Transient Restore Pair

The earlier particle/object mapping was correct, but this pass makes the transient render model much more concrete.

### `0x17821`

`0x17821`:

- computes a screen byte pointer from `(x, y)`
- writes the temporary draw byte from `CL`
- marks the coarse dirty-region byte at `0x1ad97 + cell_index` as `3`
- queues three restore fields:
  - original screen byte value
  - screen byte pointer
  - dirty-region pointer

Those queued fields are stored in the restore tables rooted at:

- `0x201f7`
- `0x201fb`
- `0x201ff`

with a count at:

- `0x1ad8b`

### `0x17875`

`0x17875` walks that restore queue in reverse and:

- restores the original screen byte
- rewrites the corresponding dirty-region cell to `3`
- clears the queued-item count back to `0`

Call graph proof:

- only caller of `0x17821` found so far: `0x2f69` inside `0x2f24`
- only caller of `0x17875` found so far: `0x048f` at the top of the outer gameplay loop

Practical reading:

- `0x2f24` draws transient particle/object pixels for the current rendered frame
- `0x17875` restores those pixels at the start of the next outer frame before the next simulation/render pass

So this is not a generic dirty-rect system by itself. It is a specific transient-pixel restore queue used by the particle/object layer.

## Porting Impact

This gives the future source port several preservation-important rules:

- `Esc` should route to main menu during live play, not immediately quit
- post-game-over `Esc` should restore the overlay and then hand off to high-score qualification/name entry
- transient particle/object pixels should be modeled as frame-local overlays that are restored before the next outer frame
- the session loop’s input-latch path is release-driven for this `Esc` transition, just like the rest of the frontend/menu model

## Recommended Next Move

The next strongest unresolved loop-edge target is now:

- `0x17719`

That helper still sits in the outer render phase between particle drawing and page flip, and resolving it should finish most of the remaining uncertainty around the session-loop presentation layer.
