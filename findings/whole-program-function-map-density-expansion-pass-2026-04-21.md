# Whole-Program Function-Map Density Expansion Pass

Date: 2026-04-21

## Summary

This pass advanced the whole-program map by folding the already-evident helper/runtime cluster into the living hypotheses artifact and then recomputing the rel32-call reachability summary.

Main result:

- the executable-side whole-program map is now denser
- most of that gain came from reusable helper/runtime functions, not from newly mysterious gameplay logic

## Files Updated

- [function-hypotheses.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [function-callgraph-reachability.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/function-callgraph-reachability.json)

## Added Hypotheses

This pass added `11` function hypotheses:

- `0x00000970` -> `finalize_runtime_exit_cleanup`
- `0x00002744` -> `draw_centered_text_mode_status_line`
- `0x0000e600` -> `compare_c_string_against_literal`
- `0x0000e6b0` -> `fill_bytes_with_value`
- `0x0000e6e7` -> `runtime_exit_with_code`
- `0x0000eb0c` -> `copy_bytes_with_overlap`
- `0x0000ed1f` -> `close_low_level_file_handle`
- `0x0000ee66` -> `open_low_level_file_handle`
- `0x0000f490` -> `read_low_level_file_bytes`
- `0x0000f5a3` -> `read_structured_file_block`
- `0x0000f787` -> `format_string_into_buffer`

These are mostly:

- memmove or memset style runtime helpers
- low-level file-open, read, and close helpers
- menu-string formatting helpers
- hard-exit helpers

## Quantitative Change

Before this pass, the live callgraph summary recorded:

- known functions: `114`
- resolved function edges: `523`
- unresolved rel32 calls: `1481`

After the density update:

- known functions: `125`
- resolved function edges: `706`
- unresolved rel32 calls: `1298`

So the concrete improvement is:

- `+11` known functions
- `+183` resolved direct-call edges
- `-183` unresolved direct-call sites

## What This Means

The whole-program map is still incomplete.
But this pass changed its shape in a useful way.

The biggest previously unresolved direct-call contributors were not hidden gameplay subsystems.
They were shared helper/runtime functions that many already-understood subsystems were calling.

That matters because it narrows the remaining density work toward:

- runtime-helper families
- low-level service layers
- PMODE/W or DPMI-style vector helpers

not toward:

- unknown core gameplay mechanics

## New Best Density Targets

After this pass, the strongest unresolved example targets now include:

- `0x000084bf`
- `0x0000eb58`
- `0x0000eb83`
- `0x0000ecca`
- `0x0000f915`
- `0x0000e9ee`
- `0x0000ebe1`
- `0x0000b0f4`
- `0x0000ec4c`

The most promising next density seam is therefore:

- the remaining runtime/vector/helper cluster around `0xeb58`, `0xeb83`, `0xebe1`, and `0xec4c`

## Port Implication

This pass does not materially change subsystem port readiness.
Its value is:

- denser whole-program bookkeeping
- fewer misleading unresolved-call counts
- cleaner separation between real gameplay uncertainty and plain helper-map sparsity

## Next Ordered Step

If whole-program density work continues later, prioritize:

1. the remaining vector/runtime helper cluster
2. stdio or runtime helper families still visible through the unresolved call examples
3. only then any genuinely behavior-relevant higher-level gaps
