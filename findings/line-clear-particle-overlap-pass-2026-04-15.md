# Line-Clear Particle Overlap Pass - 2026-04-15

This pass tightened one specific fidelity seam:

- how line-clear debris overlaps with row-collapse visuals across the same frame and across subsequent collapse frames

## New Artifact

- [line-clear-particle-overlap.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/line-clear-particle-overlap.json)

## Main Result

Line-clear debris is a high-density transient overlay, not a tiny cosmetic effect.

The best current reading is:

- one cleared row queues up to `640` debris objects (`8` scanlines x `80` pixels)
- a four-line clear can attempt up to `2560` queue inserts in one gameplay step
- queue allocation is hard-capped at `0x1000` (`4096`) active objects
- overflow attempts are dropped silently

So heavy line-clear frames can legitimately produce dense debris, and can also thin under load when the object pool is already busy.

That is an original-behavior detail worth preserving.

## Evidence Tightening

### 1. Emission density is explicit in the helper bodies

Across all ten level-themed helpers:

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

the shared structure is:

- iterate `80` pixels per scanline (`cmp ..., 0x50`)
- iterate `8` scanlines per board row (`cmp ..., 0x8`)
- queue via `0x2f78` or `0x3034`
- clear the sampled scanline via `0x0e6b0`

That yields `640` queue attempts per cleared board row.

### 2. Queue cap behavior is concrete, not inferred

Both queue allocators:

- `0x2f78`
- `0x3034`

begin with:

- `cmp [0x2ca97], 0x1000`
- early return when full

So overflow attempts are discarded directly, with no alternate queue path.

### 3. Overlap timing across collapse frames is now cleaner

`0x09c8` line-clear phase behavior:

- detect/record clears and call `0x1414` on clear frames
- call `0x1d04(1)` when pending clears exist
- while `0x184df > 0`, short-circuit into `0x1d04` and return early from normal movement/spawn logic

But the outer gameplay frame still does:

- `0x2e18` before `0x09c8`
- `0x2f24` after `0x09c8`

So existing transient particles continue to update and redraw during collapse frames.

This gives a strong overlap rule:

- debris from the clear step can continue to overlay while `0x1d04` / `0x1e34` are shifting board pixels
- tracked cleanup through `0x17875` still prevents permanent corruption

## Practical Fidelity Rule

The port should preserve line-clear debris as:

- dense
- queue-capped
- collapse-overlapping
- frame-restored

and should avoid simplifying it into:

- "clear rows instantly and maybe spawn a few particles"

because that would lose a real part of the original visual rhythm.

## Updated Files

- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [transition-preservation-spec.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/specs/transition-preservation-spec.md)

## Next 10 Strongest Moves

1. Tighten alert-tile lifetime edge cases under simultaneous line-clear debris load.
2. Tighten spawn-collision and first visible failed-spawn presentation one more step.
3. Keep searching for indirect or computed dirty-map writes that could refine mark-`3` propagation.
4. Revisit high-score footer-exit conceal rhythm against captures now that name-entry flow is sharper.
5. Keep converting gameplay layering findings into direct renderer constraints for the C++23/SDL3 port.
6. Build a dedicated renderer contract note that groups page ring, dirty propagation, transient cleanup, and overlap rules in one place.
7. Refresh the root session log once this next cluster lands so restart context stays crisp.
8. If we get a line-clear-heavy gameplay capture, correlate density and thinning behavior against this new queue-cap model.
9. Keep `0x1765a` explicitly provisional unless new caller evidence appears.
10. Continue preserving confidence boundaries so speculative details do not leak into implementation rules.
