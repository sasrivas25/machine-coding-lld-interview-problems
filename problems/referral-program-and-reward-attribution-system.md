# Referral Program and Reward Attribution System

Difficulty: Hard. Core topic: attribution, idempotency.

The reward-fraud round. A code gets issued, a referee is attributed to it, a qualifying event fires — maybe twice — and exactly one reward must land for that user. Miss an ownership check or an attribution window and the program pays for self-referrals and replays.

## Scenario

You inherit a partially implemented referral program and reward attribution backend. The models, repositories, and services exist, and the visible tests describe issuing referral codes, attributing referees, a time-bounded attribution window per code, a pending-to-qualified-to-rewarded lifecycle, idempotent qualifying events, and exactly one reward per referred user.

The behavior leaks in the ways a referral program gets abused. A code owner can be attributed to their own code, attributions land outside the window they should respect, a repeated qualifying event moves the referral forward more than once, and a referee can be rewarded twice. Several visible tests fail until the validation, ownership rules, state transitions, and repeat-safety are correct.

## Requirements

- Issue referral codes and attribute referees to them.
- Prevent a code owner from referring themselves.
- Enforce a time-bounded attribution window tied to each code.
- Move a referral through pending, qualified, and rewarded in order.
- Make qualifying events idempotent — a repeat does not advance the referral again.
- Grant exactly one reward per referred user.

## Edge cases to handle

- An owner attempting to be attributed to their own code
- An attribution arriving just after the window closes
- The same qualifying event delivered more than once
- A second reward attempt for an already-rewarded referee
- A qualifying event for a referral that was never attributed

## What interviewers look for

Whether ownership and window checks gate attribution before any state moves, and whether qualifying events are idempotent so a replay cannot re-advance the lifecycle or double-pay. Full marks require the one-reward-per-referee guarantee to hold under repeated events, not just on the happy path.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/referral-program-and-reward-attribution-system