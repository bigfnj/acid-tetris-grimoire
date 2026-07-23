# Particle Ramp Semantics Pass

Date: 2026-04-14

## Summary

This pass resolves an important ambiguity in the transient particle/object system:

- the chunk-3 ramp table is not keyed by high-level gameplay event IDs
- it is keyed by sampled on-screen palette indices

That is a useful preservation result because it tells us the particle system is fundamentally color-driven. The helpers choose motion and lifetime; the row pixels themselves choose the color ramp.

## The Shared Particle Argument Model

Both particle allocators:

- `0x2f78` `queue_particle_polar`
- `0x3034` `queue_particle_linear`

use the same two stack arguments:

1. lifetime divisor
2. effect byte

Executable proof from `0x2f78`:

- stores `[object + 0x18] = 0x400 / lifetime`
- stores `[object + 0x1c] = effect_byte << 2`

Then `0x2f24` uses:

- `([object + 0x14] >> 8)` as the stage index `0..3`
- `[object + 0x1c]` as the effect base
- `0x2c6a3[effect_base + stage]` as the plotted palette index

That means the raw `0x400` block in `chunk 3` is a true:

- `256 x 4` four-stage color-ramp table

## The Important Caller-Side Discovery

Every currently known caller to `0x2f78` or `0x3034` pushes the sampled live-screen byte as the effect byte.

That is true for:

- all ten line-clear debris helpers at `0x14b4 .. 0x1c40`
- the game-over dissolve worker at `0x1edc`

In other words:

- the particle system does not say "use line-clear effect ID 7"
- it says "take the row pixel color already on screen and run that palette index through a four-step decay ramp"

So the visual identity of a debris pixel comes from:

1. the source pixel color
2. the chunk-3 ramp table
3. the helper's motion pattern

## What The Per-Level Helpers Actually Control

This changes the interpretation of the ten line-clear leaf helpers.

They are still level-themed, but the theme is primarily:

- motion pattern
- directionality
- target position
- lifetime

not:

- a separate per-level color bank

Color is shared across the helpers through the sampled row pixels and the chunk-3 ramps.

## Lifetime Is Also Concrete Now

The first stack argument is a real duration control.

Because the allocators store:

- `0x400 / lifetime`

and the updater retires particles once their age reaches `0x400`,

the pushed values map directly to practical lifetimes.

Common values now visible in the helper bank:

- `0x30` -> `48` update steps
- `0x40` -> `64` update steps
- random `0x10 .. 0x4f` in the burst-style helpers

So the helpers are choosing not just direction, but also how long the debris remains active.

## Game-Over Uses The Same Color-Driven Model

The staged game-over dissolve path extends the same system rather than using a separate effect model.

`0x1f8c`:

- advances the game-over row index
- calls `0x1edc` once per step

`0x1edc`:

- walks every other pixel across the active gameplay row
- skips zero pixels
- samples the live screen byte
- feeds that source color into `0x2f78`
- clears the row afterward

So game-over debris and line-clear debris share the same underlying color-ramp system.

## Table-Level Observations

The resolved ramp table has:

- `256` total ramps
- `68` unique four-byte ramp patterns
- `189` ramps that are all zero

Practical reading:

- many palette indices never produce visible transient particles
- the non-zero ramp entries are the authored subset used for visible debris

That fits the gameplay captures well: only certain bright gameplay colors visibly break into particles.

## Porting Impact

For the future C++23/SDL3 port, the correct preservation model is:

- give each transient particle an initial source palette index
- map that source index through the four-stage chunk-3 ramp table over its life
- let the line-clear or game-over helper choose motion profile and lifetime
- do not replace this with event-colored particles unless we intentionally modernize later

## Bottom Line

This pass closes the last big semantic gap in the chunk-3 ramp table.

The particle system is now best described as:

- motion chosen by helper
- color chosen by sampled source pixel
- fade chosen by the shared four-stage chunk-3 ramp table

That is a strong preservation-friendly result and a much better target for the eventual source port than an event-ID-based approximation.
