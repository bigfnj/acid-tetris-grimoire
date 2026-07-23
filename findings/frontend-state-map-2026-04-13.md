# Frontend State Map

Date: 2026-04-13

## Summary

This pass resolved the menu-side dispatcher and most of the visible frontend states. The combination of the `0x3830` jump table, the menu-local selection jump tables, the referenced UI strings, and the existing capture set is enough to name the main frontend handlers with good confidence.

The practical result is that the menu flow is no longer a loose set of anonymous handlers. We can now describe which executable state owns:

- the main menu
- the options screen
- the keyboard setup screen
- the high-score table and name-entry path
- the credits sequence
- the first-run sound setup screen

## Confirmed Dispatcher State Map

The frontend dispatcher at `0x3830` uses the jump table at `0x3804`.

- State `0`
  No-op or redisplay loop state.
- State `1`
  Main menu via `0x40c0`.
- State `2`
  Enter or resume gameplay path.
  Evidence:
  selected by `Return to Game`, and also by `Play Game` from the sound setup screen.
- State `3`
  Exit-to-DOS path.
  Evidence:
  selected by `Exit Game` from the main menu and `Exit to Dos` from the sound setup screen.
- State `4`
  Start-new-game path.
  Evidence:
  selected by `New Game`, and followed by extra post-dispatch calls at `0x2d88` and `0x05e0`.
- State `5`
  Sound setup screen via `0x5a94`.
- State `6`
  Options screen via `0x4584`.
- State `7`
  Keyboard setup screen via `0x49bc`.
- State `8`
  High-score table view via `0x50b0` with `EAX = 0`.
- State `9`
  High-score qualification and name-entry path via `0x50b0` with `EAX = 1`.
- State `10`
  Credits sequence via `0x4f38`.

## Main Menu

`0x40c0` is the main menu handler.

The referenced strings match the captured title screen exactly:

- `New Game`
- `Options`
- `Music:'...'`
- `Level %d`
- `High Scores`
- `Credits`
- `Exit Game`
- `Return to Game`

The main-menu selection jump table at `0x40a0` confirms the item actions:

- Index `0` `New Game` -> dispatcher state `4`
- Index `1` `Options` -> dispatcher state `6`
- Index `2` `Music` -> cycles the active music track through `0x6544`
- Index `3` `Level` -> increments the stored starting level and wraps after `9`
- Index `4` `High Scores` -> dispatcher state `8`
- Index `5` `Credits` -> dispatcher state `10`
- Index `6` `Exit Game` -> dispatcher state `3`
- Index `7` `Return to Game` -> dispatcher state `2`, but only if the live-game flag at `0x2c72b` is set

That last point is important for the port:

- `Return to Game` is not the same path as `New Game`
- it is gated on active game state

## Options Screen

`0x4584` is the options screen handler.

Its strings match the captured options screen:

- `Music Volume:%d`
- `Sound FX Volume:%d`
- `Keyboard Setup`
- `Back To Main Menu`

The handler directly edits:

- `0x2c6eb` for music volume percent
- `0x2c6e7` for sound-effects volume percent

It also calls `0x68bb` after music-volume changes, which matches the already established music-volume conversion path.

The enter-key action table inside this handler confirms:

- `Keyboard Setup` -> dispatcher state `7`
- `Back To Main Menu` -> dispatcher state `1`

## Keyboard Setup Screen

`0x49bc` is the keyboard setup screen.

The strings match the screenshot and runtime behavior:

- `Down:%s`
- `Left:%s`
- `Right:%s`
- `Rotate Left:%s`
- `Rotate Right:%s`
- `Back to Options Menu`

The currently bound keys live at:

- `0x2c73b` down
- `0x2c73c` left
- `0x2c73d` right
- `0x2c73e` rotate left
- `0x2c73f` rotate right

The handler uses a key-name table rooted at `0x18c8b` to format the visible key names, then waits for a new key when an action line is selected. That fits both the executable behavior and the captured keyboard-setup screen.

The enter-key jump table at `0x49a4` maps cleanly to the five bindings plus a return option.

