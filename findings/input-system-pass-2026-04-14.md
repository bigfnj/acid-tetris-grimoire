# Input System Pass

This pass resolves the keyboard input model and ties it back to both the gameplay bindings and the persisted `SETUP.DAT` state.

## Main Result

The game uses a custom keyboard interrupt hook rather than polling DOS input APIs directly for gameplay and menu navigation.

That input system has two separate outputs:

- a live `0x100`-byte pressed-state table at `0x2c22b`
- a `0x20`-byte latch block at `0x2c227` whose last byte at `+0x1f` holds the most recently released key code

Gameplay reads the live table. Menus read the release latch.

## Hook Install And Removal

The startup path calls `0x8e4`, which:

- saves the previous interrupt `9` vector into `0x2c233:0x2c237`
- allocates the `0x100`-byte live key-state table
- allocates the `0x20`-byte latch block
- installs the game’s own keyboard handler

The teardown path at `0x8a0`:

- restores the previous interrupt `9` vector
- frees both input buffers
- clears the hook-active flag at `0x184d3`

## Keyboard Handler Semantics

The installed keyboard ISR reads raw bytes from port `0x60`.

Behavior:

- `0xE0` sets a prefix flag at `0x184d7` to `0x80`
- the next key code becomes `(scan_code & 0x7f) + prefix_flag`
- unextended keys keep their standard Set-1 values
- E0-prefixed extended keys become `base_code + 0x80`

Examples:

- Left arrow becomes `0xCB`
- Right arrow becomes `0xCD`
- Down arrow becomes `0xD0`
- Up arrow becomes `0xC8`

On key press:

- the ISR sets `key_state[code] = 1` in the table at `0x2c22b`

On key release:

- the ISR shifts the first `31` bytes of the latch block left by one
- stores the released code at `0x2c227 + 0x1f`
- clears `key_state[code] = 0`

That means:

- `0x964` returns the most recently released key code
- `0x94c` clears the whole latch block
- menu navigation is release-driven
- gameplay movement and rotation are press-state-driven

The ISR also special-cases the combined code `0xAA`, resets the E0-prefix state after each completed event, toggles port `0x61`, and sends the PIC EOI through port `0x20`.

## Binding Model

This resolves the binding representation used everywhere else in the executable:

- the five configurable controls are stored as raw scan-style codes
- gameplay reads them as direct indexes into the live key-state table
- keyboard setup writes the released code returned by `0x964` directly back into the binding bytes

So the modern port should preserve this model logically:

- one table of current pressed states
- one event/latch path for menu-style discrete actions

## Default Controls

`0x358c` seeds the compiled default bindings from `0x18a38`.

Those bytes are:

- `D0`
- `CB`
- `CD`
- `1E`
- `1F`

Decoded through the executable’s key-name table at `0x18c8b`, that is:

- Down = `Down`
- Left = `Left`
- Right = `Rght`
- Rotate Left = `A`
- Rotate Right = `S`

This matches the captured keyboard-setup screen exactly.

## Default Setup State

Before any load from disk, `0x358c` seeds:

- sound device = `Autodetect`
- mixing rate = `44100`
- stereo = `1`
- bit depth = `1` (`16-bit`)
- music volume = `100`
- SFX volume = `90`
- current track index = `0`

It also copies the default `0x8c`-byte high-score block from `0x18a3f` and immediately persists the resulting state.

## `SETUP.DAT` Load/Save Correction

The file path at `0x17b0b` is the setup-state file.

Correct roles:

- `0x3604` saves the `178`-byte setup block to disk
- `0x36dc` loads it back, validates the first `5` bytes against the `AciD` signature block at `0x18873`, and falls back to `0x358c` if the file is missing or invalid

After a successful load, `0x36dc` also normalizes the current music track index modulo `6`.

## Current Practical Meaning For The Port

This gives us a clean input model for reconstruction:

- use scan-code-like logical actions for preservation parity
- support held-state checks for gameplay
- support last-released-key capture for keyboard rebinding and menus
- preserve the shipped default controls as Down / Left / Right / A / S before adding modern remapping UX
