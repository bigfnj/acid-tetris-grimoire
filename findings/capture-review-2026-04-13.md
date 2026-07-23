# Capture Review

Date: 2026-04-13
Source folder: `Decompilation.Effort/captures/`

## Summary

The newly added screenshots, video captures, and MP3 references are immediately useful to the decompilation effort.

They confirm several runtime behaviors and UI layouts that were previously inferred only from the binary and raw data:

- the menu and credits presentation style
- the options and keyboard setup flow
- the exact seeded high-score display
- the gameplay HUD layout
- the visual style of the playfield, pieces, and supporting UI graphics
- the existence of intro/title presentation before the main menu
- the correspondence between the six recovered music chunks and six playable reference tracks

## File Set Reviewed

### Audio

- `captures/audio/01 - continuum.mp3`
- `captures/audio/02 - tearing up spacetime.mp3`
- `captures/audio/03 - inner walls released.mp3`
- `captures/audio/04 - costumed.mp3`
- `captures/audio/05 - i see it now.mp3`
- `captures/audio/06 - simple song.mp3`

### Screenshots

- `captures/screenshots/01-title.png`
- `captures/screenshots/02-options.png`
- `captures/screenshots/03-keyboardsetup.png`
- `captures/screenshots/04-hiscores.png`
- `captures/screenshots/05-credits.png`
- `captures/screenshots/05.1-credits.png`
- `captures/screenshots/05.2-credits.png`
- `captures/screenshots/06-newgame.png`
- `captures/screenshots/07-gameplay.png`
- `captures/screenshots/08-gameover.png`

### Video

- `captures/video/01-title&options.mkv`
- `captures/video/gameplay.mkv`

Working frame grabs were also created under:

- `captures/video/frames/`

## What The Screenshots Confirm

### Main Menu / Title Presentation

- The title art is dominated by a large stylized `ACiD TETRiS` logo on a black background.
- The menu text uses large beveled purple lettering with a metallic highlight.
- A green rotating or shifting decorative cursor/marker appears beside the selected option.
- The menu list includes:
  - `New Game`
  - `Options`
  - `Music: "Continuum"`
  - `Level 0`
  - `High Scores`
  - `Credits`
  - `Exit Game`
  - `Return to Game`
- The presence of `Return to Game` on the title/menu screen suggests the menu system is reused in both fresh-launch and in-game pause/return contexts.

### Options / Input Setup

- The options screen clearly exposes:
  - `Music Volume:100`
  - `Sound FX Volume:90`
  - `Keyboard Setup`
  - `Back To Main Menu`
- The keyboard setup screenshot confirms the default control mapping:
  - `Down: Down`
  - `Left: Left`
  - `Right: Right`
  - `Rotate Left: A`
  - `Rotate Right: S`
- No dedicated hard-drop control is visible in the captured keyboard setup screen.

### High Scores

- The `04-hiscores.png` screenshot exactly matches the five seeded names recovered from `SETUP.DAT`:
  - Jason
  - Scott
  - Bobby
  - Liam
  - Paul
- The score values also match the decoded file contents:
  - `100000`
  - `15000`
  - `10000`
  - `6000`
  - `3000`
- The rightmost values in the high-score table also match `SETUP.DAT`:
  - `100`
  - `50`
  - `30`
  - `20`
  - `15`
- Later executable analysis resolved the meaning of this second numeric column:
  it is total `LINES` cleared in the recorded run.

### Credits

- The credits screens validate multiple names already seen in the docs and executable metadata:
  - `Acid Tetris`
  - `Copyright 1997`
  - `Dungeon Dwellers Design`
  - `Protected Mode Extender By: Charles Scheffold, Thomas Pytel`
  - `Programming: Jason Pimble`
  - `Graphics: Scott Emerle`
- This strongly reinforces the PMODE/W finding from the executable strings.

### Gameplay Screen Layout

- The gameplay screen is much more visually elaborate than the menu system.
- The playfield sits in the center with a wooden or carved border.
- The surrounding backdrop is a blue marbled/organic texture with black jagged edging.
- The left panel contains:
  - `NEXT`
  - `HIGH-SCORE`
  - `SCORE`
  - `LEVEL`
