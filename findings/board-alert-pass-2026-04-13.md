# Board Alert Pass

Date: 2026-04-13

## Summary

This pass resolved the fixed `50x50` gameplay-side alert tile system.

The biggest result is that the executable-side UI logic now lines up directly with recovered chunk data:

- the alert art bank uses `0x9c4`-byte frames
- `0x9c4` is exactly `50 x 50`
- that matches the recovered chunk-6 tile bank

So the small gameplay-side icon or alert area is no longer an abstract helper cluster. It is a concrete asset-driven subsystem.

## The Alert Tile Lives At A Fixed Screen Position

Multiple helpers use screen offset `0xaf14`.

In a `320`-byte linear framebuffer that is:

- `y = 140`
- `x = 20`

That matches the calls that mark the dirty region through `0x2d30`:

- `EAX = 0x14`
- `EDX = 0x8c`
- `EBX = 0x32`

So this subsystem owns a fixed `50x50` tile area at:

- `x = 20`
- `y = 140`

## `0x2138` And `0x2188` Save And Restore The Alert Region

`0x2188`

- copies `50` bytes per row for `50` rows
- source: live screen at `0xaf14`
- destination: backing buffer at `0x2c6b3`

Best reading:

- save the current background under the gameplay alert tile

`0x2138`

- copies the same `50x50` region back from `0x2c6b3`
- writes it to the live screen at `0xaf14`
- marks the region dirty afterward

Best reading:

- restore the background under the gameplay alert tile

Together these two functions give the alert tile a clean save/restore lifecycle instead of drawing destructively into the gameplay art.

## `0x1793d` Draws Full Alert Frames

`0x1793d` treats the alert art bank as:

- frame size `0x9c4`
- `50x50` pixels per frame
- source base pointer at `0x2c69b`

It draws the selected frame into the alert region at `x = 20`, `y = 140`.

Important detail:

- source pixels with palette index `0x20` are treated as transparent

That means these are not opaque square tiles. They are icon-like overlays intended to sit on top of the saved gameplay background.

## `0x17983` Progressively Reveals Alert Frames

`0x17983` uses the same frame bank and target region, but takes a row count in `EBX`.

It:

- draws only the requested number of rows from the alert frame
- fills the remaining rows from the saved background buffer at `0x2c6b3`

Best reading:

- reveal or unreveal the alert icon vertically over the saved background

This is the helper that turns the alert tile into a staged animation instead of a one-frame pop-in.

## `0x206c` Is The Alert Tile Runtime Update Loop

`0x206c` is the controller for this subsystem.

When called with `EAX = 1`:

- clears the active alert lifetime at `0x2c75f`
- clears the six-step reveal counter at `0x2c767`
- restores the saved background through `0x2138`
- sets the active effect ID at `0x2c76f` to `-1`
- resets the warning cooldown fields used elsewhere in gameplay

During normal updates:

- if an alert is active, it decrements its remaining lifetime
- while the reveal counter is still positive, it uses the fixed table:
  - `50`
  - `40`
  - `30`
  - `20`
  - `10`
  - `1`
- those values are fed into `0x17983`
- after each draw, the alert tile region is marked dirty and the reveal counter is decremented
- when the lifetime reaches zero, the background is restored and the active effect ID is cleared

Best reading:

- update the gameplay alert icon area each frame
- animate it in with a staged vertical reveal
- keep it visible for a requested lifetime
- then restore the background

## `0x2008` Triggers Alert Effects

`0x2008` is the public trigger helper for this subsystem.

Its inputs are effectively:

- `EAX` -> alert or effect ID
- `EDX` -> duration-like value
- `ECX` -> caller-supplied mode or priority value used during staging

Observed behavior:

- if the requested effect is new and no alert is active, it arms the effect and starts the reveal sequence
- if the same effect is requested again, or another alert is already active, it forces an immediate full-frame draw through `0x1793d`
- it stores the active effect ID in `0x2c76f`
- it stores the active lifetime in `0x2c75f`
- it sets the reveal counter in `0x2c767`

Best reading:

- trigger or refresh the gameplay alert tile animation

## Where This Shows Up In Gameplay

This same subsystem is reused in several places:

- stack-height warning logic at `0x21c4`
- line-clear reward or feedback logic in the gameplay loop at `0x09c8`
- spawn-failure / game-over entry path

That reuse strongly suggests the alert tile is a general-purpose gameplay status area rather than a one-off warning icon.

## Asset Correlation

The strongest cross-check in this pass is:

- chunk 6 was already shown to contain `50x50` tiles
- the alert draw path consumes `50x50` frames from `0x2c69b`

Best current reading:

- chunk 6 is the source art bank for the gameplay alert tile system

That is one of the cleanest executable-to-asset matches in the project so far.

## Recommended Next Move

The next best follow-up is to connect the alert IDs to captured gameplay evidence:

- map which chunk-6 tile corresponds to effect IDs `3`, `4`, `5`, `6`, `8`, `10`, `11`, and `12`
- correlate those IDs with warning, line-clear, and game-over moments in the video captures
- determine whether the alert tile system also covers any menu-side or setup-side status art

Update:

- the direct ID-to-tile correlation work is now captured in [alert-id-correlation-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/alert-id-correlation-pass-2026-04-13.md)
