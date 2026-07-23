# Line-Clear Style Source Closure Pass

Date: 2026-04-20

## Summary

This pass closes the old open question about where the line-clear "theme" actually comes from.

Current best closure:

- level does choose among `10` line-clear helper variants through `current_level % 10`
- but that level choice selects motion pattern, directionality, and lifetime
- debris color does **not** come from a hidden per-level art bank
- debris color comes from the cleared row's sampled on-screen palette indices, passed through the shared chunk-`3` four-stage ramp table

So the line-clear look is now best modeled as:

- helper-selected motion
- row-pixel-selected color
- shared chunk-`3` fade behavior

not:

- per-level color art loaded from a separate gameplay theme bank

## Why This Pass Was Needed

An older open question in [line-clear-theme-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-theme-pass-2026-04-13.md) asked:

- which asset bank provides the per-level visual identity for the row-clear effects

Later passes answered most of that, but the answer was scattered:

- [startup-and-chunk3-resolution-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-and-chunk3-resolution-pass-2026-04-14.md)
- [particle-ramp-semantics-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/particle-ramp-semantics-pass-2026-04-14.md)
- [line-clear-particle-overlap-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-particle-overlap-pass-2026-04-15.md)

This pass turns that scattered answer into an explicit closure.

## Consolidated Evidence

### 1. Level Selects The Helper Family

The dispatcher `0x1414` still does:

- `current_level % 10`
- jump through the ten-entry helper table at `0x13ec`

So level is genuinely part of the row-clear presentation path.

But what it selects is the helper body:

- `0x14b4`
- `0x158c`
- `0x1694`
- `0x1770`
- `0x1830`
- `0x192c`
- `0x19dc`
- `0x1a9c`
- `0x1b54`
- `0x1c40`

Those helpers differ in:

- spray direction
- angle family
- target-point behavior
- lifetime choices
- overall motion pattern

### 2. Color Comes From Sampled Row Pixels

The later particle semantics pass tightened the shared allocator argument model:

- `0x2f78` stores `effect_byte << 2` into `[object + 0x1c]`
- `0x3034` uses the same convention
- current callers pass the sampled live-screen byte as that effect byte

That means line-clear debris color is not chosen as:

- "level 3 effect color bank"

It is chosen as:

- "whatever palette index was already on the cleared row at that pixel"

### 3. Chunk `3` Supplies The Shared Fade Ramps

`0x2f24` then combines:

- stage index from object age
- effect base from the sampled source pixel

and reads:

- `0x2c6a3[effect_base + stage]`

That table is already resolved as the raw `0x400`-byte chunk-`3` block:

- `256` ramps
- `4` stages each

with the preserved extraction at:

- [lookup-table-0x400.ramps.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/graphics/verified/auxiliary/atet-dat.chunk-0003.off-000125CB.len-00000641.lookup-table-0x400.ramps.json)

So the shared color-fade asset is chunk `3`, not a per-level theme bank.

### 4. `chunk 7` Is Not The Source

This also resolves the old "if not chunk `7`, then what?" ambiguity.

Because:

- [chunk7-usage-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/chunk7-usage-pass-2026-04-13.md)
- [frontend-object-bank-identification-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-object-bank-identification-pass-2026-04-14.md)

already closed chunk `7` as a frontend floating-object bank family, not a gameplay line-clear styling source.

### 5. Capture Support Is Useful But No Longer Required For This Question

The owned gameplay capture still supports:

- dense colored speckles around gameplay-side effects

That remains behaviorally consistent with sampled-pixel debris.

But importantly, the asset-source closure no longer depends on matching each of the ten motion helpers to capture footage.

The remaining capture-side gap is:

- which exact helper motion profile is visible in which owned gameplay moment

That is a motion-correlation question, not an asset-source question.

## Closure

The old open question can now be closed.

Closed answer:

- there is no owned evidence for a separate per-level line-clear color art bank
- level-specific styling is primarily the helper-selected motion family
- line-clear debris colors come from the cleared row's sampled palette indices
- the shared four-stage color decay comes from chunk `3`

So the per-level visual identity is assembled from:

1. current level -> helper motion selection
2. current row pixels -> source color IDs
3. chunk `3` ramps -> transient fade path

## Port Implication

A faithful first-pass port should preserve line-clear styling as:

- level picks motion helper family
- source board pixels pick debris color
- shared chunk-`3` ramps age those colors over particle lifetime

It should **not** invent:

- a per-level debris color palette swap
- a hidden gameplay theme sprite bank for line-clear particles
- event-colored debris unrelated to the cleared row's actual pixels

## Artifact

This pass adds:

- [line-clear-style-source-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/line-clear-style-source-closure.json)

## What I Now Treat As Resolved

- the line-clear style source question is closed strongly enough for preservation work
- chunk `3` is the shared color-ramp asset behind line-clear and top-out debris
- level-specific line-clear behavior now means motion-profile selection, not color-bank selection

## Next Ordered Step

- pause the line-clear style-source branch unless one of these becomes newly useful:
  - a capture-correlation pass that maps individual helpers to owned gameplay footage
  - a new build or asset set that contradicts the shared chunk-`3` ramp model
  - a port-implementation task that needs the ten helper families named more concretely for code structure
