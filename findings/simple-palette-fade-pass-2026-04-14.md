# Simple Palette Fade Pass

Status: corrected by [simple-palette-direction-correction-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/simple-palette-direction-correction-pass-2026-04-14.md).

Date: 2026-04-14

## Summary

This pass closes the remaining ambiguity around the small fade helpers at:

- `0x6498`
- `0x64f0`

The important result is that these are not lightweight variants of the chunk-7 frontend animation fades.
They are simpler palette-only fades that:

- wait on the global tick counter
- scale a source `0x300`-byte palette through `0x284c`
- upload the scaled palette through `0x2574`

They do not redraw, flush, or present through the normal frontend frame pipeline.

## Correction

This note got the direction of the two simple palette helpers reversed.

What changed:

- `0x6498`
  is the simple palette **fade-in** helper
- `0x64f0`
  is the simple palette **fade-out** helper

Why:

`0x284c` scales each palette byte by:

- `(0x40 - EBX) / 0x40`

So `EBX` is a darkness term, not a direct brightness term.

That means:

- `0x6498` with `EBX = 0x40 - elapsed`
  becomes a visible fade **from black to full**
- `0x64f0` with `EBX = elapsed`
  becomes a visible fade **from full to black**

The chunk-7 fade pair `0x62e0` / `0x63b8` did **not** need reversal.
Only the simple pair was mislabeled.
