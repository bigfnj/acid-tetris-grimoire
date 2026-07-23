# Untracked Helper Indirect Reachability Pass

Date: 2026-04-15

## Summary

This pass extends the `0x1765a` question beyond direct `rel32` calls/jumps.

Result:

- no new static reachability evidence was found for `0x1765a`
- the helper still looks behaviorally real, but currently not proven as a shipped public entry

So the prior reading strengthens further:

- likely internal, cut, or otherwise non-normal entry in the shipping build

## New Owned Artifacts

- [untracked-pixel-helper-reachability-extended.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/untracked-pixel-helper-reachability-extended.json)
- [analyze_untracked_helper_indirect_reachability.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/analyze_untracked_helper_indirect_reachability.py)

## Method Extension

Against the relocated flat binary:

- direct `E8/E9` rel32 call/jump target sweep (baseline check)
- absolute-indirect scans:
  - `FF 15 [imm32]` (call through pointer slot)
  - `FF 25 [imm32]` (jump through pointer slot)
  - with dereference of the slot value
- register-indirect scans:
  - `FF D0..D7` / `FF E0..E7`
  - plus conservative nearby immediate-load heuristics
- far-call immediate scan (`0x9A`)
- push-immediate trampoline scan (`push imm32; ret`)
- little-endian dword/word materialization checks

## Key Results

From [untracked-pixel-helper-reachability-extended.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/untracked-pixel-helper-reachability-extended.json):

- `0x1765a` direct rel32 calls: `0`
- `0x1765a` direct rel32 jumps: `0`
- little-endian dword hits for `0x1765a`: `0`
- little-endian word hits for `0x1765a`: `0`
- absolute-indirect op totals: `14` calls, `24` jumps, with `0` slots resolving to `0x1765a`
- register-indirect totals: `4` call-reg sites, `0` with immediate setup to `0x1765a`
- far calls scanned: `44`, with `0` offsets matching `0x1765a`
- push-imm-ret trampolines: `0`

Reference sanity checks still hold in the same scan:

- `0x175c5` direct rel32 callers: `14`
- `0x17613` direct rel32 callers: `14`
- `0x17821` direct rel32 callers: `1`

## Fidelity Impact

For the `C++23 + SDL3` port:

- keep `0x1765a` as a documented but non-required primitive
- do not build gameplay/frontend rendering correctness around needing that helper
- preserve confirmed helper families first (`0x175c5/0x17613`, `0x17821/0x17875`, block blitters)
- if dynamic tracing later proves `0x1765a` reachability, promote it without redesigning the current renderer model

## Bottom Line

`0x1765a` remains behaviorally understandable but statically unreachable by every currently scanned calling form in the shipped flat image.

That reduces risk for the port: we can preserve fidelity without depending on this helper until runtime evidence says otherwise.
