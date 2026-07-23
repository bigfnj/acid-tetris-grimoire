# `0x1765A` Reachability Closure Pass

Date: 2026-04-20

## Summary

This pass consolidates the current `0x1765a` evidence into one closure verdict.

Main result:

- `0x1765a` remains behaviorally understandable
- it remains statically unreachable by every currently owned calling form in the shipped relocated flat binary
- runtime no-hit evidence is still **not** final closure by itself unless a validated state-steering lane is first proven

So the best current reading is now specific enough to pause:

- documented internal-or-unused helper candidate
- not a required shipped public rendering entry point
- not worth further decompilation time until a new runtime steering lane or a new caller family appears

## New Owned Artifact

- [1765a-reachability-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/1765a-reachability-closure.json)

This artifact records:

- the helper body and entry-layout facts
- cumulative direct, indirect, and callgraph reachability results
- the runtime breakpoint caution boundary
- the current pause recommendation for decompilation scope

## Key Artifacts Reused

- [untracked-pixel-helper-reachability-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/untracked-pixel-helper-reachability-pass-2026-04-14.md)
- [untracked-helper-indirect-reachability-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/untracked-helper-indirect-reachability-pass-2026-04-15.md)
- [untracked-helper-callgraph-reachability-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/untracked-helper-callgraph-reachability-pass-2026-04-15.md)
- [runtime-wsl-protected-mode-breakpoint-pass-2026-04-16.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/runtime-wsl-protected-mode-breakpoint-pass-2026-04-16.md)
- [runtime-address-probe-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/runtime-address-probe-pass-2026-04-15.md)
- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [raw-175c5-178af.asm](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/exports/decompilations/renderer-primitive-pass/raw-175c5-178af.asm)

## Findings

### 1. The Helper Body Is Real, And The Entry Boundary Is Now Stable

The raw helper neighborhood still closes the local structure cleanly:

- `0x17613` returns at `0x17659`
- `0x1765a` begins immediately after that return
- `0x17660` is just an internal instruction inside `0x1765a`

And the helper body remains behaviorally solid:

- compute a working-screen byte in the linear `320x240` buffer at `0x2c727`
- read the prior byte
- write the new byte from `CL`
- mark the corresponding dirty `8x4` cell at `0x1ad97` with the value `3`
- return the previous byte in `AL`

So the behavior label still stands:

- `swap_pixel_and_mark_dirty_untracked`

### 2. The Static Reachability Picture Is Now Overwhelmingly Negative

Cumulative owned static evidence now says the same thing from three different angles.

Direct rel32 sweep:

- `0x175c5`
  - `14` direct rel32 callers
- `0x17613`
  - `14` direct rel32 callers
- `0x17821`
  - `1` direct rel32 caller
- `0x1765a`
  - `0` direct rel32 callers
  - `0` direct rel32 jumps
- `0x17660`
  - `0` direct rel32 callers
  - `0` direct rel32 jumps

Indirect / immediate / trampoline sweep:

- no little-endian dword or word materialization for `0x1765a`
- no absolute-indirect pointer slots resolving to `0x1765a`
- no register-indirect immediate setup resolving to `0x1765a`
- no far-call immediate hits
- no `push imm; ret` trampolines

Known-function callgraph reachability:

- entry seeds:
  - `0x18`
  - `0x23b`
- reachable known functions:
  - `103`
- `0x1765a` reachable from entry seeds:
  - `false`

That combination is now much stronger than any one scan by itself.

### 3. The Helper Still Looks Unlike The Neighboring Confirmed Public Entries

The neighborhood comparison remains useful:

- `0x175c5`
  - confirmed frontend point draw helper
  - bounds checks
  - many callers
- `0x17613`
  - confirmed frontend point clear helper
  - bounds checks
  - many callers
- `0x1765a`
  - no bounds checks
  - no static callers
  - no tracked-restore queue integration

That makes `0x1765a` look less like a normal public subsystem entry and more like one of:

- an internal fast primitive
- cut or unused shipped code
- an indirectly reached helper not yet exposed by a proven runtime lane

### 4. Runtime Work Changed The Caution Boundary, Not The Static Verdict

The runtime side is now better framed than it was on 2026-04-15.

Owned debugger work shows:

- native WSL DOSBox-X can re-enter live protected-mode code with `VRT`
- protected-mode code breakpoints are viable

But the important caution still holds:

- a no-hit on `0x1765a` is only meaningful after steering into a validated state where the neighboring helper family is expected to execute

So runtime work did **not** overturn the static reading.
It only clarified what future dynamic closure would need:

- state steering first
- then meaningful `0x1765a` breakpoint interpretation

### 5. This Branch Is Now Specific Enough To Pause

At this point, continuing to revisit `0x1765a` without a new steering lane or a new caller family would mostly repeat the same conclusion in different words.

What we now already own is enough:

- helper body known
- entry layout known
- direct reachability negative
- indirect reachability negative
- callgraph reachability negative
- runtime caution boundary explicit

That is strong enough to preserve one stable project rule:

- keep `0x1765a` documented, but do not treat it as a required primitive for faithful port correctness

## Practical Porting Impact

For the port:

- preserve the confirmed helper families as canonical:
  - `0x175c5 / 0x17613`
  - `0x17821 / 0x17875`
  - `0x17688 / 0x176df / 0x17700`
  - `0x17719`
- keep `0x1765a` as a documented provisional primitive only
- only promote it if future runtime steering or a new caller proof actually reaches it

That keeps the renderer model faithful without overfitting to a helper that still has no shipped reachability proof.

## Next Strongest Move

Pause the `0x1765a` branch here unless one of these becomes newly useful:

1. a validated runtime steering lane that proves neighboring helper execution and allows a meaningful `0x1765a` breakpoint no-hit or hit
2. a newly recovered caller family or binary variant that actually references `0x1765a`
3. a port implementation task that specifically needs a provisional fallback for untracked single-pixel dirty writes

If decompilation continues now, another unresolved subsystem is likely a stronger use of time.

## Bottom Line

The important closure is:

- `0x1765a` is still a real helper body, but it is now specific enough to treat as a documented non-required primitive and pause
