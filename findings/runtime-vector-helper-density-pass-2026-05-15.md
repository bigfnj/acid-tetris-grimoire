# Runtime Vector Helper Density Pass

Date: 2026-05-15

## Summary

This pass closes the handoff's next whole-program density seam: the runtime/vector helper cluster around `0xeb58`, `0xeb83`, `0xebe1`, and `0xec4c`.

Main result:

- the DOS interrupt-vector helpers are now named in the living function map
- the runtime heap-free helper is now named
- the runtime exit-callback stack is now split into registration and dispatch helpers
- the callgraph reachability summary no longer carries this cluster as unresolved direct-call noise

## Files Updated

- [function-hypotheses.json](/home/bigfnj/projects/atet/Decompilation.Effort/research/ghidra/function-hypotheses.json)
- [function-callgraph-reachability.json](/home/bigfnj/projects/atet/Decompilation.Effort/research/ghidra/function-callgraph-reachability.json)

## Added Hypotheses

This pass added `5` function hypotheses:

- `0x0000eb58` -> `set_dos_interrupt_vector`
- `0x0000eb83` -> `free_runtime_heap_block`
- `0x0000ebe1` -> `get_dos_interrupt_vector`
- `0x0000ec15` -> `run_registered_exit_callbacks`
- `0x0000ec4c` -> `register_runtime_exit_callback`

## Helper Roles

`0x0000eb58` installs interrupt vectors.

- input shape: `AL` carries the interrupt number; `ECX:EDX` carries the vector pointer
- ordinary path: `INT 21h` with `AH = 0x25`
- protected/extender path: when mode byte `0x1a9a9` is in `2..8`, it uses `AX = 0x2504` and moves the requested interrupt number into `CL`
- owned callers use it for keyboard, critical-error, and audio/timer vectors

`0x0000ebe1` reads interrupt vectors.

- input shape: `EAX` carries the interrupt number
- output shape: `EAX = offset`, `EDX = segment`
- ordinary path: `INT 21h` with `AH = 0x35`
- protected/extender path: when mode byte `0x1a9a9` is in `2..8`, it uses `AX = 0x2502` and `CL = interrupt_number`
- owned callers save the prior keyboard, critical-error, and audio/timer vectors before installing project handlers

`0x0000eb83` releases runtime heap blocks.

- input shape: `EAX` carries the block pointer
- it searches the runtime allocation/list root at `0x1a963`
- it delegates the unlink/free operation through `0x10420`
- it updates the accounting/high-water field at `0x1a96b`
- owned callers free keyboard buffers, file-structure auxiliary buffers, and audio-service allocations

`0x0000ec4c` registers shutdown callbacks.

- input shape: `EAX` carries the callback pointer
- callback stack count is at `0x439cf`
- callback array starts at `0x4394f`
- capacity is `0x20`
- it installs `0xec15` into the runtime final-exit hook slot at `0x1a947`
- it returns `0` on success and `-1` when full

`0x0000ec15` dispatches registered shutdown callbacks.

- it reads the old callback count from `0x439cf`
- it writes `0x20` back to that field before dispatch, preventing ordinary registration during shutdown
- it calls entries in reverse order from the `0x4394f` callback array
- this makes the callback stack last-in-first-out

## Quantitative Change

Before this pass, the live callgraph summary recorded:

- known functions: `125`
- resolved function edges: `706`
- unresolved rel32 calls: `1298`

After this pass:

- known functions: `130`
- resolved function edges: `753`
- unresolved rel32 calls: `1251`

Concrete improvement:

- `+5` known functions
- `+47` resolved direct-call edges
- `-47` unresolved direct-call sites

Direct call counts resolved by this cluster:

- `0x0000eb58`: `10`
- `0x0000eb83`: `28`
- `0x0000ebe1`: `5`
- `0x0000ec4c`: `4`
- `0x0000ec15`: `0` direct calls; it is installed as the exit dispatcher by `0xec4c`

## Interpretation

This was bookkeeping density work, not a new gameplay discovery.

The cluster sits in the runtime service layer:

- interrupt vector save/restore and install
- heap allocation cleanup
- shutdown callback registration and LIFO dispatch

That makes prior unresolved calls into this range less interesting as port blockers. They are infrastructure calls that explain how the shipped program protects and restores DOS vectors and runtime-owned allocations.

## New Best Density Targets

After this pass, the strongest remaining direct-call density targets by raw unresolved call count include:

- `0x0000b329`
- `0x000084bf`
- `0x0000714b`
- `0x00007172`
- `0x0000ccce`
- `0x0000b353`
- `0x0000e014`
- `0x0000d4c8`
- `0x0000e9ee`

These should be treated as call-count candidates only until their caller families are inspected.

## Port Implication

No behavior spec change is needed.

For a modern port, this cluster mostly says:

- preserve the cleanup intent, not DOS interrupt-vector mechanics
- model shutdown callbacks and owned resource teardown explicitly in native code
- do not treat calls into this cluster as gameplay or rendering uncertainty

## Verification

- Parsed `function-hypotheses.json` after the update.
- Regenerated `function-callgraph-reachability.json` with `analyze_function_callgraph_reachability.py`.
- Confirmed the target cluster no longer appears in the `unresolved_call_examples` sample.
