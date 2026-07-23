# Alert ID 13 Usage Closure Pass

Date: 2026-04-20

## Summary

This pass closes the lingering "unused or unresolved" wording around alert ID `13`.

Current best closure:

- the shipped executable's direct `0x2008` alert-trigger set is now bounded cleanly
- the shared fast-follow-up callsite at `0x0f9f` reaches alert IDs `1` and `10`
- that bounded set does **not** include alert ID `13`
- chunk-`6` tile `13` therefore remains preserved art, but not a currently used shipped gameplay alert

## Why This Pass Was Needed

[alert-id-correlation-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/alert-id-correlation-pass-2026-04-13.md) already closed the direct ID-to-tile mapping strongly, but left one old hedge:

- `13` -> chunk-`6` tile `13` -> currently unused or still unresolved

By now, the alert trigger family is much tighter than it was on `2026-04-13`, so that wording can be revisited directly.

## Direct Trigger Set For `0x2008`

A direct `E8 rel32` scan against the flat relocated shipping image finds exactly `9` direct callsites to `0x2008`:

- `0x0ed3`
- `0x0f0c`
- `0x0f36`
- `0x0f9f`
- `0x1046`
- `0x10c7`
- `0x2221`
- `0x227f`
- `0x22ce`

These resolve to the currently owned gameplay-side trigger families:

- wake-from-sleepy one-line clear
- repeated back-to-back four-line reward
- generic line-clear alert table
- fast follow-up clear shared callsite
- top-out / game-over
- sleepy timeout
- high stack warning
- medium stack warning
- low stack warning

## Resolved Reachable Alert IDs

From those direct callsites, the reachable alert IDs are:

- direct immediates:
  - `12`
  - `8`
  - `1`
  - `10`
  - `6`
  - `11`
  - `5`
  - `4`
  - `3`
- line-clear table path:
  - `0`
  - `2`
  - `7`
  - `9`

So the owned shipped direct-trigger set is:

- `{0,1,2,3,4,5,6,7,8,9,10,11,12}`

And the missing IDs from that direct-trigger set are:

- `13`

This pass is only closing `13`.

## Why ID `13` Is Now Closed More Strongly

The alert system itself is not the unknown anymore:

- `0x2008` is the shared alert trigger
- `0x206c` is the runtime alert update path
- `0x1793d` / `0x17983` draw chunk-`6` tiles directly by `effect_id * 0x9c4`

That means the remaining question for `13` is simply:

- does any owned shipped path actually stage `EAX = 13` into `0x2008`

Current answer:

- no owned direct trigger path does

And because the recovered direct trigger family already covers the known gameplay contexts:

- line-clear rewards
- stack warnings
- sleepy timeout / wake-up
- top-out / game-over

the old wording is now stronger as:

- shipped but unused alert art

rather than:

- still unresolved shipped behavior

## Confidence Boundary

What is strong:

- chunk-`6` tile indexing is direct
- the shipping binary's direct `0x2008` caller family is bounded
- no bounded shipped direct caller reaches alert ID `13`

What is still not claimed:

- that no hypothetical unknown indirect or variant build could ever use tile `13`

So the closure is:

- unused in the owned shipped executable evidence
- preserved as an extra alert-tile art asset

## Port Implication

For a faithful first-pass port:

- keep tile `13` extracted and preserved
- do not bind alert ID `13` to any required gameplay event
- if surfaced in tooling or manifests, label it as shipped-but-unused preserved alert art

## Artifact

This pass adds:

- [alert-id13-usage-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/ghidra/alert-id13-usage-closure.json)

## What I Now Treat As Resolved

- alert ID `13` should no longer be carried as a live shipped-behavior question
- the owned shipping alert trigger set does not include `13`
- chunk-`6` tile `13` is preserved unused content under the current evidence set

## Next Ordered Step

- pause the alert-ID-`13` branch unless one of these becomes newly useful:
  - a recovered build that stages alert `13`
  - an indirect alert trigger path not present in the current owned direct caller family
  - new capture or author evidence naming the intended role of tile `13`
