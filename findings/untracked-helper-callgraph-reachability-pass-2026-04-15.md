# Untracked Helper Callgraph Reachability Pass

Date: 2026-04-15

## Summary

This pass adds callgraph-level reachability on top of the direct/indirect static scans.

Main result:

- `0x1765a` remains unreachable from startup/session entry seeds in the current known function-level callgraph model
- neighboring confirmed primitives (`0x175c5`, `0x17613`, `0x17821`) remain normally reachable

## New Owned Artifacts

- [function-callgraph-reachability.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-callgraph-reachability.json)
- [analyze_function_callgraph_reachability.py](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/scripts/analyze_function_callgraph_reachability.py)

## Method

Inputs:

- relocated flat binary
- known function starts from [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)

Process:

1. scan all `E8 rel32` calls
2. map call sites to nearest-lower known function start (practical caller ownership heuristic)
3. keep edges where target is also a known function start
4. run reachability from entry seeds `0x18` and `0x23b`
5. test target reachability for `0x1765a`

## Key Results

From [function-callgraph-reachability.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-callgraph-reachability.json):

- known function starts modeled: `114`
- rel32 calls scanned: `2004`
- resolved function-to-function edges: `523`
- reachable known functions from entry seeds: `103`
- direct rel32 call sites to `0x1765a`: none
- direct rel32 call sites to `0x17660`: none
- reachability flag:
  `target_reachable_from_entry_seeds = false`
- reverse path from entry to target:
  `null`

Sanity references inside the same run:

- `0x175c5` direct rel32 sites: `14`
- `0x17613` direct rel32 sites: `14`
- `0x17821` direct rel32 sites: `1`

## Fidelity Impact

For the `C++23 + SDL3` port, this further reduces risk of over-modeling `0x1765a`:

- keep it documented as a possible internal primitive
- do not make it part of required rendering correctness
- continue preserving verified helper families as canonical behavior

## Scope Note

This is still a static callgraph approximation tied to currently modeled function starts.

It is stronger than direct-caller checks alone, but runtime trace proof is still the final closure step if we want absolute certainty.
