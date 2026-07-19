# Feature Flag and Rollout Targeting System

Difficulty: Hard. Core topic: deterministic rule evaluation.

Feature flags are the control plane of modern deployment — every gradual rollout, A/B test, and 2 a.m. emergency shutoff runs through one. The question sounds administrative: given a user and a flag, is the feature on? The grading lives in properties that sound obvious and break constantly.

## Scenario

You maintain a feature-flag service: flags carry targeting rules (user lists, attributes, percentage rollouts) per environment, evaluation answers on/off per user, and a kill switch exists for emergencies. It returns plausible answers that are wrong in every dimension that matters: the same user flips between on and off across requests at a fixed percentage, growing a rollout un-enrolls users it already admitted, rules fire out of priority order, staging configuration leaks into production evaluations, and the kill switch loses to a specific-targeting rule.

## Requirements

- Determinism: a user at 30% rollout gets the same answer on every request.
- Monotonic rollouts: growing 30% → 50% keeps every previously enrolled user enrolled.
- Rules evaluate in priority order with first-match semantics.
- The kill switch beats everything, unconditionally — nothing overrides it.
- Environments are isolated: a flag enabled in staging means nothing in production.
- A defined default when no rule matches.

## Edge cases to handle

- The same user evaluated repeatedly at a fixed percentage
- A rollout percentage being raised, then lowered
- A user matching both a targeting rule and the percentage gate
- Kill switch flipped while specific-targeting rules would say "on"
- Two flags at the same percentage (must not enroll the identical user set)

## What interviewers look for

Deriving the rollout decision from a pure function, not a random draw: hash a stable key (user + flag) into a bucket and compare against the percentage — determinism and monotonicity fall out of that one choice. Most bugs here are ordering bugs; the evaluation pipeline (kill switch, then rules by priority, then percentage, then default) must be explicit and untouchable.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/feature-flag-and-rollout-targeting-system-coding-problem
