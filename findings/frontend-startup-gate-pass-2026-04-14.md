# Frontend Startup Gate Pass

Date: 2026-04-14

## Summary

This pass corrects an important early-startup assumption.

The key result is:

- the frontend dispatcher has an **early conditional** startup entry in `state 5`
- that early entry happens when either:
  - the game is launched with the `setup` command-line argument
  - `SETUP.DAT` is missing or invalid and defaults must be seeded
- this is **not** the only startup frontend entry; later startup still reaches `0x3830(1)` for the normal main-menu path

That means the original startup/frontend story is conditional in its first state, but still frontend-backed on normal cold boot.

## The `setup` Command-Line Flag

At the top-level runtime entry at `0x00000018`, the code builds and compares the embedded command-line token:

- `setup`

against the incoming argv strings through `0xe600`.

If any argument matches, it sets:

- `ESI = 1`

So there is a real explicit setup-mode launch path, which matches the shipped batch file:

- `SETUP.BAT`
  calls `atet setup`

## What `0x36dc` Contributes

Later in the same startup path, `0x36dc`:

- loads `SETUP.DAT` if it exists and has the correct `AciD` signature
- otherwise falls back to `0x358c`
  `seed_default_setup_state`

When that fallback happens, `0x36dc` returns:

- `EAX = 1`

So a missing or invalid setup file sets the same effective gate as explicit setup mode.

## How The Gate Is Combined

The startup path does:

- `or esi, eax`

where:

- `ESI`
  setup-mode flag from argv
- `EAX`
  missing/invalid-setup fallback flag from `0x36dc`

Then it checks:

- `cmp esi, 1`

and only on equality calls:

- `0x3830`

with:

- `EAX = 5`

The dispatcher jump table confirms:

- state `5` -> `0x5a94`
  first-run sound setup

So the first startup frontend state is specifically the sound-setup flow.

## What This Changes

This corrects a softer earlier assumption that the frontend path simply begins at the main menu after the startup splashes.

What we can now say more accurately is:

1. startup preloads the chunk-7 frontend object banks and the chunk-4 title/logo base assets
2. if setup mode is requested or setup must be initialized, startup enters the frontend dispatcher in state `5`
3. that path starts on the sound-setup screen, using the same title/logo/object presentation framework as the rest of the frontend
4. startup then continues into the SFX loads, startup splashes, gameplay-side resource loads, and music start
5. later startup enters the frontend dispatcher again in state `1` for the normal main-menu path

So the title/logo presentation is still a frontend asset path, but the first handler is not always the main menu.

## What Stays True

This does **not** invalidate the earlier frontend work.

The following still hold:

- `0x5f78` preloads the title/logo base from chunk `4`
- `0x3830` owns the shared frontend bootstrap and fade sequencing
- `0x40c0` is the main menu
- `0x5a94` is the first-run sound setup screen
- chunk-7 floating objects are a shared persistent frontend subsystem

The correction is about entry conditions and initial state, not about the later menu/render analysis.

## Porting Impact

For a faithful port, we should preserve the startup gate semantics:

- `atet setup`-style explicit setup mode should enter the sound-setup frontend path
- missing/invalid config should also force that path
- we should also preserve the later normal `state 1` main-menu entry rather than treating normal cold boot as direct gameplay

## Bottom Line

This pass makes the startup model more honest:

- the first startup frontend entry can be conditional
- startup frontend state `5` is the sound-setup flow
- the main menu is still a later cold-start frontend state
