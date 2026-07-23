# `0x26D000` DOSBox-Environment Freshness Pass

Date: 2026-04-17

## Summary

This pass followed the fresh-clone order-sensitivity result:

- simple clone position did not reopen the Shape B bridge
- the remaining live signal had retreated to the second `no-inspect` step in the earliest batch only

The narrow question here was:

- does changing only DOSBox-side environment state such as `HOME` and `XDG_*` reopen the early Shape B bridge families

The short Shape B lane stayed fixed:

- `no-inspect -> no-inspect -> inspect`

I compared four scenarios:

- default environment + fresh clone
- fresh env-A + fresh clone
- reused env-A + second fresh clone
- fresh env-B + fresh clone

Main result:

- every bridge probe stayed flat `0868`
- the strongest live signal again appeared one slot earlier, and only in the earliest scenarios of the pass

So the best new closure is:

- DOSBox env freshness alone does not restore the bridge
- the remaining unexplained signal looks more like broader process or session progression

## New Owned Artifacts

- [26d000-dosbox-environment-freshness-results-2026-04-17.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/26d000-dosbox-environment-freshness/26d000-dosbox-environment-freshness-results-2026-04-17.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T232230Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T232424Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T232616Z/batch-summary.json)
- [batch-summary.json](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-trace/branch-family-batch/20260417T232812Z/batch-summary.json)

Fixture root:

- [shape-b-env-freshness-20260417T224855Z](/home/bigfnj/projects/@Project-Tetris/Decompilation.Effort/research/runtime-fixtures/shape-b-env-freshness-20260417T224855Z)

## Harness Preset

This pass added and exercised:

- `selector-26d000-shape-b-env-control-single-clone`

## Results

### Default Environment + Fresh Clone

- `no-inspect` -> `0868:00023A85`
- `no-inspect-again` -> `0008:00001491`
- `bridge` -> `0868:000258CB`

### Fresh Env-A + Fresh Clone

- `no-inspect` -> `0008:00001E7E`
- `no-inspect-again` -> `0008:0000191C`
- `bridge` -> `0868:0002B880`

### Reused Env-A + Second Fresh Clone

- `no-inspect` -> `0868:0002B517`
- `no-inspect-again` -> `0868:00023A23`
- `bridge` -> `0868:00025389`

### Fresh Env-B + Fresh Clone

- `no-inspect` -> `0868:0002481E`
- `no-inspect-again` -> `0868:00023AC8`
- `bridge` -> `0868:00023AC8`

### File Diffs

Clone tree hash diffs:

- default-clone -> `0`
- env-a-first-clone -> `0`
- env-a-second-clone -> `0`
- env-b-clone -> `0`

Disposable env-root file diffs:

- env-a -> `0`
- env-b -> `0`

So this pass observed no writes into either:

- the mounted clone trees
- or the disposable `HOME` / `XDG_*` roots

## Findings

### 1. DOSBox Env Freshness Did Not Reopen The Bridge

This is the main negative closure.

All four bridge probes stayed flat `0868`.

So varying:

- `HOME`
- `XDG_CONFIG_HOME`
- `XDG_CACHE_HOME`

was not sufficient to bring back:

- `0008`
- or `0870`

at the bridge itself.

### 2. The Earliest Scenarios Still Leaked Object-`1` Families One Slot Earlier

This is the strongest positive signal in the pass.

The default environment reached:

- `0008` at the second `no-inspect`

Fresh env-A reached:

- `0008` at the opening `no-inspect`
- `0008` again at the second `no-inspect`

But those earlier object-`1` families still did not survive into the bridge.

### 3. Reusing Env-A Collapsed, But A Separate Fresh Env-B Also Collapsed

This is the key comparison.

If env freshness alone were the deciding factor, the two fresh env roots should have behaved similarly.

They did not:

- fresh env-A showed the strongest pre-bridge live signal
- fresh env-B collapsed fully

So the observed effect is not explained by:

- “fresh env root good, reused env root bad”

by itself.

### 4. Nothing Was Written Into The Controlled Workspace Trees

This closes another candidate.

There were no observed file changes in:

- the clone trees
- the disposable env-A root
- the disposable env-B root

So the differing behavior is still not being persisted into the workspace-controlled file trees we measured.

### 5. The Strongest Remaining Model Is Broader Session Progression

At this point we have ruled out:

- in-tree file mutation
- simple clone slot order
- simple `HOME` / `XDG_*` freshness

The remaining signal pattern fits best with:

- broader process or session progression outside those controlled roots

This is an inference from the evidence, not a direct proof.

## Practical Interpretation

This pass moved the boundary again.

We now know:

- DOSBox env roots are not the main missing reset lever
- but the earliest scenarios of a pass can still surface object-`1` families one slot earlier
- and none of the differing behavior is being written into the workspace-controlled trees

So the next pass should focus on progression across whole probe sessions rather than inside clone trees or env roots.

## Recommended Next Move

The strongest next move is now:

- **`0x26D000` Probe-Session Progression Pass**

Focus:

- keep the short Shape B lane fixed
- compare otherwise identical single-clone runs at:
  - the first probe session after a clean restart point
  - the second
  - the third
- record whether the live signal begins at:
  - bridge
  - second `no-inspect`
  - or collapses immediately to `0868`

## Bottom Line

Changing `HOME` and `XDG_*` roots did not reopen the Shape B bridge.

The strongest live signal again appeared only one slot earlier and only in the earliest scenarios of the pass, while all bridge probes stayed flat `0868`. The best remaining explanation is now broader session progression outside the clone tree and outside the disposable DOSBox env roots.
