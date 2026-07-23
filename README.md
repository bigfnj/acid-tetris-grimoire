# ACiD Tetris Grimoire

The reverse-engineering knowledge base behind
[**acid-tetris-reborn**](https://github.com/bigfnj/acid-tetris-reborn), a modern
source port of the 1997 MS-DOS game *ACiD Tetris*.

This repository is documentation, not code. It collects the behavioural findings,
specifications, and parity notes recovered by studying the original executable —
the "how it actually works" that the port was rebuilt from. If you are porting,
preserving, or just curious how a late-'90s DOS demoscene game was put together,
start here.

## What's inside

- **`findings/`** — dated reverse-engineering passes, one topic each: the startup
  and frontend state machine, the menu/highscore text animation, the gameplay
  input and gravity/soft-drop timing, the RNG and piece selection, the tracker
  music and SFX mixer, the line-clear and top-out particle systems, the alert /
  mood-face system, the palette and graphics decoding, the frame cadence, and
  much more. `findings/INDEX.md` is the table of contents.
- **`specs/`** — the consolidated behaviour specification.
- **`parity/`** — the port parity checklist (what has been matched against the
  original and how it was verified).
- **`reference/`** — supporting reference material.

## Method

The findings were produced by static analysis of the original program (addresses,
control flow, and algorithms described in prose, with short illustrative
snippets), cross-checked against captures of the running original in an emulator.
They describe *behaviour and structure* — timing constants, data-file layout,
decode algorithms, particle physics, sound-event mappings — so the port can
reproduce the game faithfully. They deliberately do **not** reproduce the
original program wholesale: there is no full decompiled source, no disassembly
dump, and no game binary or data here.

## Scope and legal

*ACiD Tetris* is © 1997 Jason Pimble & Scott Emerle / Dungeon Dwellers Design,
distributed as freeware. This is an independent, non-commercial project of
interoperability/preservation reverse-engineering and original written analysis;
it is not affiliated with or endorsed by the original authors. It contains none
of the original game's code, data, or assets. "Tetris" is a registered trademark
of The Tetris Company, LLC, used here only to identify the 1997 work; no
association is claimed. (The game was itself renamed "SABA" in 2002.)

The written analysis in this repository is licensed under
[CC BY 4.0](LICENSE). That license covers this documentation only, not the
original game it describes.
