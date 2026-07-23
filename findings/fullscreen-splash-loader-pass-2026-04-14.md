# Fullscreen Splash Loader Pass

Date: 2026-04-14

## Summary

This pass closes the last major ambiguity around `0x2998`.

The important result is that `0x2998` is not just a generic "load a screen chunk" helper.
It is the startup-era full-screen splash presenter for local-palette `320x240` screen chunks.

Its behavior is now much clearer:

- black the palette first
- load the chunk-local palette and compressed image
- decompress and upload the screen to planar VGA
- fade in
- hold
- fade out
- clear VRAM for the next stage

That matters because the startup presentation feel is part of the original game's identity, and we now have enough detail to recreate it deliberately in the future port.

## What `0x2998` Loads

The helper takes the chunk id in `EAX` and performs this load sequence:

1. zero a `0x300`-byte palette buffer on the stack
2. upload that black palette through `0x2574`
3. call `0x3300` to open `ATET.DAT` and seek to the requested chunk
4. read:
   - `0x300` palette bytes
   - `u32 compressed_size`
   - compressed payload
5. decompress the payload through `0x2880` into the linear `320x240` screen buffer at `0x2c727`
6. upload that linear screen to planar VGA through `0x2938`

So the screen is already present in VGA memory before the fade starts.
The visible transition is driven entirely by palette changes, not by repeated image redraw.

## Fade-In Phase

After the screen upload, `0x2998` snapshots the global tick at `0x2d2a3` and enters a palette-ramp loop.

Per iteration it:

- waits until the tick changes
- computes `elapsed = current_tick - start_tick`
- derives a half-speed brightness term from that elapsed count
- calls `0x284c` to build a temporary scaled palette
- uploads that temporary palette through `0x2574`

The effective visible result is:

- start at black
- brighten toward the chunk's full local palette
- over roughly `0x80` ticks

This is a palette-only fade.
It does not use `0x24d0`, `0x17719`, or any of the later frontend pointfield machinery.

## Hold Phase

Once the screen has finished fading in, the helper starts a second timed loop.

This phase:

- leaves the screen fully shown
- waits for about `0x78` ticks
- continues polling `0x0964` for `Esc` release

So each splash is not just a fade-in/fade-out flash.
It actually dwells on the fully visible screen for a short timed presentation window.

## Fade-Out Phase

After the hold interval, `0x2998` enters a second palette-scaling loop.

This loop mirrors the fade-in behavior:

- wait on the global tick
- compute elapsed ticks from the new phase start
- build a temporary scaled palette with decreasing brightness
- upload that palette through `0x2574`

The visible result is a fade back toward black over another approximately `0x80` ticks.

At the end of the routine, it selects all VGA planes and clears VRAM at `0xa0000` so the next startup stage begins from a clean screen state.

## Escape-Key Skip Behavior

`0x2998` checks `0x0964` during all three phases:

- fade-in
- hold
- fade-out

If it sees released scancode `0x01` (`Esc`), it:

- clears the input latch through `0x094c`
- forces a black palette upload if needed
- selects all VGA planes
- clears VRAM
- returns early

That means the startup splashes are explicitly skippable, and the skip path is careful to leave the display in a known black/cleared state.

## Call-Site Meaning

The current executable only calls `0x2998` twice:

1. `chunk 0`
2. `chunk 2`

Those are now firmly understood as:

1. `DDD Dungeon Dweller Designs`
2. `WARNING: This game has been known to cause severe brain damage.`

So `0x2998` is best modeled as the startup full-screen splash presenter, not as a general menu-screen loader.

## Porting Impact

This is a good fidelity checkpoint for the future Windows port.

The faithful version should preserve these properties:

- splash screens use local palettes
- the image is loaded once, then revealed by palette ramp
- there is a visible hold interval at full brightness
- `Esc` can skip the presentation
- each splash returns the display to a black/cleared state before the next stage

We do not need literal VGA plane writes in SDL, but we do want the same presentation rhythm and user-control behavior.

## Bottom Line

`0x2998` is now resolved well enough to stop treating it as a vague chunk loader.

It is a concrete startup splash routine:

- load local-palette full-screen chunk
- upload image
- fade in
- hold
- fade out
- allow `Esc` skip
- clear the screen

That is the exact kind of detail that will make the eventual port feel true to the original rather than merely similar.