## High Scores

`0x50b0` is the high-score screen handler, with two distinct modes controlled by its input argument.

Mode `0`:

- display the current high-score table
- no insertion logic

Mode `1`:

- compare the current run score at `0x2c6cb` against the five stored entries at `0x2c623`
- insert a new blank record if the score qualifies
- shift lower entries down as needed
- copy the total line count from `0x2c72f`
- allow interactive name entry for the inserted row

The table layout matches the already decoded `SETUP.DAT` records:

- `5` entries
- each record is `0x1c` bytes
- `20` bytes of name
- `4` bytes score
- `4` bytes total lines cleared

The interactive name-entry path uses additional character-translation tables around:

- `0x18acb`
- `0x18bcb`
- `0x18c0b`
- `0x18c4b`

Those tables are selected based on modifier state and are used to turn keyboard input into displayable name characters. The name buffer is capped at `10` visible characters.

This also explains the dispatcher split:

- state `8` is "show high scores"
- state `9` is "enter a newly qualified high score"

## Credits Sequence

`0x4f38` is the credits-sequence handler.

It prints fixed-width `0x28`-byte lines from pages rooted at `0x1918b`, with `8` lines per page and a page size of `0x140` bytes. The code advances through multiple pages over time, which matches the captured multi-screen credits flow.

Recovered credits pages from the executable:

- Page `0`
  `Acid Tetris`
  `Copyright 1997`
  `Dungeon Dwellers Design`
- Page `1`
  `Programming:`
  `Jason Pimble`
  `Graphics:`
  `Scott Emerle`
- Page `2`
  `Music System:`
  `Jean Paul Mikkers`
  `Orignal Concept`
  `Design and Program By:`
  `Alexey Pazhitnov`
- Page `3`
  `Protected Mode Extender By:`
  `Charles Scheffold`
  `Thomas Pytel`
- Page `4`
  `Music:`
  `Chris Emerle`
  `Scott Emerle`
  `Liam Hesse`
  `Jon Dal Kristbjornsson`
  `Bobby Tamburrino`
- Page `5`
  `Special Thanks To:`
  `Paul Mason`
  `Cathleen Crandall`
  `John Hood`

The capture set had already confirmed part of this, but the executable now gives us the full page structure directly.

## Sound Setup Screen

`0x5a94` is the sound setup screen used before gameplay.

The screen strings and selection jump table identify it as the configuration flow behind `ATET setup`, not the regular in-game options screen.

Visible items:

- sound device name via `%s`
- `Mixing Rate:%d`
- `Stereo` or `Mono`
- `16 Bit` or `8 Bit`
- `Play Game`
- `Exit to Dos`

Resolved device-name table at `0x18878`:

- `Autodetect`
- `Gravis Ultrasound`
- `SB Family`
- `Ensoniq Soundscape`
- `None`
- `PAS Family`

Resolved mixing-rate table copied from `0x3570`:

- `11025`
- `16357`
- `22050`
- `27562`
- `33074`
- `38586`
- `44100`

The selection jump table at `0x5a7c` confirms:

- device cycling
- mixing-rate cycling
- stereo/mono toggle
- 16-bit/8-bit toggle
- `Play Game` -> dispatcher state `2`
- `Exit to Dos` -> dispatcher state `3`

That in turn tightens the meaning of the `SETUP.DAT` fields loaded at `0x3604`.

## Decompilation Impact

This pass materially improves the source-port plan:

- the frontend flow can now be recreated as explicit named states instead of guessed transitions
- the high-score screen and name-entry behavior are structurally understood
- the keyboard setup screen is no longer just "some input menu"
- the sound setup screen and the regular options screen are clearly separate systems
- the credits text can be preserved exactly from the executable-side string pages

## Recommended Next Move

The strongest next step is to carry these names back into the machine-readable hypothesis map and the `SETUP.DAT` analysis output, then continue into the gameplay-entry side:

- `0x2d88`
- `0x05e0`
- `0x5868`

That should clarify the boundary between frontend completion, new-game initialization, and live-game resume behavior.
