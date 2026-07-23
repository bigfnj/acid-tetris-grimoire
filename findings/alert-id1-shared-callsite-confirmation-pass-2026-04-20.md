# Alert ID 1 Shared Callsite Confirmation Pass

Date: 2026-04-20

## Summary

This pass closes the stale uncertainty around alert ID `1`.

Current best closure:

- alert ID `1` is **not** missing from the owned shipped direct `0x2008` trigger set
- the shared fast-follow-up callsite at `0x0f9f` reaches both alert IDs:
  - `1`
  - `10`
- only alert ID `13` remains absent from the owned shipped direct trigger set

## Why This Pass Was Needed

The earlier [alert-id13-usage-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/alert-id13-usage-closure-pass-2026-04-20.md) correctly closed alert ID `13`, but one supporting detail was too narrow:

- the direct callsite at `0x0f9f` was resolved only from the later immediate `mov eax, 0x0a`

That missed the earlier one-line fast-follow-up branch that jumps into the same callsite with:

- `EAX = 1`

So this pass revisits the shared callsite precisely enough to separate the two staging forms.

## Shared `0x0f9f` Callsite

In the line-clear reward block inside `0x09c8`, the code first dispatches the ordinary line-clear alert and sound, then checks whether the new clear arrived inside the fast-follow-up window:

- `current_tick - last_line_clear_tick < 0x00a0`

From there, the branch splits:

### One-line fast follow-up

When:

- `current_lines_cleared == 1`
- and the fast-follow-up window is still open

the code does:

- `mov edx, 0x00a0`
- `mov eax, edi`
- where `edi == current_lines_cleared == 1`
- `jmp 0x0f9f`

So the shared `0x0f9f` callsite reaches:

- alert ID `1`

### Two- or three-line fast follow-up

When:

- `current_lines_cleared < 4`
- and the fast-follow-up window is still open

the later branch does:

- `mov edx, 0x00a0`
- `mov eax, 0x0a`
- `call 0x2008`

So the same callsite also reaches:

- alert ID `10`

## Corrected Owned Shipped Direct Trigger Set

With the shared `0x0f9f` callsite resolved correctly, the owned shipped direct `0x2008` trigger set is:

- `{0,1,2,3,4,5,6,7,8,9,10,11,12}`

The only currently missing ID from that direct trigger set is:

- `13`

## What This Confirms

The older executable-derived label from [alert-id-correlation-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/alert-id-correlation-pass-2026-04-13.md) now stays on firmer ground:

- `1` -> chunk-`6` tile `01` -> fast follow-up one-line clear

That label is no longer just a visual or naming guess. It is now directly supported by the owned shipped call-flow into `0x2008`.

## Correction Propagated

This pass also corrects the machine-readable support data used by the alert-ID-`13` closure:

- [alert-id13-usage-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-id13-usage-closure.json)

That artifact now records:

- `0x0f9f` as a shared callsite reaching IDs `1` and `10`
- `13` as the only missing ID from the owned shipped direct trigger set

## Artifact

This pass adds:

- [alert-id1-shared-callsite-confirmation.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-id1-shared-callsite-confirmation.json)

## Port Implication

For a faithful first-pass port:

- keep alert ID `1` as a real shipped gameplay-side alert behavior
- model it as the one-line fast-follow-up branch
- keep alert ID `10` as the broader fast-follow-up two- or three-line branch
- keep alert ID `13` unbound unless new evidence appears

## What I Now Treat As Resolved

- alert ID `1` is a real shipped direct `0x2008` trigger
- the shared `0x0f9f` callsite must be modeled as a two-identity branch, not a single fixed alert ID
- alert ID `13` remains the only absent ID in the owned shipped direct trigger set

## Next Ordered Step

- pause the alert-ID trigger branch unless one of these becomes newly useful:
  - capture correlation for alert IDs `1` and `10`
  - a newly recovered indirect alert trigger path
  - author evidence naming the intended role of tile `13`
