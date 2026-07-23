# GUS And Module Stream Density Pass

Date: 2026-05-15

## Summary

This pass closes the two strongest direct-call density candidates from the prior handoff:

- the `0x0000b329` family is Gravis Ultrasound GF1 register, DRAM, voice-count, and IRQ helper code
- the `0x00008497` / `0x000084bf` / `0x000084e3` trio is a bounded MikMod tracker-command stream reader/consumer family

The result is audio/library infrastructure clarification, not new gameplay or frontend behavior.

## Files Updated

- [function-hypotheses.json](/home/bigfnj/projects/atet/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [function-callgraph-reachability.json](/home/bigfnj/projects/atet/Decompilation.Effort/research/ghidra/function-callgraph-reachability.json)

## Added Hypotheses

This pass added `16` function hypotheses:

- `0x00008497` -> `begin_module_command_stream`
- `0x000084bf` -> `read_module_command_stream_byte`
- `0x000084e3` -> `skip_module_command_stream_operands`
- `0x0000b329` -> `write_gus_register_byte`
- `0x0000b353` -> `write_gus_register_word`
- `0x0000b383` -> `read_gus_register_byte`
- `0x0000b3ad` -> `read_gus_register_word`
- `0x0000b3d8` -> `delay_gus_io_settle`
- `0x0000b459` -> `read_gus_dram_byte`
- `0x0000b498` -> `write_gus_dram_byte`
- `0x0000b4d9` -> `write_gus_dram_bytes_same_page`
- `0x0000b565` -> `write_gus_dram_block`
- `0x0000b5c2` -> `read_gus_dram_dword`
- `0x0000b5fc` -> `write_gus_dram_dword`
- `0x0000b669` -> `initialize_gus_voice_count`
- `0x0000bdcd` -> `handle_gus_irq`

## GUS Helper Evidence

`0x0000b329` and nearby helpers address the Gravis Ultrasound GF1 register interface:

- register selection writes use GUS base + `0x103`
- byte values pass through GUS base + `0x105`
- word values pass through GUS base + `0x104`
- the selected GF1 register is cached at `0x43432`
- DRAM byte access uses GF1 register `0x43` / `0x44` plus GUS base + `0x107`
- the block writer wraps across 64 KiB GUS DRAM pages
- the IRQ handler restores the cached selected register before `iret`

The embedded setup strings around the same driver region include `ULTRASND`, a parsed `%hx,%hd,%hd,%hd,%hd` environment shape, and the shipped "Couldn't detect gus" error path.

## Module Stream Evidence

`0x00008497` initializes a bounded command stream:

- it stores the caller stream pointer at `0x41773`
- it stores the active cursor at `0x4176f`
- it derives the stream end at `0x4176b` from the low five bits of the first byte

`0x000084bf` reads the stream:

- it compares the active cursor against the end pointer
- it returns the next byte when still in range
- it returns `0` at end of stream

`0x000084e3` consumes operands:

- it maps command bytes through the operand-count table at `0x199db`
- it repeatedly calls `0x000084bf` until the command's operand count is exhausted

The two larger parser regions at `0x000092bf` and `0x00009542` call this stream family and then dispatch to music/audio helpers such as `0x9241`, `0x91cc`, `0x8ba9`, `0x8cfd`, `0x8d52`, `0x8ef5`, `0x8f75`, `0x8ff5`, `0x906f`, `0x9194`, `0x91b3`, `0x8dc8`, `0x8e27`, and `0x8e89`.

## Quantitative Change

Before this pass, after the runtime/vector helper pass, the live callgraph summary recorded:

- known functions: `130`
- resolved function edges: `753`
- unresolved rel32 calls: `1251`

After the GUS family alone:

- known functions: `143`
- resolved function edges: `896`
- unresolved rel32 calls: `1108`

After the module-command stream family:

- known functions: `146`
- resolved function edges: `932`
- unresolved rel32 calls: `1072`

Concrete improvement for this pass:

- `+16` known functions
- `+179` resolved direct-call edges
- `-179` unresolved direct-call sites

## Current Next Density Shape

The prior top candidates, `0x0000b329` and `0x000084bf`, are now named and should no longer drive the next density pass.

The regenerated unresolved-call sample is now concentrated around:

- `0x00006ee9` caller-owned audio/module internals
- `0x000084e3` caller-owned module parser internals
- repeated targets such as `0x0000f915`, `0x0000ecca`, `0x00007172`, `0x0000e9ee`, and `0x0000b0f4`

The next density pass should inspect the caller families before promoting any of those targets. The strongest practical split is either an audio-module-parser expansion around `0x000092bf` / `0x00009542` or an audio-service expansion around `0x00006ee9` / `0x0000668c`.

## Port Implication

No gameplay behavior spec change is needed.

For a modern port:

- preserve the module playback behavior, not GUS-specific I/O mechanics
- treat GUS register/DRAM/IRQ helpers as backend driver implementation detail
- keep the bounded tracker-command stream behavior as a useful clue for the music playback path
- do not spend modernization effort recreating DOS interrupt or GF1 register plumbing unless the port intentionally includes a hardware-faithful backend

## Verification

- Parsed `function-hypotheses.json` after the update.
- Regenerated `function-callgraph-reachability.json` with `analyze_function_callgraph_reachability.py`.
- Parsed the regenerated callgraph JSON.
