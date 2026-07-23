# Frontend Object Cycle Continuity Pass

Date: 2026-04-14

## Summary

This pass closes an important frontend behavior question:

- the chunk-7 floating-object system is not re-seeded per menu
- it is a single shared cycle that begins at startup and keeps running across frontend states

That is a meaningful fidelity detail for the future port.

It means the menu smiley and tetromino objects should feel like one continuously evolving decorative system, not a fresh animation attached to each screen.

## The Key Bank-Selection Rule In `0x39c4`

At the top of `0x39c4`, the code checks the cycle counter at:

- `0x1990b`

When it reaches `0x258`, the logic is:

1. compare `next_bank` at `0x2d24b` against `current_bank` at `0x2d233`
2. if they are equal, choose a fresh random bank modulo `8`
3. repeat until the new `next_bank` differs from `current_bank`
4. call `0x3c70` to build the delta table for the upcoming morph

If `next_bank` is already different, `0x39c4` does **not** roll a new bank.
It just uses the bank that was already staged.

That explains the startup anchor behavior cleanly.

## What Startup Actually Seeds

`0x5ef4` initializes:

- the projection/drift state
- `next_bank = 0`
- `current_bank = random(0..7)`

That ordering matters.

Because `next_bank` is forced to `0` before the random current bank is chosen:

- if startup current bank is non-zero, the first long morph targets bank `0`
- if startup current bank is already `0`, then the first `0x258` threshold forces a new random different bank

So the "bank 0 anchor" is real, but specifically as a startup consequence, not as a permanent rule for every later transition.

## The Cycle Is Shared Across Frontend States

The strongest evidence here is writer ownership.

Current direct writes found in the relocated flat binary:

- `0x5ef4` writes:
  - `0x2d233` current bank
  - `0x2d24b` next bank
- `0x39c4` writes:
  - `0x2d233` when a morph commits
  - `0x2d24b` when it needs a fresh random target
  - `0x1990b` for increment/reset

No other currently observed frontend handlers write those three state fields.

That means:

- main menu
- options
- keyboard setup
- sound setup
- credits
- high-score presentation

all appear to share the same ongoing object-cycle state rather than resetting it when a new screen begins.

## Why This Matters Visually

This explains why the menu objects feel like a living shared background system instead of a per-screen effect.

The player is not seeing:

- open menu
- start local smiley/tetromino animation from frame zero

They are seeing:

- open menu
- inherit the current global chunk-7 object state
- continue advancing it with the shared fixed-step update path

That is a more specific and more faithful model.

## Timing Model

The object cycle now reads like this:

1. startup seeds transform state and chooses a random current bank
2. the object system projects that bank for `0x258` logical steps
3. it morphs for `0x80` logical steps
4. the next bank commits as current
5. a fresh random target is only chosen when needed
6. all frontend handlers keep advancing the same shared state through `0x39c4`

Combined with the earlier catch-up finding, this means the frontend object subsystem is:

- global
- fixed-step
- continuous across menu states

## Porting Impact

For a faithful port, the chunk-7 object system should be modeled as one shared frontend subsystem with persistent state:

- current bank
- next bank
- cycle counter
- transform angles
- drift phases
- origin accumulators

We should **not** restart that state whenever the user enters Options, Keyboard Setup, Credits, or High Scores unless later reverse engineering proves a specific exception.

## Bottom Line

This pass sharpens the frontend fidelity target:

- chunk-7 menu objects are a shared continuous frontend cycle
- startup seeds the first bank and the initial bank-0 anchor behavior
- later menu screens inherit and keep advancing the same object state

That is exactly the kind of subtle behavior worth preserving in a crisp restoration.
