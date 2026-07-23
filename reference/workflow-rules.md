# Workflow Rules

This document records the non-negotiable workflow rules for the ACiD Tetris decompilation effort.

## Extraction Manifest Rule

Every extracted item must be represented in a machine-readable manifest.

Required fields:

- `source_file`
- `source_id`
- `chunk_id`
- `offset`
- `length`
- `sha256`
- `detected_type`
- `converted_paths`

Recommended fields:

- `confidence`
- `magic`
- `notes`
- `tool_version`
- `extraction_time`

The manifest is the authoritative index for extracted materials. Filenames should stay stable and traceable, but the manifest carries the richer metadata.

## Write Scope Rule

No script may write into `Original.Game/`.

`Original.Game/` is preserved as source evidence and must remain untouched.

All extraction, decoding, reports, manifests, logs, captures, and converted outputs must be written only under:

`Decompilation.Effort/`

This includes:

- raw dumps
- converted assets
- generated manifests
- reverse-engineering notes
- runtime captures
- helper reports
- source-port code

## Practical Implication

For this project, scripts should be designed around these assumptions:

- inputs may be read from `Original.Game/`
- outputs must be written under `Decompilation.Effort/`
- manifests are required, not optional
- raw extracted data must be preserved before conversion work begins

## Standing Project Policy

These rules are part of the active project workflow and should be followed for all future scripting and extraction work unless explicitly changed.
