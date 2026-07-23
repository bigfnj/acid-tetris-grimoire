# Frontend Title Base Loader Pass

Date: 2026-04-14

## Summary

After tightening the engine structure, this pass moved directly into visible behavior by resolving `0x5f78`.

The important result is that `0x5f78` prepares the frontend title/logo base screen during startup.
It does not present the title screen yet, but it loads the exact visual base that the frontend later reveals and animates.

## What `0x5f78` Loads

`0x5f78` performs a straightforward startup asset load:

1. open `chunk 4`
2. read its `0x300`-byte palette into `0x2cdd3`
3. read its compressed payload into the chunk scratch buffer
4. decompress the image through `0x2880` into `0x2c6af`

That matches the already verified role of `chunk 4` as the ACiD Tetris title/logo screen.

Important detail:

- the helper does **not** upload the image to VGA directly
- it prepares the title base in an off-screen/front-end working buffer

So the later frontend transition logic is revealing a preloaded base screen, not loading it on demand each frame.

## It Also Resets Initial Menu State

After loading the title screen base, `0x5f78`:

- clears the shared dirty-region map through `0x2d88`
- stores `0` to `0x1886f`

`0x1886f` is already tied to the main-menu selected row.
So this startup helper also establishes the initial frontend title/menu baseline:

- the title/logo background is ready
- the selected main-menu row starts on the first later main-menu entry

That explains why the frontend can land in a consistent main-menu state later even though startup may first route through sound setup and later returns preserve other row selections.

## Why This Matters

This is a small but important fidelity detail.

The original frontend startup is not:

- load title screen on first menu loop
- guess a default selected row ad hoc

It is:

- preload the title/logo base screen
- clear stale dirty state
- seed the selected row explicitly
- then hand off to the later frontend presentation paths

That sequencing is part of why the original presentation feels deliberate.

## Porting Impact

For the future Windows port, we should preserve this startup rhythm conceptually:

- prepare the title/logo base asset before the menu loop begins
- initialize frontend dirty/present state before the first frontend reveal
- seed the initial selected row explicitly to the first menu item

That keeps the title/logo-backed frontend state deterministic and true to the original.

## Bottom Line

`0x5f78` is now resolved as:

- load the frontend title/logo base screen from `chunk 4`
- store its palette separately
- clear frontend dirty state
- seed the initial main-menu selection for later main-menu entry

That is a useful visible-behavior checkpoint, and it fits cleanly with the engine-structure work that came just before it.
