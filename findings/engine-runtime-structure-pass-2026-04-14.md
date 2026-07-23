# Engine Runtime Structure Pass

Date: 2026-04-14

## Summary

This pass focused on engine structure first rather than another visible subsystem.

The goal was to understand the startup/runtime architecture as a system:

- what gets allocated first
- what service layers are installed
- what DOS-era hooks are real engine policy versus replaceable platform plumbing
- how shutdown/cleanup is coordinated

The biggest result is that several vague helpers are now much more concrete:

- `0x2da0`
  initializes the transient object pool
- `0x3520`
  installs a DOS critical-error handler, not a generic engine hook bundle
- `0x34f0`
  restores that critical-error handler during cleanup
- `0x6948`
  is the cleanup bridge for the configured audio service layer

That gives us a much clearer line between:

- behavior we should preserve in the port
- DOS-specific machinery we can replace cleanly

## Startup Architecture Is Now Clearer

The early runtime entry at `0x00000018` now reads as a disciplined startup chain, not a loose pile of init code.

Current high-level order:

1. seed core page pointers and runtime globals
2. allocate the main working buffers
3. initialize the transient object pool
4. allocate and later fill the random table
5. enter graphics mode
6. install keyboard input handling
7. install the DOS critical-error handler
8. build the sine table
9. load the `ATET.DAT` offset table
10. load frontend resources
11. load or create `SETUP.DAT`
12. configure audio service mode
13. initialize the chosen audio backend, with fallback to `None`
14. load SFX slots and continue into the rest of startup

That is a healthy architecture signal:
the game distinguishes runtime scaffolding, asset loading, config loading, and service installation instead of blending them together arbitrarily.

## `0x2da0` Initializes The Transient Object Pool

`0x2da0` was previously just an unnamed startup call.

It now resolves cleanly as the allocator/bootstrap path for the transient runtime object system used by:

- `0x2f78`
- `0x3034`
- `0x2e18`
- `0x2f24`

Concrete structure:

- allocate `0x20020` bytes for the object arena
- allocate `0x4000` bytes for a pointer ring
- zero the pool state counters
- fill the pointer ring with `0x20`-byte-spaced record addresses from the arena
- set the initial free-list head to the arena base

This is strong evidence that the particle/transient subsystem is not using ad hoc allocation.
It uses a preallocated fixed-size pool with recycling.

For the future port, that means we should preserve:

- bounded object count
- cheap allocation/recycle behavior
- deterministic lifetime expiry

We do not need to preserve the exact pointer-ring implementation, but we do want the same practical constraints.

## `0x3520` Is A DOS Critical-Error Handler Installer

This helper is no longer well-described as "install engine handlers once."

It does something much more specific:

1. read the current DOS `INT 24h` vector through `0xebe1`
2. save the old offset/segment pair to `0x2cb4d:0x2cb51`
3. install `CS:0x1781e` as the new handler through `0xeb58`
4. register `0x34f0` on the runtime exit-callback stack through `0xec4c`
5. set the once flag at `0x2cb37`

The installed handler at `0x1781e` is simply:

- `mov al, 3`
- `iret`

That matches a DOS critical-error return policy, not gameplay logic.
So this whole path is best understood as:

- install a predictable DOS disk/device error policy for the game
- guarantee restoration on shutdown

This matters for the port because it tells us what *not* to emulate literally.
On Windows 11 we do not want an interrupt-24 recreation.
What we do want is the higher-level intent:

- controlled failure policy
- consistent cleanup on fatal I/O/device trouble

## `0xec4c` / `0xec15` Form A Small Runtime Exit-Callback Stack

This was a useful structural find while tracing `0x3520` and the audio bootstrap.

`0xec4c` registers callback pointers into a bounded stack.
`0xec15` walks that stack in reverse order and calls each registered function.

That means the game/runtime already has a built-in LIFO cleanup model.
Two important current users are:

- `0x34f0`
  restore DOS critical-error handler
- `0x6948`
  remove the configured audio service layer

This is valuable preservation context because the original program does not just "hope" teardown happens.
It explicitly stacks cleanup responsibilities.

## `0x6948` Is The Audio Service-Layer Cleanup Bridge

`0x68d8` chooses an audio service mode and registers `0x6948` through the exit-callback stack.

`0x6948` then:

- checks `0x2d297`
- if mode `1`, removes the periodic IRQ audio service through `0x6ee9`
- otherwise removes the legacy scheduler through `0x6c1f`
- then tail-calls a shared lower cleanup path

This is an important distinction:

- `0x6948`
  removes the chosen *service layer*
- `0x67cf`
  is still the broader backend/resource shutdown path

So the audio architecture is even more layered than it first looked:

- service-layer bootstrap/teardown
- backend/resource bootstrap/teardown
- music fade controller
- SFX slot registry and playback

## What This Means For The Port

The engine structure now suggests a better preservation boundary.

We should preserve:

- startup ordering
- fixed-size transient object behavior
- keyboard/state install before runtime use
- cleanup ordering and service ownership
- audio mode abstraction

We should not preserve literally:

- DOS `INT 24h` mechanics
- PMODE/W callback internals
- IRQ vector programming
- PIT divisor programming

Those are platform-specific implementation details, not player-visible behavior.

## Bottom Line

This pass improves the decompilation effort in a deeper way than a single visual helper would.

The runtime is now much easier to reason about as a system:

- startup scaffolding is ordered
- transient objects have a dedicated pool
- DOS critical-error policy is isolated
- cleanup is callback-driven
- audio service teardown is layered and explicit

That is exactly the kind of structural clarity we want before beginning a faithful source port.
