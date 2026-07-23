# Naming Convention

This document defines the file and chunk naming rules for the ACiD Tetris decompilation effort.

The goal is to keep raw dumps stable, traceable, and easy to map back to the original files.

## Principles

- Raw dumps must have stable names.
- Raw dump names must be traceable back to the source file and byte range.
- Classification should live in the manifest first, not only in filenames.
- Converted assets should preserve the raw dump stem so provenance is always visible.

## Source Identifiers

Use short, lowercase source identifiers:

- `atet-dat`
- `setup-dat`
- `atet-exe`
- `runtime`

## Canonical Raw Dump Pattern

Use this pattern for raw extracted files:

`<source>.chunk-<id>.off-<offset>.len-<length>.raw.bin`

Example:

`atet-dat.chunk-0012.off-000F4A20.len-00012C00.raw.bin`

## Field Rules

- `source`
  Use the source identifier from this document.
- `chunk-<id>`
  Use zero-padded decimal chunk numbering in archive order.
  Recommended width: 4 digits.
- `off-<offset>`
  Use uppercase hexadecimal byte offsets.
  Recommended width: 8 digits.
- `len-<length>`
  Use uppercase hexadecimal byte lengths.
  Recommended width: 8 digits.
- `.raw.bin`
  Marks the file as an unmodified raw dump.

## Why Raw Names Stay Type-Neutral

Raw filenames should not change if our understanding improves later.

For example, a file first thought to be `unknown` may later prove to be a palette, sprite sheet, music module, or compressed data block. If the type is encoded into the raw filename, we create rename churn and break references. Keeping raw names type-neutral avoids that.

## Converted Asset Pattern

Converted assets should keep the raw stem and add a detected type or target format:

`<raw-stem>.wav`

`<raw-stem>.png`

`<raw-stem>.pal.json`

Examples:

- `atet-dat.chunk-0012.off-000F4A20.len-00012C00.wav`
- `atet-dat.chunk-0018.off-00123456.len-00045678.uni`
- `atet-dat.chunk-0021.off-00150000.len-00008000.png`

The raw stem should match the source raw dump identity, even if the converted asset uses a different extension or a more specific subtype.

## Manifest Requirements

Every extracted item should also be recorded in a machine-readable manifest.

Minimum fields:

- `source_file`
- `source_id`
- `chunk_id`
- `offset`
- `length`
- `sha256`
- `detected_type`
- `confidence`
- `raw_path`
- `converted_paths`
- `notes`

Recommended extras:

- `magic`
- `tool_version`
- `extraction_time`
- `relationships`

## Special Rule For Implicit Regions

If a file contains meaningful unindexed data before or between indexed chunks, treat it as a normal chunk in the numbering sequence.

Do not create ad hoc names for these regions if they can be represented as standard chunk entries.

## Folder Usage

- Raw archive splits go in `extracted/raw/atet-dat/`
- Raw setup data analysis outputs go in `extracted/raw/setup-dat/`
- Runtime memory or debugger dumps go in `extracted/raw/runtime-dumps/`
- Converted assets go in the matching folders under `extracted/converted/`

## Summary

The canonical identity of an extracted item is:

- source
- chunk id
- byte offset
- byte length

Everything else, including type detection and conversion output, should build on that identity rather than replace it.
