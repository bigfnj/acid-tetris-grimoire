# Line-Clear Helper Capture Coverage Pass

Date: 2026-04-20

## Summary

This pass closes one smaller but important question:

- can the owned captures actually tell us which of the ten level-selected line-clear helpers is visible on screen

Current best answer:

- the owned captures do confirm that line-clear-like debris is visibly real
- but they do **not** provide enough coverage to map individual captured moments to the ten helper families

So the helper-family capture-correlation branch should now be treated as:

- paused for insufficient capture coverage

not:

- an already-actionable mapping job waiting only on more interpretation

## Why This Pass Was Needed

[line-clear-theme-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-theme-pass-2026-04-13.md) left a reasonable follow-up open:

- correlate the ten line-clear helpers with captured gameplay footage

Later, [line-clear-style-source-closure-pass-2026-04-20.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/line-clear-style-source-closure-pass-2026-04-20.md) closed the asset-source question but explicitly left one remaining capture-side gap:

- which exact helper motion profile is visible in owned gameplay footage

Before continuing that branch, we needed to answer a simpler question first:

- do the owned captures actually support that correlation with enough fidelity

## What The Static Side Already Gives Us

The executable-side model is already strong:

- `0x1414` dispatches by:
  - `current_level % 10`
- the ten helper bodies are already bounded at:
  - `0x14b4`
  - `0x158c`
  - `0x1694`
  - `0x1770`
  - `0x1830`
  - `0x192c`
  - `0x19dc`
  - `0x1a9c`
  - `0x1b54`
  - `0x1c40`

And those helpers are already specific enough to describe as motion families:

- random or angled sprays
- mirrored bursts
- sweeping fans
- point-to-point convergence variants

So the static side is no longer the bottleneck.

The bottleneck is capture evidence.

## Owned Capture Coverage

### `07-gameplay.png`

- useful for steady gameplay layout
- useful for board / HUD / preview / piece-stat grounding
- **not** useful for row-clear helper identification

There is no active line-clear debris field in this screenshot.

### `gameplay-t15.png`

- useful as an early gameplay reference
- **not** useful for helper identity

No active debris burst is visible here.

### `gameplay-t45.png`

This is the most relevant owned still for the branch.

It does show:

- dense colored speckles below and around the board
- a gameplay alert face at the same moment

That makes it valuable evidence that line-clear-like debris is visibly real in the shipped game.

But it does **not** show enough to identify a helper family cleanly, because it is still only:

- one isolated frame
- with no preserved adjacent event sequence
- and no proven helper address or modulo-`10` level bucket tied to that image

Earlier alert-correlation work already treats this frame as likely a one-line-clear reward moment.
That is useful support for line-clear behavior in general, but it still does not select one helper out of the ten.

### `gameplay-t90.png`

- useful as a later steady gameplay reference
- **not** useful for helper identity

No active row-clear debris burst is visible here either.

### `gameplay.mkv`

This clip remains useful as a general behavioral reference.

But the currently preserved project artifacts do **not** give us:

- a helper-specific event timeline
- a controlled per-level sample set
- or a harvested frame sequence around the same clear event

So the video is potentially useful raw material, but not yet preserved in a way that can support a durable ten-helper mapping pass.

### `topout-exit-hiscore-menu.mkv`

This clip is high-value for:

- top-out
- game-over
- state `9`
- frontend bootstrap

It is **not** the right evidence source for the ten line-clear helpers.

## What The Owned Captures Can Prove

Current positive capture support:

- line-clear-like debris is visibly real
- that debris can be dense and spatially broad
- the static executable model is therefore not inventing a nonexistent subsystem

## What The Owned Captures Cannot Prove

Current unsupported claims:

- which helper address produced the `gameplay-t45` debris field
- which of the ten helper motion families appears in any given owned still
- a full helper-to-capture map across all `current_level % 10` buckets

The main blockers are structural:

- only one currently cited still clearly shows line-clear-like debris
- we do not have a preserved adjacent-frame sequence for that same clear event
- we do not have controlled level coverage across the ten modulo buckets

## Closure

The line-clear helper capture-correlation branch should now be treated as:

- capture-limited, not inference-limited

Closed answer:

- the owned captures support the existence of line-clear debris
- they do not support honest helper-by-helper mapping
- the ten-helper capture-correlation branch should pause until stronger capture evidence exists

## Port Implication

For the faithful port, we should preserve the line-clear helper family from the executable-side model first:

- keep the ten helper families distinct in structure
- keep their motion-class descriptions provisional but concrete
- do **not** overfit one still frame into a named helper mapping

That keeps the implementation honest without pretending the capture side is stronger than it is.

## Artifact

This pass adds:

- [line-clear-helper-capture-coverage.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/line-clear-helper-capture-coverage.json)

## What I Now Treat As Resolved

- helper-family capture mapping is not currently blocked by static RE
- it is blocked by capture coverage quality and preservation shape
- the current owned media is strong enough to confirm visible debris, but not strong enough to assign individual helpers

## Next Ordered Step

- pause the helper-family capture branch unless one of these becomes newly useful:
  - a line-clear-heavy controlled gameplay capture
  - adjacent-frame extraction around the same clear event with visible level context
  - a port-implementation task that only needs the static helper families, not capture-side naming
