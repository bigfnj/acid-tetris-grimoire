# Untracked Pixel Helper Reachability Pass (2026-04-14)

This pass tightened the remaining low-level question around `0x1765a` with a stricter goal than before: not “what does it do,” but “is it actually reached in the shipped executable?”

## Inputs Used

- Raw helper neighborhood:
  - `Decompilation.Effort/research/ghidra/exports/decompilations/renderer-primitive-pass/raw-175c5-178af.asm`
- Relocated flat binary:
  - `Decompilation.Effort/research/disassembly/pmodew/extracted/flat/ATET.EXE.flat-relocated.bin`
- Existing function map:
  - `Decompilation.Effort/research/ghidra/function-hypotheses.json`

## What Tightened

### 1. Entry layout is clearer now

The raw helper neighborhood shows:

- `0x17613` returns at `0x17659`
- `0x1765a` begins immediately after that return
- `0x17660` is just an interior instruction inside `0x1765a`

So the old “maybe there is a second entry at `0x17660`” concern is weaker now. The better reading is:

- `0x1765a` is the helper body
- `0x17660` is not a separately proven public entry point

### 2. The helper behavior remains strong

The body still reads as:

- compute a working-screen byte in the linear `320x240` buffer at `0x2c727`
- read the old pixel byte
- write the new color from `CL`
- mark the corresponding dirty `8x4` cell at `0x1ad97` with the value `3`
- return the prior byte

And it still differs materially from the tracked particle path:

- no append to the tracked restore queue at `0x1ad8b`
- no local bounds checks

That keeps the behavioral label reasonable:

- `swap_pixel_and_mark_dirty_untracked`

### 3. The shipped reachability evidence is now much stronger

A whole-flat-binary sweep over the relocated flat image found:

- `0x175c5`: `14` direct `rel32` callers
- `0x17613`: `14` direct `rel32` callers
- `0x17821`: `1` direct `rel32` caller
- `0x1765a`: `0` direct `rel32` callers
- `0x17660`: `0` direct `rel32` callers

The same sweep found:

- `0` direct `rel32` jumps to `0x1765a`
- `0` direct `rel32` jumps to `0x17660`

And a literal little-endian dword scan of the whole relocated flat image found:

- `0` hits for `0x1765a`
- `0` hits for `0x17660`

Those dword results are not decisive on their own, because the neighboring helpers also do not appear as absolute address constants, but they are still consistent with the stronger direct-call result.

## Best Current Reading

This is now the cleanest wording:

- behavior known
- entry layout clearer
- shipped direct reachability still unproven

So `0x1765a` should remain:

- real helper body
- medium-confidence behavioral label
- provisional subsystem ownership

It is currently better modeled as one of:

- an internal fast primitive not reached by ordinary direct calls in the shipped build
- an indirectly reached helper we have not yet proven
- dead or cut shipped code

## Port Guidance

For the future `C++23 + SDL3` port, this means:

- preserve the confirmed helper families first:
  - frontend object pixel helpers `0x175c5 / 0x17613`
  - tracked transient helpers `0x17821 / 0x17875`
  - gameplay block blitters `0x17688 / 0x176df / 0x17700`
- do **not** promote `0x1765a` into a required top-level rendering primitive yet
- if later evidence proves a caller, then it can be elevated cleanly without revising the confirmed layering model

## Output Artifact

- `Decompilation.Effort/research/ghidra/untracked-pixel-helper-reachability.json`

## Update (2026-04-15)

An extended indirect/static sweep is now also available:

- [untracked-helper-indirect-reachability-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/untracked-helper-indirect-reachability-pass-2026-04-15.md)

That follow-up scan adds negative checks for:

- absolute-indirect pointer-slot calls/jumps (`FF 15` / `FF 25`)
- register-indirect call setup heuristics
- far-call immediates
- push-immediate trampolines

And still finds no static route to `0x1765a` in the shipped relocated flat binary.

A second follow-up now adds known-function callgraph reachability from startup/session entry seeds:

- [untracked-helper-callgraph-reachability-pass-2026-04-15.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/untracked-helper-callgraph-reachability-pass-2026-04-15.md)

That callgraph layer also keeps `0x1765a` unreachable in the current model.
