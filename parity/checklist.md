# ACiD Tetris Parity Checklist

Date: 2026-04-21

This checklist converts the current behavior spec into forward-looking port parity work items. Current `◑` rows reflect the Milestone C/D gameplay and transition parity shell, the Milestone B frontend-shell proof, and the preserved extracted-`WAV` probe path, not full subsystem parity.

Status legend:

- `◻` Not matched in the port yet.
- `◑` Partially matched or only verified as pipeline scaffolding.
- `✅` Behavior matched with owned evidence.

## Controls

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `CTRL-01` | `[Manual]` The main selection screen uses the arrow keys plus `Enter` to choose menu items, view high scores, view credits, enter keyboard setup, and change music selection. | ◻ | [Spec](../specs/behavior-spec.md#controls) |  |
| `CTRL-02` | `[Manual]` On the main menu, moving to the `Music:` row and pressing `Enter` switches to the next song. | ◻ | [Spec](../specs/behavior-spec.md#controls) |  |
| `CTRL-03` | `[Observed]` The default keyboard setup screen shows these gameplay bindings: | ◑ | [Spec](../specs/behavior-spec.md#controls) | Milestone B seeds the default five binding rows from original-style scan codes and can reload later build-local `SETUP.DAT` edits. |
| `CTRL-04` | `Down -> Down` | ◑ | [Spec](../specs/behavior-spec.md#controls) | Default row is present before rebinding. |
| `CTRL-05` | `Left -> Left` | ◑ | [Spec](../specs/behavior-spec.md#controls) | Default row is present before rebinding. |
| `CTRL-06` | `Right -> Right` | ◑ | [Spec](../specs/behavior-spec.md#controls) | Default row is present before rebinding. |
| `CTRL-07` | `Rotate Left -> A` | ◑ | [Spec](../specs/behavior-spec.md#controls) | Default row is present before rebinding. |
| `CTRL-08` | `Rotate Right -> S` | ◑ | [Spec](../specs/behavior-spec.md#controls) | Default row is present before rebinding. |
| `CTRL-09` | `[RE-derived]` The live gameplay loop reads exactly five configurable gameplay bindings from saved setup data: down, left, right, rotate-left, and rotate-right. | ◑ | [Spec](../specs/behavior-spec.md#controls) | Milestone B stores the five binding bytes in build-local binary `SETUP.DAT` and applies them to the live gameplay presentation piece; preset mode still uses them as an overlay-selection probe. |
| `CTRL-10` | `[RE-derived]` Selecting one of the first five rows in keyboard setup enters a capture mode where the next accepted key becomes the new binding for that action. | ◑ | [Spec](../specs/behavior-spec.md#controls) | Enter on rows `0..4` shows the underscore prompt, accepts the next supported non-`Esc` key, redraws the row, and saves the scan code to build-local `SETUP.DAT`. |
| `CTRL-11` | `[RE-derived]` Left, right, rotate-left, and rotate-right act immediately on press, then repeat independently every 8 gameplay frames while held. | ◑ | [Spec](../specs/behavior-spec.md#controls) | Saved left/right/rotate bindings now move or rotate the live presentation piece immediately and every 8 rendered frames while held. |
| `CTRL-12` | `[RE-derived]` Down does not use the same repeat rule. It uses a separate soft-drop path plus a hold-lockout field that suppresses carried `Down` input across some line-clear and spawn transitions. | ◑ | [Spec](../specs/behavior-spec.md#controls) | Saved down is one-shot soft drop with a spawn lockout field; exact original lockout timings still need certification. |
| `CTRL-13` | `[RE-derived]` `Esc` during live gameplay exits to the main menu rather than quitting the program. | ◑ | [Spec](../specs/behavior-spec.md#controls) | `Esc` now captures a gameplay texture snapshot, snapshots logical gameplay state, and returns to the main frontend with `Return to Game` enabled; exact transient restore ordering remains pending. |
| `CTRL-14` | `[RE-derived]` `Esc` after the finished game-over presentation hands control to the high-score qualification or name-entry flow rather than back to the ordinary main menu. | ◑ | [Spec](../specs/behavior-spec.md#controls) | Top-out advance keys no longer enter high scores before the temporary `GAME OVER` overlay phase has been reached and removed; the final manual handoff is release-gated, while the exact original latch byte source remains pending. |

## Scoring

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `SCORE-01` | `[Manual]` The bundled manual does not explain the scoring formula. It only says the game is "pretty self explanitory." | ◻ | [Spec](../specs/behavior-spec.md#scoring) |  |
| `SCORE-02` | `[Observed]` The gameplay HUD shows `HIGH-SCORE`, `SCORE`, `LEVEL`, and `LINES`. | ◑ | [Spec](../specs/behavior-spec.md#scoring) | The live presentation run now updates score, level, and line counters in the authored HUD regions. |
| `SCORE-03` | `[Observed]` The high-score table shows player name, score, and a second numeric field that matches total lines cleared in the run. | ◑ | [Spec](../specs/behavior-spec.md#scoring) | Milestone B renders saved records from binary `SETUP.DAT`, and live presentation score/line totals are used when qualifying a new entry. |
| `SCORE-04` | `[RE-derived]` The scoring table is: | ◑ | [Spec](../specs/behavior-spec.md#scoring) | The live presentation model uses the recovered scoring table through the `C` line-clear simulation key. |
| `SCORE-05` | `1 line -> 100` | ◑ | [Spec](../specs/behavior-spec.md#scoring) | `C` simulation applies this base value before level multiplier. |
| `SCORE-06` | `2 lines -> 300` | ◑ | [Spec](../specs/behavior-spec.md#scoring) | `C` simulation applies this base value before level multiplier. |
| `SCORE-07` | `3 lines -> 600` | ◑ | [Spec](../specs/behavior-spec.md#scoring) | `C` simulation applies this base value before level multiplier. |
| `SCORE-08` | `4 lines -> 1200` | ◑ | [Spec](../specs/behavior-spec.md#scoring) | `C` simulation applies this base value before level multiplier. |
| `SCORE-09` | `[RE-derived]` The awarded score for a clear is `(current_level + 1) * base_clear_value`. | ◑ | [Spec](../specs/behavior-spec.md#scoring) | Live presentation line-clear simulation multiplies the base clear value by `current_level + 1`. |
| `SCORE-10` | `[RE-derived]` Total cleared lines are accumulated during the run and copied into the saved high-score record if the run qualifies. | ◑ | [Spec](../specs/behavior-spec.md#scoring) | Live presentation line totals accumulate, are passed into high-score qualification, and are written to build-local `SETUP.DAT` on commit. |
| `SCORE-11` | `[RE-derived]` The game keeps only five saved high-score entries. | ◑ | [Spec](../specs/behavior-spec.md#scoring) | Build-local `SETUP.DAT` preserves and rewrites the fixed five-record layout. |

## Levels And Speed

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `LEVEL-01` | `[Manual]` The main menu exposes a `Level` row, which is the player's visible pre-game level setting. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Milestone B draws the level row in the main frontend shell. |
| `LEVEL-02` | `[Observed]` The title screen capture shows `Level 0` as the selected displayed start level. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Milestone B initializes the visible level to `0`. |
| `LEVEL-03` | `[RE-derived]` Pressing `Enter` on the `Level` row increments the stored starting level and wraps after `9`. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Enter cycles the frontend start level and the gameplay presentation HUD level uses that value; real gravity seeding is not implemented. |
| `LEVEL-04` | `[RE-derived]` The live level increases after each additional 10 cleared lines. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | The live presentation run updates level as `start_level + lines / 10` after simulated clears. |
| `LEVEL-05` | `[RE-derived]` Gravity is seeded as `(current_level << 9) + 0x200`, which gives a nominal natural descent interval of `128 / (level + 1)` gameplay frames per row. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Live gameplay advances gravity with this per-tick rate. The frame cadence is now decoded and certified (`frame-cadence-timer-decode-pass-2026-07-21`): the base timer is calibrated to the VGA vertical refresh of the 320x200 mode-13h (`0x6c62` counts PIT clocks between two `0x3da` vretrace edges), so the game runs at **70.086 Hz**. The pacer `0x24d0` consumes one tick per rendered frame (6-step catch-up). The port now paces at ~14 ms (~70 fps) to match instead of 60 fps. |
| `LEVEL-06` | `[RE-derived]` A newly spawned piece starts with gravity accumulator `0`, so the first live gameplay step after a fresh spawn normally does not drop the piece by gravity yet. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Spawn and next-piece promotion now reset the gravity accumulator to `0`. |
| `LEVEL-07` | `[RE-derived]` The game uses 10 level-themed row-clear effect helpers chosen by `current_level % 10`, so level affects clear presentation as well as speed. | ◑ | [Spec](../specs/behavior-spec.md#levels-and-speed) | Debris varies by `level % 10` motion family and uses recovered chunk-`3` color ramps. The ten helpers (`0x14b4`..`0x1c40`) and both spawners (`0x2f78` polar, `0x3034` target) are fully decoded (see `line-clear-helper-decode-pass-2026-07-21`), and the port now IMPLEMENTS them faithfully: a 16.16 fixed-point particle system with the recovered polar/target spawn constructors, the shared sine table, `0x400/lifetime` decay, 4096-particle cap, and the ten `level%10` emitters with their exact angle/speed/lifetime. The RNG-burst families (`#0`/`#1`/`#2`) draw from the certified gameplay RNG in the exact per-particle order, so clears consume RNG as the original does (verified: a 4-line clear at level 0 draws 4x640x3 = 7680; levels 4/7 draw 0). All ten emitters — including the `#4`/`#7` target-converge families (target-Y `8*row+0xf` / `8*row+0x18`, re-derived exact from `0x1830`/`0x1a9c`) — match the decode. |

## Setup Flow

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `SETUP-01` | `[Manual]` First-time setup is performed by running `SETUP.BAT` or the provided Windows shortcut for setup. After that, the player runs `ATET.EXE` to play. | ◻ | [Spec](../specs/behavior-spec.md#setup-flow) |  |
| `SETUP-02` | `[Manual]` The manual describes the first-run screen as a sound-card selection screen and recommends `Autodetect` when appropriate. | ◻ | [Spec](../specs/behavior-spec.md#setup-flow) |  |
| `SETUP-03` | `[RE-derived]` Cold startup actually has two possible frontend phases: | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Milestone B models the main-menu path, early sound-setup path, and recovered splash pair; full resource/audio-init timing is not implemented. |
| `SETUP-04` | optional early sound setup when explicit `setup` mode is requested or `SETUP.DAT` is missing or invalid | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | `setup` / `--setup` and missing/invalid build-local `SETUP.DAT` enter the sound setup shell. |
| `SETUP-05` | later normal main-menu entry after startup splashes and resource loads | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Valid build-local setup enters the recovered splash pair and then the normal main menu; exact resource/audio timing is not implemented. |
| `SETUP-06` | `[RE-derived]` Normal launch with valid setup follows this visible order: | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Parent row; Milestone B reproduces the visible DDD -> warning -> menu order without original timing certification. |
| `SETUP-07` | `DDD` splash | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Recovered chunk-0 DDD splash asset is shown as the first normal-startup page. |
| `SETUP-08` | warning splash | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Recovered chunk-2 warning screen asset is shown after the DDD splash. |
| `SETUP-09` | title and main menu | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | The title-backed menu reveal begins after the timed splash pair. |
| `SETUP-10` | gameplay after `New Game` | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | `New Game` now seeds the live gameplay presentation run after the title menu; full gameplay engine startup is not implemented. |
| `SETUP-11` | `[RE-derived]` Setup or first-run launch follows this visible order: | ◻ | [Spec](../specs/behavior-spec.md#setup-flow) | Parent row; child rows below break out exact items. |
| `SETUP-12` | title-backed sound setup screen | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Milestone B draws a title-backed sound setup shell in the shared frontend family. |
| `SETUP-13` | if the user continues instead of exiting | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | `Play Game` and `Esc` leave sound setup for the splash pair; `Exit to Dos` quits. |
| `SETUP-14` | `DDD` splash | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Setup continuation now enters the recovered DDD splash page. |
| `SETUP-15` | warning splash | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Setup continuation advances from DDD to the recovered warning screen. |
| `SETUP-16` | title and main menu | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Setup continuation enters the normal title menu reveal after the splash pair. |
| `SETUP-17` | `[RE-derived]` The first-run sound setup rows are: | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Milestone B draws and navigates the six setup rows. |
| `SETUP-18` | sound device | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row cycles the six recovered device labels and writes build-local `SETUP.DAT`. |
| `SETUP-19` | mixing rate | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row cycles the recovered seven-rate table and writes build-local `SETUP.DAT`. |
| `SETUP-20` | stereo or mono | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row toggles the saved stereo flag. |
| `SETUP-21` | bit depth | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row toggles the saved 8-bit/16-bit flag. |
| `SETUP-22` | `Play Game` | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row returns to the main menu shell. |
| `SETUP-23` | `Exit to Dos` | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Row quits the native port. |
| `SETUP-24` | `[RE-derived]` `Esc` in first-run sound setup means "leave setup and continue," not "exit to DOS." | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | `Esc` leaves sound setup for the main menu. |
| `SETUP-25` | `[RE-derived]` Sound-setup edits rewrite the saved configuration values immediately in memory, but the screen does not live-preview or reinitialize the audio backend while the user is editing. | ◑ | [Spec](../specs/behavior-spec.md#setup-flow) | Edits rewrite build-local `SETUP.DAT` immediately; no backend reinitialization is attempted. |
| `SETUP-26` | `[RE-derived]` The startup backend init can fall back to device `None` if the configured backend fails, but the presentation flow still continues into the main menu. | ◻ | [Spec](../specs/behavior-spec.md#setup-flow) |  |

## Menus: Main Menu

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `MENU-00` | Port can open a native window and present a scaled low-resolution frame. | ◑ | [Port README](../../port/README.md#what-it-does) | Verified through the native Linux build path, Milestone B frontend shell, and `--smoke-frames` offscreen run; not full parity complete yet. |
| `MENU-01` | `[Observed]` The main menu sits under a large `ACiD TETRIS` logo on a black background, with a moving green chunk-7 decorative object behind the menu text. | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Renders the authored title base, chunk-5 menu text, and the animating chunk-7 object behind the text. Capture-certified 2026-07-22 vs the original (red `ACiD` + blue `TETRIS` logo, purple menu rows, green rotating object) after the color fix; exact randomized bank selection remains modeled. |
| `MENU-02` | `[Observed]` The visible rows are: | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Milestone B draws, reveals, and navigates the eight recovered rows; exact capture-certified transition timing is not complete. |
| `MENU-03` | `New Game` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and activates the gameplay presentation demo. |
| `MENU-04` | `Options` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and opens the options shell. |
| `MENU-05` | `Music:"..."` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and cycles track names, and activating it now loads and plays the selected recovered tracker module via libmikmod; exact original track-change timing remains modeled. |
| `MENU-06` | `Level N` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and cycles the displayed start level. |
| `MENU-07` | `High Scores` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and opens the seeded high-score presentation shell. |
| `MENU-08` | `Credits` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and opens the fixed-page credits shell. |
| `MENU-09` | `Exit Game` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and exits the native port. |
| `MENU-10` | `Return to Game` | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Row visible and gated on the port's demo live-game flag. |
| `MENU-11` | `[RE-derived]` All eight rows are always drawn. `Return to Game` is visible even when no live game exists, but activation is ignored unless a resumable live game is active. | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Milestone B always draws all eight rows and ignores `Return to Game` until gameplay has been entered. |
| `MENU-12` | `[RE-derived]` The cold-boot menu presentation is staged: | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Milestone B now gates rows behind a short object warmup and row-by-row reveal, but exact original frame timing and fade staging are not complete. |
| `MENU-13` | title and logo base appears first | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Milestone B loads the authored title base before drawing chunk-7 objects and text. |
| `MENU-14` | floating chunk-7 objects fade in | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | A deterministic projected-object cycle is visible; exact fade-in and random target-bank behavior remain future work. |
| `MENU-15` | the eight menu rows reveal afterward | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Rows are drawn from chunk-5 font assets after a view-local warmup and row cadence; exact cadence remains future work. |
| `MENU-16` | only then does the steady selected-row pulse begin | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Selected row pulse is held until all rows in the current view have revealed. |
| `MENU-17` | `[RE-derived]` The initial selected row on a normal cold boot is `New Game`. | ◑ | [Spec](../specs/behavior-spec.md#main-menu) | Milestone B initializes the frontend selection to `New Game`. |

## Menus: Options

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `OPT-01` | `[Observed]` The options screen shows four active rows: | ◑ | [Spec](../specs/behavior-spec.md#options) | Milestone B draws and navigates the four-row options shell. |
| `OPT-02` | `Music Volume` | ◑ | [Spec](../specs/behavior-spec.md#options) | Row visible and mutable; drives the live libmikmod music volume through the exact original curve (see `OPT-08`). |
| `OPT-03` | `Sound FX Volume` | ◑ | [Spec](../specs/behavior-spec.md#options) | Row visible and mutable; routed SFX events play recovered WAV slots with the saved value as stream gain, while exact original mixer behavior remains pending. |
| `OPT-04` | `Keyboard Setup` | ◑ | [Spec](../specs/behavior-spec.md#options) | Row visible and opens the keyboard setup shell. |
| `OPT-05` | `Back To Main Menu` | ◑ | [Spec](../specs/behavior-spec.md#options) | Row visible and returns to the main frontend shell. |
| `OPT-06` | `[RE-derived]` Options still uses the shared eight-row frontend scaffold; the four unused rows are blank padding rather than a different dialog type. | ◑ | [Spec](../specs/behavior-spec.md#options) | Milestone B pads the options row vector to the shared eight-row scaffold, with four blank rows. |
| `OPT-07` | `[RE-derived]` Row movement is release-gated, but value edits use held-state checks, so the screen feels like a menu with held controls rather than a static form. | ◑ | [Spec](../specs/behavior-spec.md#options) | SDL key-repeat events are ignored for row movement, while held `Left` / `Right` repeats editable option and setup values after an initial delay. |
| `OPT-08` | `[RE-derived]` Changing `Music Volume` applies immediately to the currently playing music. | ◑ | [Spec](../specs/behavior-spec.md#options) | Music volume edits route a music-volume event that sets the target volume on the live libmikmod stream; it applies immediately when idle, and while a fade is running the ramp now lands on the new target. Volume follows the exact original curve now: the ramp (`0x6965`) is linear in the configured percent, and the mixer volume is `percent * 7 / 20` (`0x68bb`), i.e. 100% -> 35 on MikMod's 0..128 scale (music sits below full). Implemented in `ToMikModVolume`. |
| `OPT-09` | `[RE-derived]` Changing `Sound FX Volume` rewrites the stored value immediately, but its audible effect is only heard on later sounds; there is no dedicated live SFX preview helper. | ◑ | [Spec](../specs/behavior-spec.md#options) | Edits persist immediately, and later routed SFX playback scales the saved percent through the RE-derived `0x6817` engine range (`0..0x40`) before mixing (e.g. 90% -> 57/64, 100% -> 64/64), instead of a flat percent gain. Observable as `engine=N/64` in the per-play `SFX mix:` log. |
| `OPT-10` | `[RE-derived]` Moving between option rows plays the normal menu navigation sound using the current SFX-volume setting. | ◑ | [Spec](../specs/behavior-spec.md#options) | Menu row movement routes and plays recovered slot `1` through the queued audio-event layer at the current SFX volume. |

## Menus: Keyboard Setup

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `KEY-01` | `[Observed]` The keyboard screen lists five bindings and a `Back to Options Menu` row. | ◑ | [Spec](../specs/behavior-spec.md#keyboard-setup) | Milestone B draws the binding rows, return row, row-local capture prompt/restore flow, and build-local binary `SETUP.DAT` binding persistence. |
| `KEY-02` | `[RE-derived]` Like options and sound setup, keyboard setup uses the shared eight-row scaffold with blank padded rows at the bottom. | ◑ | [Spec](../specs/behavior-spec.md#keyboard-setup) | Milestone B pads keyboard setup to the shared eight-row scaffold with two blank rows and the shared reveal/pulse gate. |
| `KEY-03` | `[RE-derived]` The first five rows are binding-capture rows; the sixth returns to options. | ◑ | [Spec](../specs/behavior-spec.md#keyboard-setup) | Enter on rows `0..4` shows the underscore prompt and accepts/saves the next supported non-`Esc` key; row `5` returns to options. |

## Menus: High Scores And Name Entry

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `HS-01` | `[Observed]` The steady high-score screen keeps the shared title and logo base plus the floating green object behind the table text. The footer reads `Return to Main Menu`. | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Milestone B draws a title-backed high-score shell with shared chunk-7 object animation and return footer. |
| `HS-02` | `[Observed]` The capture shows five rows of names with score and line totals. | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Milestone B displays the five saved records loaded from build-local binary `SETUP.DAT`; demo name entry can update them. |
| `HS-03` | `[RE-derived]` There are two related states: | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Milestone B now has a steady table view and a separate name-entry shell, but not the exact game-over handoff timing. |
| `HS-04` | state `8`: show the saved high-score table | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | The steady high-score shell reads and displays the five saved records; exact state timing and row reveal remain future work. |
| `HS-05` | state `9`: qualify a new score, insert it if needed, then allow name entry | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Live presentation and top-out demo handoffs now use the current run score/lines and only open name entry if the score qualifies for the five-row table. |
| `HS-06` | `[RE-derived]` Qualifying a score does not open an instant text field. The visible sequence is: | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Top-out qualification now runs a state-`9` shared title/logo/chunk-`7` bootstrap before table reveal; direct demo name-entry still uses the shared reveal gate. |
| `HS-07` | row-by-row reveal of the table | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | High-score and name-entry rows use the shared frontend reveal gate before the inserted row displays the underscore cursor, with top-out state-`9` delaying that reveal until after the bootstrap phase. |
| `HS-08` | live row-local name entry for the inserted record | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | The inserted row shows a local underscore cursor and accepts letters, digits, space, and backspace. |
| `HS-09` | one-shot commit gate | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Pressing `Enter` now stages the commit and released `Enter` commits the temporary name into build-local `SETUP.DAT`. |
| `HS-10` | footer exit gate | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Steady high-score exit waits for table reveal and then accepts the footer exit gate. |
| `HS-11` | conceal back to the main menu | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Footer exit now enters a timed row-conceal-to-main path, hiding high-score rows from the footer upward before returning to the main menu; exact original conceal art/timing remains pending. |
| `HS-12` | `[RE-derived]` Name entry uses a local temporary buffer, caps visible names at 10 characters, supports backspace, and has a case-toggle behavior. | ◑ | [Spec](../specs/behavior-spec.md#high-scores-and-name-entry) | Temporary buffer, 10-character cap, backspace, and `Shift` case toggling are present; exact original case-toggle keying is not certified. |

## Menus: Credits

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `CRED-01` | `[Observed]` Credits uses the same title-backed presentation family as the other frontend screens, but displays centered text pages instead of a selectable menu. | ◑ | [Spec](../specs/behavior-spec.md#credits) | Milestone B draws title-backed centered credits pages with the shared chunk-7 object animation. |
| `CRED-02` | `[Observed]` The captured pages include `Acid Tetris`, `Copyright 1997`, programming and graphics credits, and protected-mode extender credits. | ◑ | [Spec](../specs/behavior-spec.md#credits) | Milestone B includes the recovered fixed-page credit text. |
| `CRED-03` | `[RE-derived]` Credits is a six-page fixed pager with eight lines per page, not a continuous scroll. | ◑ | [Spec](../specs/behavior-spec.md#credits) | Six pages are present, manually pageable, and auto-advance on a fixed timer; exact original cadence is not certified. |
| `CRED-04` | `[RE-derived]` Each page's text is drawn once, then left in place while only the shared floating-object system and a page timer continue running. | ✅ | [Spec](../specs/behavior-spec.md#credits) | Behaviorally matched: the port shows static per-page text with only the chunk-7 objects and the page timer animating (identical on screen). The original's "draw once into a page-local backing surface" is a 1997 performance optimization with no observable effect; the port redraws the same static text each frame through the shared frontend renderer instead, which is intentionally not special-cased (replicating the backing surface would fork the shared renderer for zero visible benefit). |
| `CRED-05` | `[RE-derived]` `Esc` interrupts credits immediately and returns to the main menu using the animated frontend exit path. | ◑ | [Spec](../specs/behavior-spec.md#credits) | `Esc` returns to the main menu and restarts the view-local reveal; exact animated exit path is not complete. |

## Gameplay Loop

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `GAME-01` | `[Observed]` The main gameplay screen has a tall central well, a `NEXT` preview box at left, score and level counters below that preview, a `LINES` counter below the well, and a piece-stat panel on the right. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | The live presentation run updates the authored HUD, preview, well, and piece-stat regions from run state. |
| `GAME-02` | `[Observed]` The observed gameplay screen uses a themed background with decorative borders around the well and side panels. | ✅ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Renders the extracted chunk-`1` gameplay base art. Capture-certified 2026-07-22 against the original running in DOSBox-X (blue cave/water background, wood-framed well, side panels) after fixing the texture pixel-format color bug (`capture-certification-2026-07-22`). |
| `GAME-03` | `[RE-derived]` The logical playfield is a 10x20 board. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | The live presentation model owns a 10x20 board array and renders within the recovered well bounds. |
| `GAME-04` | `[RE-derived]` Pieces use four orientations, and placement checks allow negative Y so a new piece may begin partly above the visible top of the well. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | The live presentation piece has four rotations and placement checks against bounds/occupied cells; full spawn-above-top behavior is not exercised yet. |
| `GAME-05` | `[RE-derived]` A new piece starts at board `X = 4`, `Y = 0`, `rotation = 0`, with gravity accumulator `0`. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | `New Game` and promoted spawns seed `X=4`, `Y=0`, `rotation=0`, and gravity accumulator `0`; exact first-frame certification remains pending. |
| `GAME-06` | `[RE-derived]` The game keeps both a current piece and a next piece. Promoting the next piece also increments the per-piece statistics panel for the promoted piece type. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Live gameplay promotes next pieces after lock/collapse, increments piece stats before the spawn-collision check, and selects pieces through the random-table modulo-`7` path, now certified bit-exact against the decompilation (`rand` `0xeceb`, fill `0x32b0`, consume+wrap-at-`0x1FFE` `0x3280`): the port reproduces the original piece sequence for any given seed. The startup seed source is resolved as a non-deterministic shared entropy global (`0x2d2a3`, no recovered fixed write), modeled with a settable/default seed. |
| `GAME-07` | `[RE-derived]` Piece draw and piece erase are separate helpers. The game restores the old footprint before moving or rotating, then redraws the piece in its new position. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | The renderer redraws the authored base/well each frame, then draws the live piece at its updated position. |
| `GAME-08` | `[RE-derived]` When one or more rows clear, the game: | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Live gameplay now detects full rows after lock and enters a collapse phase; exact visual helper parity remains pending. |
| `GAME-09` | records the clear count | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Full-row detection records the pending clear rows; `C` now injects rows only as a smoke helper. |
| `GAME-10` | updates score and total lines | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Collapse completion applies the recovered scoring table, total lines, and level progression. |
| `GAME-11` | runs the current level's row-clear visual helper, where level selects motion profile while sampled row pixels choose debris colors through shared chunk-3 ramps | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Dense debris is seeded from cleared tile pixels, colored through the recovered chunk-`3` four-stage ramp table, and varied by `level % 10`. Exact helper motion is decoded (polar `speed*(cos,sin)` via `0x2f78` / target `(target-start)/lifetime` via `0x3034`; 16.16 fixed-point, `0x400/lifetime` decay, 4096-particle cap) — see `line-clear-helper-decode-pass-2026-07-21` — and now implemented faithfully in the port (`SeedLineClearDebris` dispatches the ten `level%10` emitters through the recovered polar/target spawners; RNG-burst families consume the certified gameplay RNG). |
| `GAME-12` | enters a multi-frame collapse path that temporarily bypasses normal movement and spawn logic | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Pending-clear collapse now bypasses falling, movement, gravity, lock, and spawn while rows hide progressively until collapse completes. |
| `GAME-13` | `[RE-derived]` Line-clear debris is intentionally dense, queue-capped, and allowed to overlap the collapse frames rather than being a tiny one-shot particle effect. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Debris is sampled per non-zero tile pixel, queue-capped, and ages independently across collapse frames. |
| `GAME-14` | `[RE-derived]` Rising stack height triggers escalating warning visuals and warning sounds before top-out. The highest occupied row determines which warning band is active. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Highest occupied row now selects warning bands, shows the alert face, and logs recovered warning slots with cooldowns; exact art/timing remains pending. |
| `GAME-15` | `[RE-derived]` Pressing `Esc` during live play snapshots the clean gameplay image, opens the main menu, and leaves the current music playing. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | `Esc` captures a gameplay texture snapshot, saves logical gameplay state, and logs music continuity, then runs the RE-derived screen fade-out-to-black before the menu fades in (`0x3830` entry: `0x64f0`/`0x63b8`, 64 ticks each; `frontend-fade-transitions-2026-07-21`). Music keeps playing across it. |
| `GAME-16` | `[RE-derived]` Choosing `Return to Game` restores the saved gameplay snapshot and preserved run state rather than starting a partial rebootstrap. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | `Return to Game` fades the menu out, restores falling/collapse/top-out logical state (no gameplay bootstrap), then fades gameplay back in (`0x3830` tail: `0x62e0`/`0x6498`); the first resumed tick advances the restored phase. |
| `GAME-17` | `[RE-derived]` The first visible frame after `New Game` is already live in engine terms, but it usually shows the newly seeded scene rather than a gravity-dropped piece. Held movement or rotation can still affect that first frame. | ◑ | [Spec](../specs/behavior-spec.md#gameplay-loop) | Live gravity now starts from accumulator `0`; held inputs can still be processed before the first natural drop. |

## Topout

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `TOP-01` | `[Observed]` The captured early game-over frame shows a large red `GAME OVER` overlay over the playfield and a yellow face graphic on the left side of the gameplay presentation. | ◑ | [Spec](../specs/behavior-spec.md#topout) | The top-out presentation demo draws the recovered game-over overlay and chunk-6 alert face tile; exact coordinates and live trigger are not certified. |
| `TOP-02` | `[Observed]` The top-out capture confirms that the post-game-over path continues into the shared title and logo family before the high-score table fully settles. | ◑ | [Spec](../specs/behavior-spec.md#topout) | After the demo sequence, control enters a title-backed state-`9` bootstrap phase before high-score rows reveal and settle. |
| `TOP-03` | `[RE-derived]` The original top-out path is a staged sequence, not an instant jump from failed spawn to score table: | ◑ | [Spec](../specs/behavior-spec.md#topout) | Failed spawn and the `O`/`--topout-demo` smoke shortcut now route through the staged top-out path with capture-bounded overlay and handoff timing; exact per-frame effect parity remains pending. |
| `TOP-04` | failed spawn triggers alert face `6` and the top-out sound | ◑ | [Spec](../specs/behavior-spec.md#topout) | Failed spawn now advances preview/stat state, draws the colliding spawned piece in top-out staging, and plays recovered slot `5`; exact mixer class/pan behavior remains pending. |
| `TOP-05` | the gameplay image dissolves row by row | ◑ | [Spec](../specs/behavior-spec.md#topout) | The demo blacks out the playfield in 20 row slices across the capture-bounded pre-overlay interval before high-score handoff; exact dissolve effect is not certified. |
| `TOP-06` | a late saved-under `GAME OVER` overlay appears | ◑ | [Spec](../specs/behavior-spec.md#topout) | The recovered `GAME OVER` overlay appears after the slot-`5` tail window and remains through the late top-out interval. |
| `TOP-07` | the overlay is later removed | ◑ | [Spec](../specs/behavior-spec.md#topout) | The overlay is removed before the capture-bounded state-`9` handoff. |
| `TOP-08` | only then does the shared high-score bootstrap and reveal begin | ◑ | [Spec](../specs/behavior-spec.md#topout) | Automatic handoff enters the recovered state-`9` stronger bootstrap onset window, holds the title/logo/chunk-`7` presentation, then lets the high-score rows reveal. |
| `TOP-09` | `[RE-derived]` The game-over overlay is a real saved-under overlay phase, not just text drawn as the first top-out frame. | ◑ | [Spec](../specs/behavior-spec.md#topout) | The overlay now composites over a saved-under backing texture that is captured once when the overlay first appears and held frozen (rather than live-redrawn) through overlay removal; advance keys are gated around that temporary overlay and final manual handoff waits for key release. The backing spans the full backdrop; exact original saved-under rectangle bounds remain modeled. Observable as `savedunder=` in `--debug-state`. |
| `TOP-10` | `[RE-derived]` After the finished game-over sequence, the session loop restores the overlay background and then hands control to the high-score qualification or entry state. | ◑ | [Spec](../specs/behavior-spec.md#topout) | Overlay background is now restored from the saved-under backing texture (not a live redraw) after the overlay is removed, before calling the existing high-score entry insertion path. |
| `TOP-11` | `[RE-derived]` The high-score family later conceals and returns to the main menu rather than staying in a dead-end game-over screen. | ◑ | [Spec](../specs/behavior-spec.md#topout) | Existing high-score entry/table controls return to the main menu; exact conceal animation is not implemented. |

## Audio Triggers

| Item | Original behavior | Port status | Evidence link | Notes |
| --- | --- | --- | --- | --- |
| `AUD-00` | Port can load and play one extracted WAV on keypress. | ◑ | [Port README](../../port/README.md#what-it-does) | Preserved as a keypress smoke path on `Space`; the port also loads the recovered twelve-slot SFX table for routed events, but tracker playback remains pending. |
| `AUD-01` | `[Manual]` The bundled manual names six songs and tells the player to use the `Music:` menu row to cycle tracks. | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-02` | `[Manual]` The shipped track list is: | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) | Parent row; child rows below break out exact items. |
| `AUD-03` | `Continuum` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-04` | `Tearing Up SpaceTime` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-05` | `Inner Walls Released` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-06` | `Costumed` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-07` | `I See It Now` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-08` | `Simple Song` | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-09` | `[Manual]` If the game feels sluggish, the manual recommends lowering the mixing rate and only disabling music as a last resort. | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-10` | `[Observed]` The capture set preserves six extracted music references that match the manual's credited songs. | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-11` | `[Observed]` The title capture shows the default visible track as `Continuum`. | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |
| `AUD-12` | `[RE-derived]` Startup selects and starts the current music track before the main menu rows appear. | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Startup completion routes a current-track start event before the menu reveal, and that event now loads and plays the recovered tracker module via libmikmod (verified: track 0 loads as `Continuum`); exact original start timing remains modeled. |
| `AUD-13` | `[RE-derived]` The current music track continues across: | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Audio-event state now logs continuity across gameplay/frontend boundaries; real tracker playback is pending. |
| `AUD-14` | `New Game` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | `New Game` logs current-track continuity. |
| `AUD-15` | `Return to Game` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | `Return to Game` restores live state and logs current-track continuity. |
| `AUD-16` | the top-out to high-score handoff | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Top-out to high-score handoff now routes a music-continuity event. |
| `AUD-17` | return from high scores to the main menu | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | High-score footer exit routes a conceal-to-main continuity event. |
| `AUD-18` | `[RE-derived]` The only normal track-change actions are: | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Startup current-track and explicit `Music:` track-change events drive real libmikmod playback through the RE-derived track loader (`0x6544`): a track change fades the current track out over `0x20` ticks, then loads and fades the new module in over `0x20` ticks (cold startup skips the fade-out and fades in). Verified via the `music_ramp`/`music_ramp_left`/`music_vol` fields in `--debug-state`; tick-to-realtime cadence remains modeled. |
| `AUD-19` | cold startup's initial selected track load | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Startup completion routes current-track start. |
| `AUD-20` | explicit `Music:` row activation in the main menu | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Music row activation persists the track index and logs a track-change event. |
| `AUD-21` | `[RE-derived]` `Exit Game` is the audio event that schedules a fade-out before shutdown. | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | The main-menu `Exit Game` row runs the RE-derived `0x40`-tick music fade-out, and the screen now fades to black over the same 64 ticks (the state-`3` exit path `0x3830` runs the frontend palette fade-out — `0x64f0`, also `0x40` ticks — alongside the music ramp). The run loop defers the quit until the fade completes. Other quit paths (Esc, sound-setup `Exit to Dos`) schedule no fade and stop at once. Verified self-quit after 64 fade ticks at ~70 Hz. |
| `AUD-22` | `[RE-derived]` High-confidence sound effects are: | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Parent row; the port now models the `0x6817` mixer entry point: each play carries a slot, an engine volume (percent scaled into `0..0x40`), a per-play pan (`0x80` centered), and a coarse voice class. Pan is rendered as left/right gains through a mono->stereo mixdown. The previously-missing slot-`0` piece-lock/board-impact sound is now routed at lock with the exact recovered pan curve `(pieceCol - 5) * 21 + 0x80` (decoded from the `0x0d4e` caller; verified: column 4 -> pan `107`, column 7 -> pan `170`). Realized params observable via the `SFX mix:` log. |
| `AUD-23` | menu row move -> slot `1` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Menu row movement routes and plays recovered slot `1` centered (pan `0x80`) through voice class `0` (verified via `SFX mix:` log). |
| `AUD-24` | low stack warning -> slot `3` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Low warning band routes and plays recovered slot `3` centered through voice class `1` (`0x6817` EDX=1) with cooldown. |
| `AUD-25` | medium or high stack warning -> slot `4` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Medium/high warning bands route and play recovered slot `4` centered through voice class `1` with cooldown. |
| `AUD-26` | top-out -> slot `5` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Top-out staging routes and plays recovered slot `5`. |
| `AUD-27` | sleepy no-clear timeout -> slot `6` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Live gameplay tracks a no-clear timeout and plays recovered slot `6`; exact timeout duration remains pending. |
| `AUD-28` | 1-line clear -> slot `9` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | One-line collapse routes and plays recovered slot `9`. |
| `AUD-29` | 2-line clear -> slot `7` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Two-line collapse routes and plays recovered slot `7`. |
| `AUD-30` | 3-line clear -> slot `7` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Three-line collapse routes and plays recovered slot `7`. |
| `AUD-31` | 4-line clear -> slot `10` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Four-line collapse routes and plays recovered slot `10`. |
| `AUD-32` | repeated back-to-back 4-line clears -> slot `11` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Back-to-back four-line clears route and play recovered slot `11`. |
| `AUD-33` | waking from the sleepy state with a 1-line clear -> slot `8` | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | A one-line clear while sleepy routes and plays recovered slot `8` before the normal one-line event. |
| `AUD-34` | `[RE-derived]` Warning sounds are cooldown-gated by stack-height band, and those cooldowns pause during line-clear collapse even though alert visuals continue aging. | ◑ | [Spec](../specs/behavior-spec.md#audio-triggers) | Warning cooldowns advance outside collapse and pause while collapse is active; exact original durations remain pending. |
| `AUD-35` | `[RE-derived]` The first-run sound setup screen is configuration-only. It rewrites saved settings but does not live-preview device changes or live-reinitialize the backend while the user edits rows. | ◻ | [Spec](../specs/behavior-spec.md#audio-triggers) |  |

<!-- spec-row-count: 168 -->
<!-- checklist-row-count-including-bootstrap: 170 -->
