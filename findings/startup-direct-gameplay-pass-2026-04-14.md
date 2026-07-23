# Startup Direct Gameplay Pass

Status: superseded by the later same-day correction in [startup-sequence-correction-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/startup-sequence-correction-pass-2026-04-14.md).

Date: 2026-04-14

## Summary

This note captured an intermediate interpretation while tracing only the early startup gate.

The corrected result is:

- the early conditional `0x3830(5)` call is real
- but startup later performs an unconditional `0x3830(1)` call after splash/resource/music setup
- so normal launch does **not** skip the frontend entirely

## What The Top-Level Entry Actually Does

At the top-level runtime entry:

- the code builds a `setup` command-line flag
- loads or initializes `SETUP.DAT` through `0x36dc`
- ORs those two conditions together
- only calls `0x3830` when that combined gate is `1`

The startup dispatcher call is:

- `eax = 5`
- `call 0x3830`

And we already know:

- dispatcher state `5` is `0x5a94`
- the first-run sound setup screen

So the startup frontend path is specifically the sound-setup path, not a generic main-menu path.

## Why This Was Not The Full Story

Later in the same startup chain, after:

- SFX slot loads
- the DDD splash
- the warning splash
- gameplay/front-end resource loads
- selected music start through `0x6544`

the code does:

- `mov eax, 1`
- `call 0x3830`

That later call is the normal main-menu entry before gameplay begins.

So the earlier "falls through directly into gameplay" reading was incomplete.
What startup really skips, when the gate is false, is only the **early sound-setup detour**.

## The Shipped Docs Support This Reading

The bundled documentation says:

- first run: use `SETUP.BAT`
- afterward: run `ATET.EXE` to play the game

Relevant file:

- [ATET.DOC](/home/bigfnj/projects/@Project-Tetris/Original.Game/ATET.DOC)

That still lines up with the executable-side behavior:

- `SETUP.BAT` / `atet setup`
  -> early sound-setup frontend path
- `ATET.EXE`
  -> normal splash -> main menu -> gameplay launch when config already exists

## What This Means For The Frontend

The useful takeaway is more specific:

- `0x3830(5)` is an optional early setup-mode frontend entry
- `0x3830(1)` is the later normal cold-start main-menu entry

So the frontend is still a true cold-start subsystem of the shipped game.
The correction is about **which frontend state happens first**, not about frontend being absent.

## What Stays True

This does not invalidate the earlier frontend findings.

We still know that:

- `0x5f78` preloads the title/logo backdrop from chunk `4`
- `0x3830` owns frontend bootstrap/fades/dispatch
- `0x40c0` is the main menu
- `0x5a94` is the startup sound-setup screen
- chunk-7 floating objects are the shared frontend decorative system

The correction is about startup sequencing.

## Porting Impact

For a faithful port, the corrected behavioral model is:

- first-run or explicit setup mode can enter sound setup first
- startup still proceeds through splashes and then the main menu
- gameplay begins after the frontend `New Game` path returns

If we later decide to modernize this into a cold-start main menu, that should be treated as an intentional modernization, not as the original behavior.

## Bottom Line

This note is preserved as a useful stepping stone, but its headline conclusion is not the final one.

The corrected startup model is:

- optional early sound setup
- startup splashes
- normal main-menu entry
- gameplay after the frontend handoff
