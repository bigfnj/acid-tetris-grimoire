# Title/Options Screenshot Certification Pass

Date: 2026-04-21

## Summary

This pass re-ran the corrected 2026-04-21 menu-composite branch against the cleaner screenshot-alignment track instead of the noisier `title-options` desktop-video frames.

Main result:

- the options screenshot is now specific enough to certify one real frontend state
- the title screenshot is still **not** specific enough to certify the main-menu state or selected-row pulse

So this branch advanced, but did not fully close.

The practical narrowing is:

- `Options` now has screenshot-backed certification support
- `Main Menu` still needs a better certification source

## Artifacts Used

- [frontend-alignment/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-alignment/manifest.json)
- [frontend-menu-pulse-correlation-screenshots-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-correlation-screenshots-2026-04-21/manifest.json)
- [frontend-menu-pulse-delta-screenshots-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/frontend-menu-pulse-delta-screenshots-2026-04-21/manifest.json)
- [title-options-motion-screenshots-2026-04-21/manifest.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-motion-screenshots-2026-04-21/manifest.json)
- [title-options-composite-boundary-correction-pass-2026-04-21.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/title-options-composite-boundary-correction-pass-2026-04-21.md)

## Findings

### 1. The `Options` Screenshot Now Positively Fits One Corrected Menu State

For the aligned screenshot:

- `frontend-alignment/02-options.aligned-320x240.png`

the corrected full-frame menu correlation now prefers:

- `options-menu`
- selected row `1`
- selected text `Sound FX Volume:90`

The delta-only pulse pass also turns positive instead of merely "least bad":

- best masked improvement: `+29.54646`
- best phase family: `1376` or `1696`
- reveal amount: `20`

That is strong enough to treat the screenshot as:

- real support for the options screen
- real support for the second row being selected

It does **not** prove that the exact pulse phase is unique.
But it does move the screenshot beyond "generic title-base dominated frontend material."

### 2. The `Title` Screenshot Still Does Not Certify The Main Menu Honestly

For the aligned screenshot:

- `frontend-alignment/01-title.aligned-320x240.png`

the corrected full-frame pass still prefers:

- `options-menu`

not:

- `main-menu`

And the pulse-delta branch is still negative across the board.

Current least-bad readings are:

- `options-menu row 1` with masked improvement `-17.039621`
- `main-menu row 3` (`Level 0`) with masked improvement `-18.5`

So the title screenshot still behaves like the earlier video-frame branch:

- title-base dominance remains stronger than the menu-specific certification signal

### 3. The Screenshot Motion Branch Does Not Rescue Chunk-7 Spatial Certification

The screenshot-only motion artifact's best Jaccard scores stay small:

- top motion Jaccard: `0.018734`
- top delta Jaccard: `0.01847`

That is useful as a weak spatial sanity check.
It is not strong enough to re-open exact chunk-7 object-placement certification.

## Closure

This pass improves the branch from:

- generic title/options capture-certification wish

to:

- screenshot-backed certification for `Options`
- still-open certification for `Main Menu`

So the remaining title/options capture ask is now narrower:

- we no longer need to speak about the whole title/options family as equally uncertified
- the main unresolved screenshot-certification target is the title or main-menu state specifically

## Port Implication

For subsystem porting:

- treat the executable-owned options row family and selected-row behavior as supported by the screenshot branch
- keep the main-menu presentation model implementation-safe from executable ownership
- do **not** overclaim screenshot-certified main-menu pulse placement yet

## Next Ordered Step

If this branch is revisited later, the highest-value new input is:

- a cleaner main-menu-focused capture source
- ideally a native-resolution or otherwise better-cropped frame than the existing desktop screenshot
