# Startup-Path Predicate Pass

Date: 2026-04-17

## Summary

This pass closes the executable-side startup predicate chain between:

- top-level entry `0x00000018`
- setup load/seed path `0x36dc` / `0x358c`
- the optional early `0x3830(5)` sound-setup detour
- the later backend-init path through `0x668c`
- and the unconditional cold-start main-menu entry through `0x3830(1)`

Main result:

- the pre-main-menu startup path is now bounded to a small, concrete predicate family
- there is no owned evidence for a hidden config-byte startup gate between `0x36dc` and the early `0x3830(5)` detour
- after the early sound-setup choice and backend-init fallback are resolved, the cold-start path into `0x6544` and `0x3830(1)` is deterministic

## New Owned Artifact

- [startup-path-predicate-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/startup-path-predicate-closure.json)

## Direct Static Closure

### 1. Explicit `setup` Arg Is The First Startup Predicate

The top-level startup entry at `0x00000018` scans argv entries through `0x0e600`.

The direct disassembly at `0x00000040..0x0000005b` shows:

- argv pointer walk
- compare against the embedded token `setup`
- `ESI = 1` if any compare succeeds

So the first startup predicate is not vague command-line handling in general.
It is a very specific boolean:

- did any argv entry equal `setup`?

### 2. `0x36dc` Only Adds Missing / Invalid Setup As The Second Startup Predicate Family

The direct `0x36dc` disassembly now closes the loader side cleanly.

At `0x36e2..0x36f9`:

- open `SETUP.DAT`
- if open fails:
  - close path
  - call `0x358c`
  - return `EAX = 1`

At `0x370f..0x3729`:

- read the first `5` bytes
- compare them against the embedded `AciD` signature at `0x18873`
- if the compare fails:
  - close path
  - call `0x358c`
  - return `EAX = 1`

At `0x3743..0x37ef`:

- load the persisted fields
- normalize the current track index modulo `6`
- close the file
- return `EAX = 0`

That means `0x36dc` contributes exactly this startup predicate family:

- missing file
- invalid signature
- otherwise valid load

This is narrower than a generic “config state affects startup” story.

### 3. The Early Sound-Setup Detour Is Only `argv_setup OR setup_seeded`

The branch at `0x15f..0x181` is now simple:

- call `0x36dc`
- `or esi, eax`
- `cmp esi, 1`
- only then call `0x3830(5)`

So the early sound-setup detour is controlled by exactly two boolean sources:

- explicit `setup` argv
- missing/invalid setup fallback from `0x36dc`

No owned evidence currently supports a third hidden setup-byte gate in this branch.

### 4. The Early Sound-Setup Detour Has Only One User-Controlled Predicate Of Its Own

Once startup enters `0x3830(5)`, the remaining startup-relevant branch inside that detour is the sound-setup return choice:

- state `2`
  continue startup
- state `3`
  hard exit to DOS

That matters because it closes another ambiguity:

- early sound setup is not a free-form alternate startup tree
- it is a short startup prelude with one practical user predicate:
  continue or exit

When it returns through state `2`, startup explicitly restores main-menu row `0` at `0x1886f` before continuing.

### 5. Concrete Backend Init Adds One More Predicate, Then The Path Becomes Deterministic

After the early setup detour point, startup loads:

- device from `0x2c6d7`
- rate from `0x2c70f`
- stereo from `0x2c723`
- bit depth from `0x2c6e3`

and calls `0x668c`.

The branch at `0x1a7..0x1d5` is:

- if configured init succeeds:
  continue
- otherwise:
  retry with device index `4` `None`

That is the last meaningful predicate family before the first normal main-menu entry.

After it resolves, startup always does the same thing:

- load the twelve SFX slots
- show splash `0`
- show splash `2`
- load gameplay-side resources
- call `0x6544` with the selected track
- call `0x3830(1)`

So the tail into the real cold-start main menu is deterministic once backend-init fallback is resolved.

## What This Rules Out

This pass is mostly valuable because of what it lets us stop treating as an open branch family.

Current owned evidence does **not** support any additional pre-main-menu startup predicate based on:

- device value itself
- rate value itself
- stereo flag value itself
- bit-depth value itself
- music volume value itself
- SFX volume value itself
- key binding bytes
- high-score records

Those values still matter in other ways:

- backend init inputs
- runtime audio behavior
- gameplay/menu input
- persistence

But they are not currently evidenced as hidden startup-gate selectors before `0x3830(1)`.

## Practical Interpretation

The startup path is now better bounded than it was after the config-schema pass.

Before this pass, we knew which `SETUP.DAT` fields belonged to which semantic bucket.
Now we also know the control-flow shape that consumes those buckets.

The effective startup predicates before the normal main menu are now:

1. explicit `setup` argv or not
2. setup file missing / invalid or valid
3. if state `5` is entered, continue or Exit to DOS
4. configured backend success or fallback to `None`

That is a small enough set that further startup work can now be sharper than broad mutation sweeps.

## Recommended Next Move

The strongest next move is no longer more startup-path hunting.

It is:

- **Step 6: Input-System Branch Lever Pass**

Why:

- the startup predicate family is now tightly bounded
- the current frontier still needs an earlier runtime seam than `0x178B1`
- held-state carry, release-latch semantics, and menu/input routing remain a more plausible path to a different runtime phase than more config/startup mutation

## Bottom Line

Startup is now tighter than “config and command line influence early flow.”

The owned predicate chain is:

- explicit `setup` argv
- missing/invalid setup fallback
- early sound-setup continue vs exit
- configured backend success vs fallback

After that, startup deterministically reaches `0x6544` and then `0x3830(1)`.
