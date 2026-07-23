# Screen Buffer Role Correction Pass

Status: superseded by [screen-save-buffer-resolution-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/screen-save-buffer-resolution-pass-2026-04-14.md).

Date: 2026-04-14

This note is preserved only as a breadcrumb.

Its earlier conclusion about `0x2c69f` was wrong because it interpreted the copy direction of `0xeb0c` backward.

Use the superseding note for the corrected model:

- `0x2c69f`
  saved gameplay snapshot
- `0x2c6af`
  title/menu base
- `0x2c727`
  live working screen
