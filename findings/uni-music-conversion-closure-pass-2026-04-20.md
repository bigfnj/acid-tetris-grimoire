# `UNI` Music Conversion Closure Pass

Date: 2026-04-20

## Summary

This pass closed the status of the six extracted music `UNI` chunks under the local tracker toolset.

The narrow question was:

- are these six chunks playable as-is
- convertible to `MOD` or `IT`
- or still ambiguous enough to require a dedicated decoder pass

I tested each `UNI` file in:

- [Decompilation.Effort/extracted/converted/music](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/music)

with the local tools:

- `.tools/bin/openmpt123`
- `.tools/bin/mikmod`

Main result:

- all six files are already playable as-is with `mikmod`
- `openmpt123` rejects all six as unsupported
- no `MOD` or `IT` conversion path exists in the tested local toolset

So the closure is clean:

- every music `UNI` chunk now has known status
- none of the six needs format-triage ambiguity work before playback-preservation decisions

## New Owned Artifacts

- [uni-music-conversion-closure-results-2026-04-20.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/formats/atet-dat/uni-music-conversion-closure-results-2026-04-20.json)

## Probe Method

`openmpt123` probe command:

```sh
.tools/bin/openmpt123 --probe <file>
```

Interpretation:

- textual `Probe......: Failure` means unsupported
- exit code stays `0`, so the text output is the authoritative signal

`mikmod` load/playback command:

```sh
timeout 2s .tools/bin/mikmod -d 6 -q <file>
```

Interpretation:

- `-d 6` uses the nosound driver
- exit code `124` with empty stderr means the file loaded and playback continued past the timeout
- immediate loader failure would have produced an early error instead of timing out cleanly

## Six-Chunk Status Table

| Chunk | File | Header bytes | `openmpt123` | `mikmod` | Final status |
| --- | --- | --- | --- | --- | --- |
| 9 | `atet-dat.chunk-0009.off-0002BF21.len-0001F030.uni` | `55 4e 30 35 0b 19 00 00 00 0c 00 29 00 60 00 06` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |
| 10 | `atet-dat.chunk-0010.off-0004AF51.len-00029A03.uni` | `55 4e 30 35 0c 13 00 00 00 0c 00 4e 00 60 00 05` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |
| 11 | `atet-dat.chunk-0011.off-00074954.len-000295D6.uni` | `55 4e 30 35 0c 1d 00 00 00 0e 00 5b 00 21 00 06` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |
| 12 | `atet-dat.chunk-0012.off-0009DF2A.len-0005D6B4.uni` | `55 4e 30 35 0c 34 00 00 00 34 00 d5 00 3f 00 05` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |
| 13 | `atet-dat.chunk-0013.off-000FB5DE.len-00040510.uni` | `55 4e 30 35 0c 1e 00 00 00 15 00 48 00 10 00 06` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |
| 14 | `atet-dat.chunk-0014.off-0013BAEE.len-0005C75C.uni` | `55 4e 30 35 08 0b 00 00 00 0b 00 23 00 0c 00 06` | `Probe......: Failure` | `timeout -> exit 124`, empty stderr | Plays as-is with `mikmod`; no tested `MOD/IT` converter |

## Findings

### 1. All Six Files Are Structurally Valid MikMod `UNI`

`file(1)` identifies every tested chunk as:

- `MikMod UNI format module sound data`

And every header begins with:

- `55 4e 30 35`

which is ASCII:

- `UN05`

So the six files already share one clear, owned format identity.

### 2. `openmpt123` Does Not Support These `UNI` Files

For every file:

- `openmpt123 --probe` returned:
  - `Probe......: Failure`

So there is no ambiguity on the `libopenmpt` side:

- these files are unsupported by the local `openmpt123` build

### 3. `mikmod` Loads All Six As-Is

For every file:

- `mikmod -d 6 -q` continued running until the timeout
- exit code was:
  - `124`
- stderr stayed empty

That is the exact noninteractive result expected when the file loads and continues playing under the nosound driver.

So all six chunks are playable as-is in the tested local toolset.

### 4. No Tested `MOD` / `IT` Conversion Path Exists

Within the tested local tools:

- `openmpt123` can only probe / play / render PCM, and rejects all six files
- `mikmod` can play the files but no `MOD` / `IT` conversion utility is present in `.tools/bin`

So no converted `MOD` or `IT` outputs were produced, and:

- [Decompilation.Effort/extracted/converted/music/converted](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/extracted/converted/music/converted)

was not populated by this pass.

This is not a format ambiguity. It is simply the current local tool capability boundary.

## Practical Interpretation

The music-format closure is now straightforward.

These six chunks do **not** sit in the “unknown decoder needed before playback” bucket anymore. They already sit in:

- playable as-is with `mikmod`

What remains open, if we ever need it, is not basic format identification but:

- writing or acquiring a dedicated `UNI -> MOD/IT` converter

That is a tooling enhancement question, not a format-ambiguity question.

## Bottom Line

All six `UNI` music chunks now have known status with no ambiguity.

They are valid `UN05` / MikMod `UNI` modules, unsupported by `openmpt123`, and playable as-is with `mikmod`. No `MOD` or `IT` conversion path exists in the tested local toolset, so no converted tracker files were emitted.
