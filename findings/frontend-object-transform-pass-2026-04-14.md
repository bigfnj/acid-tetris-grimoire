# Frontend Object Transform Pass

Date: 2026-04-14

## Summary

This pass tightens the menu-object system around `0x39c4`.

The important result is that the floating frontend objects are now better understood as a fixed-step projected object system with:

- banked authored shapes from chunk `7`
- two continuously advancing rotation angles
- three harmonic origin-drift channels
- perspective projection and depth-derived shading
- a fixed-step bank-cycle and morph schedule

That is a stronger and more useful description than simply calling the effect a "pointfield."

## Why The Objects Feel Like They Float

The user-facing menu effect looks like a floating smiley or floating tetrimino because the executable is doing more than projecting a static authored bank.

Inside `0x39c4`, the active bank is transformed by a live state bundle rooted at:

- `0x2d22b`
- `0x2d227`
- `0x2d213`
- `0x2d23f`
- `0x2d243`
- `0x2d23b`
- `0x2d237`
- `0x2d247`

That state evolves every logical step, not just every presented frame.

So the menu objects are not static outlines.
They are always being:

- rotated
- drifted
- reprojected
- reshaded

## Transform Model

Current best model:

1. start from one authored record triple:
   - `+0x00`
   - `+0x04`
   - `+0x08`
2. rotate the first two components with one sine/cosine angle
3. combine that intermediate depth with the third component using a second sine/cosine angle
4. add evolving origin offsets
5. perspective-divide the result
6. center to `320x240`
7. derive a 16-step depth shade and remap it into the `0xe0..0xef` frontend palette band

So the frontend object bank is effectively a small projected pseudo-3D system.

## Evolving State

The state update after the record walk is what gives the objects their floating feel.

Current high-confidence roles:

- `0x2d237`
  primary rotation angle, advances by `+5` per logical step
- `0x2d247`
  secondary rotation angle, advances by `+2` per logical step
- `0x2d23f`
  x-drift phase, advanced by `(sin(primary_angle) >> 14) + 3`
- `0x2d243`
  y-drift phase, advanced by `(sin(secondary_angle) >> 14) + 2`
- `0x2d23b`
  z-drift phase, advanced by `(sin(secondary_angle) >> 14) + 4`
- `0x2d22b`
  origin-x accumulator, updated by `sin(x_phase) << 3`
- `0x2d227`
  origin-y accumulator, updated by `sin(y_phase) << 2`
- `0x2d213`
  origin-z accumulator, updated by `sin(z_phase) << 4`

This is why the menu objects appear to drift and wobble rather than just spin in place.

## Projection And Shading

After transform and origin drift, the system uses a perspective divisor based on transformed depth:

- `(depth + 0x14000000) >> 9`

Then:

- x is centered by `+160`
- y is centered by `+120`

The shade is derived from depth, clamped to `0..15`, and written back as:

- `0xef - shade`

That explains why the menu objects show up as dotted silhouettes with depth-dependent brightness rather than flat monochrome outlines.

## Fixed-Step Cycle Timing

The chunk-7 system is not free-running against render time.

`0x39c4` uses a cycle counter at:

- `0x1990b`

Current cycle model:

- `0x000 .. 0x258`
  steady projection of the current bank
- `0x259 .. 0x2d8`
  morph interval driven by `0x3cdc(progress)`
- `>= 0x2d8`
  commit next bank as current bank and reset the counter

Important detail:

- the steady interval is `0x258` logical steps
- the morph interval is `0x80` logical steps

That is a meaningful distinction for the port.

## Why Logical Steps Matter More Than Frames

The menu loops do not call `0x39c4` exactly once per present.
They use the same catch-up mechanism as gameplay through:

- `0x184cb`
- `0x24d0`

So if the system is behind, the frontend may advance multiple logical object steps before the next presented frame.

This means the faithful port should preserve the object system as:

- fixed-step state updates
- render after zero or more updates

not:

- one state update per present

That will matter for motion feel and bank-morph timing.

## Bank Content And Motion Together

With the newer bank-identification work, the complete model is now:

- chunk `7` contains authored tetromino-like and smiley-like object banks
- `0x5ef4` selects one bank and seeds the object-state variables
- `0x39c4` makes that authored bank float, rotate, and drift
- `0x3c70` / `0x3cdc` morph it toward another authored bank

So the floating menu smiley is not a separate sprite system layered over the frontend.
It is one authored chunk-7 bank being carried through this transform pipeline.

## Porting Impact

For the future Windows port, this subsystem should preserve:

- authored bank identities
- fixed-step cycle timing
- two-angle rotation feel
- harmonic drift feel
- perspective centering and depth-derived shade band
- steady interval followed by morph interval

We do not need the original integer math literally, but we do want the same motion character and timing semantics.

## Bottom Line

This pass makes the frontend object system much more actionable.

The menu smiley and tetrimino are now best modeled as:

- authored chunk-7 object banks
- animated by a fixed-step transform/projection state machine

That is exactly the level of understanding we want before writing a faithful port.
