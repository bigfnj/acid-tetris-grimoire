# Chunk 7 Bank Analysis Pass

Date: 2026-04-13

## Summary

This pass validates the chunk-7 morph model directly against the decompressed bank data rather than only through executable-side code.

The generated reports are:

- [chunk-0007-bank-analysis.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/formats/atet-dat/chunk-0007-bank-analysis.json)
- [chunk-0007-bank-analysis.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/formats/atet-dat/chunk-0007-bank-analysis.md)

The strongest confirmations are:

- the decompressed size is exactly `0x38000`
- the bank split is exactly `8 * 0x7000`
- each bank is exactly `1024` records of `0x1c`
- source fields `+0x00/+0x04/+0x08` vary by bank as expected for the morph system
- trailing fields `+0x0c/+0x10/+0x14/+0x18` are all zero in the stored source banks

That last point is especially important.

It means the source chunk-7 banks do **not** ship with precomputed screen coordinates, shade bytes, or active flags.

Those really are runtime-generated fields written by the executable-side update and draw path.

## Runtime Field Map Is Now Stronger

The bank analysis shows every stored bank has:

- `+0x0c = 0`
- `+0x10 = 0`
- `+0x14 = 0`
- `+0x18 = 0`

across all `1024` records and all `8` banks.

That directly supports the current field model:

- `+0x00/+0x04/+0x08` are the true authored source data
- `+0x0c/+0x10/+0x14` are projected runtime outputs
- `+0x18` is a runtime draw-success flag

So the earlier source/runtime split is no longer just an inference from callers.

It is visible in the stored chunk data itself.

## The Banks Are Meaningfully Different

The pairwise delta analysis across the first three dwords shows that the banks are not just trivial reordered copies.

Some useful patterns:

- banks `5`, `6`, and `7` are much closer to each other than to the others
- banks `3` and `4` are also relatively close
- banks involving `1` against `5/6/7` are among the most distant pairs

That is good evidence that the frontend system morphs between genuinely distinct authored point sets rather than tiny perturbations of one master bank.

## Interesting Structural Pattern In Banks 5, 6, And 7

Banks `5`, `6`, and `7` have a striking property:

- their `+0x08` field is entirely zero across all records

So those banks appear flatter or less volumetric than banks `0` through `4`, where `+0x08` varies meaningfully.

That may mean:

- a subset of the authored banks are effectively 2D point sets
- or the third component has a specialized meaning that some banks simply do not use

Either way, it reinforces that the three source components should remain generically named until more of the animation semantics are resolved.

## Practical Impact On The Port

This makes the future faithful-port strategy clearer:

- chunk `7` should be preserved as authored source banks
- the Windows port should regenerate projected positions and shades at runtime
- we do not need to preserve the old incorrect panel-image interpretation

This also justifies the tooling cleanup:

- chunk-7 panel PNGs are obsolete heuristic artifacts
- chunk-7 should instead be represented in converted outputs as:
  - the decompressed bank set
  - the eight per-bank record blobs
  - the bank-analysis reports

## Recommended Next Move

The best next move is to inspect whether the clustered banks align with specific visible frontend states:

- for example, whether the flatter `5/6/7` bank family corresponds to one menu mood or transition family and the more volumetric `0..4` banks correspond to another.
