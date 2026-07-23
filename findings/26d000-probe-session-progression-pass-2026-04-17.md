# `0x26D000` Probe-Session Progression Pass

Date: 2026-04-17

## Summary

This pass followed the DOSBox-environment freshness result:

- fresh clone trees were not enough to restore the Shape B bridge deterministically
- fresh `HOME` and `XDG_*` roots were also not enough
- the strongest remaining model was broader probe-session progression

The narrow question here was:

- if we hold the short Shape B lane fixed and run it across several otherwise identical fresh-clone probe sessions, does the signal decay monotonically by session number

The lane stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I ran four separate single-clone sessions against four fresh disposable clones.

Main result:

- the sequence did not collapse monotonically
- object-`1` families reappeared after a fully flat session
- the newest high-value signal was a bridge-side object-`1` stop at `0824:00001003` in session 4

So the best new closure is:

- simple session order alone does not explain the signal
- the external state we have not yet isolated can still reopen bridge-adjacent object-`1` families later in the sequence

## New Owned Artifacts

- [26d000-probe-session-progression-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-probe-session-progression/26d000-probe-session-progression-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T233306Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T233458Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T233650Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T233841Z/batch-summary.json)

Fixture root:

- [shape-b-session-progression-20260417T233248Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-session-progression-20260417T233248Z)

## Harness Preset

This pass reused:

- `selector-26d000-shape-b-env-control-single-clone`

No harness code changes were needed.

## Results

### Session 1

- `no-inspect` -> `0868:0002B853`
- `no-inspect-again` -> `0008:00001DFC`
- `bridge` -> `0868:00023979`

### Session 2

- `no-inspect` -> `0868:0002B81E`
- `no-inspect-again` -> `0868:0002B826`
- `bridge` -> `0868:000247FB`

### Session 3

- `no-inspect` -> `0868:00024816`
- `no-inspect-again` -> `0008:00001926`
- `bridge` -> `vrt_selector_not_discovered`

### Session 4

- `no-inspect` -> `post_vrt_rebreak_not_observed`
- `no-inspect-again` -> `0868:00023923`
- `bridge` -> `0824:00001003`

### File Diffs

Clone tree hash diffs:

- `session-1-clone` -> `0`
- `session-2-clone` -> `0`
- `session-3-clone` -> `0`
- `session-4-clone` -> `0`

So this pass again observed no writes into the mounted game trees.

## Findings

### 1. Session Progression Was Not Monotonic

This is the main closure.

If a simple earliest-session-only decay model were correct, the strongest signal should have appeared in session 1 and then steadily collapsed.

That did not happen.

Session 2 flattened completely to `0868`, but session 3 still brought back:

- `0008` at the second `no-inspect`

and session 4 brought back:

- an object-`1` bridge-side stop at `0824:00001003`

So session number alone is not enough to explain the drift.

### 2. The Strongest Repeated Pre-Bridge Signal Is Still The Second `no-inspect` Slot

This remains the most repeatable object-`1` family in the pass.

The second `no-inspect` reached:

- `0008:00001DFC` in session 1
- `0008:00001926` in session 3

That means the pre-bridge leak is still real even after a fully flat session 2.

### 3. The New Highest-Value Signal Is A Bridge-Side `0824` Object-`1` Stop

This is the most important positive result.

Session 4 bridge did not flatten to `0868`.

Instead it stopped at:

- `0824:00001003`

That is not yet the desired earlier object-`2` entry, but it is bridge-side and still in object `1`, which makes it more valuable than another second-`no-inspect` replay.

### 4. Two Runs Failed Earlier In The Probe Flow Rather Than Flattening Cleanly

This pass also surfaced two probe-flow anomalies:

- session 3 bridge -> `vrt_selector_not_discovered`
- session 4 opening `no-inspect` -> `post_vrt_rebreak_not_observed`

Those are not the same as a normal flat `0868` result. They imply that some part of the runtime state is still shifting before the usual late-family capture stabilizes.

### 5. The Differentiating State Still Is Not In The Clone Trees

As in the earlier fresh-clone and env-freshness passes, all clone hash diffs stayed empty.

So this pass again rules out:

- clone-local on-disk mutation

as the explanation for the changing behavior.

## Practical Interpretation

This pass materially changes the session-progression model.

We now know:

- the signal does not simply decay by session index
- object-`1` families can reappear after a fully flat session
- and bridge-side object-`1` is still reachable, even if only intermittently

So the best next move is no longer a generic “later sessions vs earlier sessions” repeat. The stronger target is the new bridge-side `0824` lead itself.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Bridge-Side `0824` Reproduction Pass**

Focus:

- keep the short Shape B lane fixed
- target the bridge slot specifically
- add bridge-side selector capture whenever the bridge does not flatten cleanly
- compare whether `0824` bridge-side object-`1` can be reproduced without relying on a specific session index

## Bottom Line

The short Shape B lane did not decay monotonically across four fresh-clone probe sessions.

Object-`1` families reappeared after a fully flat session, and the most valuable new signal was a bridge-side stop at `0824:00001003` in session 4. The next best move is to treat that bridge-side `0824` family as the new direct target.
