# Title/Options Correlation Closure Pass

Date: 2026-04-20

## Summary

This pass closes the long-running interpretation question around the earlier title/options capture-correlation result:

- bank `3`
- progress `64`

Current best answer:

- `bank 3 / progress 64` remains a useful historical midpoint heuristic from the early full-frame scoring passes
- it should **not** be treated as the canonical recovered title/options frontend state

The later dynamic-sequence and motion artifacts are now strong enough to tighten that reading.

The title/options capture-correlation branch should therefore be treated as:

- paused for model limitations

not:

- still waiting for one more round of scoring to certify `bank 3 / 64`

## Why This Pass Was Needed

[frontend-capture-correlation-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-capture-correlation-pass-2026-04-13.md) and [title-options-frame-correlation-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/title-options-frame-correlation-pass-2026-04-13.md) gave a real early signal:

- captures often preferred mid-morph title-overlay renders
- all six title/options frames preferred `bank 3 / progress 64`
- second place was consistently `bank 2 / progress 32`

That was useful at the time, but both passes left one explicit gap open:

- why does `bank 3 / progress 64` keep winning

Later work materially changed the evidence base, so we needed to close that older reading instead of carrying it forward as if it were still the active interpretation.

## What The Earlier Passes Really Established

The early passes did establish two durable points:

- the title/options captures do not prefer the wide steady banks `5..7`
- the captures correlate better with the compact morph-family than with a single settled steady bank

Those parts still stand.

What no longer stands as the best interpretation is the stronger leap from:

- "`bank 3 / progress 64` wins the older whole-frame ranking"

to:

- "`bank 3 / progress 64` is probably the real recovered frontend state"

## Later Evidence That Supersedes The Stronger Reading

### Dynamic-Sequence Correction

[title-options-dynamic-sequence-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/title-options-dynamic-sequence-pass-2026-04-13.md) changed the interpretation in an important way.

Once real dynamic renders were added, the raw best matches became:

- `bank 2 -> 0`
- progress `0..4`
- `drawn_count = 0`

Those are effectively pure title-screen base matches, not informative pointfield matches.

So the raw whole-frame winner is now best understood as:

- the captured title/options frames are strongly dominated by the base title screen

not:

- we have identified one specific meaningful pointfield state

### Visible-Only Ranking

When the correlation is restricted to renders that still show a visibly present pointfield, the best matches cluster around:

- `bank 3 -> 0`, progress `21..23`
- `bank 2 -> 0`, progress `103`

Per capture, the top visible winners are now:

- `title-options-01` -> `bank 3 / progress 23`
- `title-options-02` -> `bank 3 / progress 23`
- `title-options-03` -> `bank 2 / progress 103`
- `title-options-t15` -> `bank 3 / progress 23`
- `title-options-t60` -> `bank 2 / progress 103`
- `title-options-t135` -> `bank 3 / progress 23`

That is a tighter and more honest narrowing than the older `bank 3 / 64` midpoint story.

### Motion Evidence

[frontend-transition-motion-pass-2026-04-13.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-transition-motion-pass-2026-04-13.md) tightened the remaining gap further.

The real aligned title/options capture footprint is:

- capture delta union: `12571` pixels, bbox `(0,3) -> (297,173)`
- temporal motion union: `7452` pixels, bbox `(0,59) -> (124,173)`

But the best current chunk-7 motion candidate is only:

- `bank 2 -> 0`, progress `109`
- render bbox `(0,0) -> (29,60)`
- overlap `27` pixels
- motion Jaccard `0.003586`

That is far too weak to justify a strong "we recovered the exact visible title/options object state" claim.

So the motion evidence now points in the same direction as the dynamic-sequence correction:

- the current model still does not spatially explain the title/options captures well enough

## Why The Older Whole-Frame Ranking Misled

Later frontend-state work now gives a concrete reason to be cautious with whole-frame RGB scoring.

[frontend-title-base-loader-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-title-base-loader-pass-2026-04-14.md) shows:

- the title/logo base screen is preloaded from chunk `4`

[cold-boot-menu-and-held-input-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/cold-boot-menu-and-held-input-pass-2026-04-14.md) shows:

- cold boot reveals the title/logo backdrop first
- floating chunk-7 objects come over that backdrop
- menu rows reveal afterward

[frontend-menu-layering-pass-2026-04-14.md](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/docs/findings/frontend-menu-layering-pass-2026-04-14.md) shows:

- menu text and highlight redraw have visual priority over the floating objects
- the object draw is empty-pixel-only

Taken together, those later passes explain why early whole-frame similarity was never enough:

- static title/logo art is a large constant term
- text and highlight redraw can dominate visible differences
- current chunk-7 reconstruction can still be directionally correct while remaining spatially incomplete for the capture set

## Closure

The current best reading is now:

- `bank 3 / progress 64` is a useful historical midpoint heuristic from the earlier scoring model
- it is **not** the canonical recovered title/options frontend state
- the branch should pause on the capture side until the presentation model changes or stronger extracted capture sequences exist

The honest closure is therefore:

- title/options capture correlation is now model-limited, not merely under-scored

## Port Implication

For the future faithful port, we should preserve the frontend behavior that the executable side now supports strongly:

- preloaded title/logo base
- continuous chunk-7 object system
- backdrop-first then row-reveal startup rhythm
- text/highlight visual priority over floating objects

We should **not** hardcode one specific title/options object frame such as `bank 3 / progress 64` as if it were now proven ground truth.

## Artifact

This pass adds:

- [title-options-correlation-closure.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/capture-correlation/title-options-correlation-closure.json)

## What I Now Treat As Resolved

- the older `bank 3 / progress 64` result still belongs in the project history, but only as an early heuristic
- later dynamic and motion artifacts supersede it as the active interpretation
- current title/options capture correlation should pause until:
  - the render model changes materially
  - new frame extraction isolates object motion better
  - or a later port task only needs the executable-side frontend truths rather than a certified capture-state match

## Next Ordered Step

- pause the title/options capture-correlation branch unless one of these becomes newly useful:
  - a materially improved chunk-7 presentation model
  - tighter frame extraction around one title/options motion interval
  - a port-side decision that only needs the already-owned frontend layering and sequencing rules
