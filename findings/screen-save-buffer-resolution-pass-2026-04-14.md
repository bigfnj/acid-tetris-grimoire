# Screen Save Buffer Resolution Pass

Date: 2026-04-14

## Summary

This pass resolves the role of `0x2c69f` cleanly.

The important result is:

- `0x2c69f` is the saved gameplay-screen buffer
- the key was the copy direction of `0xeb0c`
- `0xeb0c` copies from `EDX` source to `EAX` destination

That means the earlier same-day "blank transition buffer" correction was wrong.

## Why The Earlier Reading Broke

The earlier pass noticed two true things:

- `0x2c69f` is allocated and zero-filled at startup
- it is only directly referenced a small number of times

But it interpreted the dispatcher's `0xeb0c` calls backward.

The decisive behavior of `0xeb0c` is already visible in multiple confirmed callers:

- RLE decode direct-copy path in `0x2880`
  destination in `EAX`, source in `EDX`
- game-over overlay restore in `0x23ac`
  destination working screen in `EAX`, source saved overlay in `EDX`

So at `0x3830`:

- `mov edx, [0x2c727]`
- `mov eax, [0x2c69f]`
- `call 0xeb0c`

means:

- save the current working screen into `0x2c69f`

not:

- restore `0x2c69f` into the working screen

## Corrected Buffer Roles

Current best model:

- `0x2c727`
  live linear `320x240` working screen
- `0x2c6af`
  preloaded title/menu base screen from chunk `4`
- `0x2c69f`
  saved gameplay-screen snapshot used across frontend entry/exit

This fits both startup and `Return to Game` much better.

## What `0x3830` Really Does At Entry

The top of `0x3830` now reads as:

1. copy the current working screen from `0x2c727` into the save buffer at `0x2c69f`
2. run the simple palette fade-out `0x64f0`
3. copy the title/menu base screen from `0x2c6af` into the working and visible pages
4. run the chunk-7/title palette fade-in `0x63b8`

So the frontend entry path explicitly preserves the gameplay screen before replacing it with the title/menu presentation.

That is exactly what we would expect from a real `Return to Game` system.

## What The Dispatcher Tail Really Restores

Later in the shared dispatcher tail:

- `mov edx, [0x2c69f]`
- `mov eax, [0x2c727]`
- `call 0xeb0c`

means:

- restore the saved gameplay snapshot from `0x2c69f` back into the working screen

Then the code mirrors that restored working screen into the visible pages.

So for state `2` `Return to Game`, the frontend really does restore the previously saved gameplay image before handing control back to the outer gameplay loop.

## Startup Meaning

This also clarifies startup more nicely.

At cold boot, when `0x3830(1)` is entered after the splash/resource/music phase:

- the current working screen already contains the gameplay-side base image loaded earlier from chunk `1`
- the dispatcher saves that screen into `0x2c69f`
- later, when `New Game` exits through state `4`, the dispatcher restores that saved gameplay-side base into the working screen before `0x05e0` continues the run-start path

So `0x2c69f` becomes:

- startup gameplay-base snapshot on cold boot
- live gameplay snapshot on in-game menu entry

depending on when the frontend is entered

That is a much stronger model than either "blank buffer" or "fixed gameplay base only."

## What This Changes In The Handoff Story

The startup-to-gameplay handoff now reads best as:

1. frontend exit fades out
2. saved gameplay snapshot is restored
3. for `New Game`, `0x05e0` seeds a fresh run on top of that restored gameplay-side base
4. gameplay palette reveal runs through `0x6498`
5. the outer gameplay loop presents the first live gameplay frame

This preserves the earlier useful result:

- the first gameplay-side present still includes at least one live outer-loop step

while replacing the incorrect "blank transition base" assumption with a real save/restore model.

## Bottom Line

The resolved screen-buffer model is:

- `0x2c727`
  live working screen
- `0x2c6af`
  title/menu base
- `0x2c69f`
  saved gameplay snapshot

And the key proof is simple:

- `0xeb0c` copies `EDX -> EAX`
