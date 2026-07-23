# Chunk 7 Render Correlation Pass

Date: 2026-04-13

## Summary

This pass turned the current chunk-7 executable model into concrete rendered artifacts.

The new renderer is:

- [render_chunk7_pointfield.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/render_chunk7_pointfield.py)

The generated outputs are indexed in:

- [manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield/manifest.json)

Rendered PNGs live under:

- [chunk7-pointfield](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/chunk7-pointfield)

The renderer mirrors:

- the chunk-7 source-bank interpretation
- the `0x5ef4` initial animation state
- the `0x39c4` projection math
- the `0x175c5` "draw only onto zero-valued pixels" behavior

## Important New Finding: Bank 0 Is The Default First Transition Target

`0x5ef4` initializes:

- current bank = random `0..7`
- next bank = `0`

Then `0x39c4` only chooses a fresh random next bank at the `0x258` threshold if `next bank == current bank`.

That means:

- if the initial random bank is non-zero
- the first long transition does **not** pick a new random target
- it morphs toward bank `0`

So bank `0` is not just another bank in the set.

It is the default first morph target under the startup state.

## Initial Frame Geometry Splits Into Two Families

The frame-0 blank renders show a strong family split.

### Banks 0 Through 4

These are more compact and collision-heavy:

- draw counts range from `609` to `791`
- all `1024` projected points remain in bounds
- collisions come almost entirely from point-on-point overlap
- the projected region stays in a compact lower-band box:
  - roughly `x = 82..291`
  - roughly `y = 132..185`

### Banks 5 Through 7

These are much wider and substantially less self-overlapping:

- draw counts range from `980` to `1012`
- again all `1024` points remain in bounds
- only `12` to `44` points are blocked by prior plotted points
- the projected region expands to:
  - `x = 68..310`
  - `y = 100..220`

This lines up cleanly with the earlier bank-analysis result that banks `5`, `6`, and `7` form a tight family and carry an all-zero `+0x08` source field.

## Title Background Does Not Block The Initial Pointfield

The title-overlay renders produced exactly the same draw counts and bounding boxes as the blank renders for all eight banks.

That means, at least for the initial frame:

- the pointfield lands entirely in zero-valued regions of the title base screen
- the title art itself does not suppress any of the projected points

This is consistent with the executable-side draw helper `0x175c5`, which only plots into pixels that are currently zero.

## Mid-Morph Renders Support The Bank-0 Anchor Theory

Sample mid-morph renders at progress `64` were generated for banks `1..7 -> 0`.

Those samples show:

- banks `5`, `6`, and `7` contract inward substantially during the morph
- their mid-morph bounds become approximately:
  - `x = 75..268`
  - `y = 116..191`
- that is much closer to the compact `0..4` family than to their wide initial state

This is exactly what we would expect if:

- bank `0` acts as the default anchor target for the first morph
- the wide `5/6/7` family transitions toward a denser, more compact shape

## Practical Decompilation Impact

This gives us a much better source-port model for the frontend decoration:

- preserve the authored source banks as data
- preserve the startup state seeded by `0x5ef4`
- preserve bank `0` as the default initial morph target unless later code changes that state
- preserve the "draw only on zero-valued pixels" rule

It also means the earlier bitmap-panel interpretation of chunk `7` is now not just weak, but actively contradicted by rendered behavior.

## Recommended Next Move

The best next move is to correlate these bank-family renders against the captured menu and credits screens to see whether the wider `5/6/7` family or the denser `0..4` family better matches the actual frontend look in motion.