- The bottom panel contains:
  - `LINES`
- The right panel appears to be a per-piece statistics table:
  - each tetromino type is shown with an icon
  - each has a 4-digit counter
- Piece colors visible in captures:
  - brown / tan
  - red
  - purple
  - cyan
  - blue
  - yellow
  - green
- The pieces are not flat-color primitives; they have shaded/glossy sprite artwork with visible bevel/highlight treatment.

### Gameplay / State Observations

- A new game begins with the board empty and a piece spawning high in the central playfield.
- The next-piece preview box shows the next tetromino with the same shaded art style used in the field.
- The right-side piece statistics increment during gameplay.
- A captured gameplay frame shows particle-like colored speckles around the lower board area, suggesting a line-clear or board-impact visual effect.
- A yellow facial/emoticon graphic appears in at least one gameplay-related capture and again on the game-over screen, suggesting character/sprite overlays tied to game state or feedback.
- The game-over screen overlays large red-brown `GAME OVER` text across the playfield area.

## What The Video Adds

### Title / Options Video

`01-title&options.mkv`

- Duration: about `171.972` seconds
- Video: `1920x1080`, AV1, about `59.94 fps`
- Audio: stereo `48 kHz` PCM

Useful additions beyond the screenshots:

- Confirms there is an intro or splash sequence before the main menu.
- Confirms the menu system is animated rather than being a purely static screen set.
- Confirms the credits and setup pages are part of a navigable flow rather than isolated assets.

### Gameplay Video

`gameplay.mkv`

- Duration: about `123.925` seconds
- Video: `1920x1080`, AV1, about `59.94 fps`
- Audio: stereo `48 kHz` PCM

Useful additions beyond the screenshots:

- Confirms gameplay state transitions in motion.
- Gives us a reference for spawn position, falling behavior, and board composition over time.
- Preserves timing feel and visual effects in a way still screenshots cannot.

Important capture note:

- The MKV files appear to be desktop/window captures with the game occupying only part of the full 1920x1080 frame.
- That means the videos are excellent behavioral references, but not ideal sources for pixel-accurate asset dimensions.
- The PNG screenshots are more useful than the videos for layout study, and future captures would be even better if cropped to the game window or captured at native output size.

## Audio Reference Value

The six MP3 files align directly with the six music identifiers previously recovered from the `UN05` chunks in `ATET.DAT`:

- `Continuum`
- `Tearing Up Spacetime`
- `Inner Walls Released`
- `Costumed`
- `I See It Now`
- `Simple Song`

This is very helpful because:

- it confirms our music-chunk interpretation is correct
- it gives us immediate listening/reference versions while the raw `UNI` payloads remain in preservation form
- it may help later when validating successful conversion or playback of recovered music assets

Important preservation note:

- These MP3s should be treated as modern reference derivatives, not as replacements for the raw `UNI` chunks inside `ATET.DAT`

## Decompilation Impact

These captures improve the confidence of several working hypotheses:

- The early custom archive chunks likely include multiple palette banks for menu/gameplay/credits presentation.
- The large unknown graphics chunk is very likely tied to gameplay art, including board framing, backdrop textures, tetromino sprites, or HUD panels.
- The high-score decoding work is validated by the screenshot evidence.
- The menu/input/settings flow is now externally documented and can be reimplemented faithfully later even before all internal code paths are named.

## Recommended Next Steps

1. Add a palette extraction/export pass aimed first at the chunks already flagged as likely palette data.
2. Build a simple indexed/raw graphics visualizer for the large unknown graphics candidate chunk.
3. Create a capture index file later if the media set grows, but the current folder is still manageable.
4. Prefer future video captures that are cropped to the game window or recorded at the game's visible output only.
5. Use the gameplay video as a reference when we begin documenting line-clear effects, pause flow, and state transitions.

## Bottom Line

The captures are absolutely worth keeping and materially improve the reverse-engineering effort.

They already confirmed:

- seeded high-score rendering
- menu and options structure
- default control bindings
- credits content
- gameplay HUD arrangement
- piece-stat tracking UI
- a richer gameplay art set than the menu screens alone suggested

They should now be treated as reference evidence alongside the binary, raw archive dumps, and decompilation notes.
