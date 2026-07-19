# Loyalty Points and Tier Management System

Difficulty: Medium. Core topic: FIFO accounting and idempotency.

The rewards-program / super-coins style problem, common at e-commerce and fintech companies. It looks like simple arithmetic on a balance, and that is the trap: points arrive in lots with their own expiry dates, redemption must consume the oldest lots first, and tiers must be recalculated from what a member actually holds right now.

## Scenario

You maintain the backend of a loyalty program. Members earn points from purchase events, redeem them for rewards, and hold a tier derived from their activity. The happy path looks plausible, but expired lots still get spent, redemption drains newer points while older ones silently expire, replayed earn events double-credit members, and tier upgrades lag or overshoot after expiry.

## Requirements

- Points are earned in lots, each with an amount and an expiry date.
- Balance queries exclude expired lots, consistently, everywhere.
- Redemption consumes the oldest live lots first, spanning partial lots where needed, and fails atomically if the live balance is short.
- Earn events are idempotent: a replayed event credits exactly once and returns the original outcome.
- Tier is recalculated from current live state after earns, redemptions, and expiry.
- Balances and histories are deterministic.

## Edge cases to handle

- Redemption that partially consumes a lot
- A lot expiring between a balance check and a redemption
- The same earn event delivered twice (infrastructure retry)
- Tier boundaries crossed downward by expiry, not just upward by earning
- Redemption amount exactly equal to the live balance

## What interviewers look for

Modelling the member as a queue of dated lots rather than one mutable integer. Every read — balance, redemption capacity, tier input — derives from live lots at that moment, so expiry becomes a filter, not a background job to keep consistent. A single mutable balance fails the round the moment expiry or replayed events appear.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/loyalty-points-and-tier-management-system-coding-problem
