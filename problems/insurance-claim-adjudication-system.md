# Insurance Claim Adjudication System

Difficulty: Hard. Core topic: coverage math, approval authority.

The claims-backend round. Coverage windows, deductibles, adjuster limits, and a coverage balance that has to decrement exactly once per approved claim. The shape is there; the money math and the authority rules are quietly wrong.

## Scenario

You are working on a partially implemented insurance claim adjudication backend. Policies have coverage windows and per-incident eligibility; approved claims apply a deductible and draw down a coverage limit; adjusters have approval authority with escalation above their limit. Several visible tests fail because some repository and service logic is incomplete or incorrect — incident-date eligibility is mis-checked, deductible and limit math drift, approval limits aren't enforced, and remaining coverage doesn't stay consistent with payouts.

Your task is to read the models, repositories, services, and visible tests, infer the intended behaviour, and fix the implementation. Do not rewrite from scratch, do not change public method contracts, and do not modify the tests.

## Requirements

- Coverage applies only when the incident date falls inside the policy window.
- Deductible and coverage-limit math compute correct payouts on approved claims.
- Adjuster approval authority is enforced, with escalation when a claim exceeds a limit.
- Remaining coverage decrements exactly once per approved claim.
- Payouts and remaining coverage stay in balance across a sequence of claims.
- Existing public contracts and tests remain untouched.

## Edge cases to handle

- An incident dated just outside the coverage window
- A claim smaller than, equal to, or larger than the deductible
- A claim that exhausts the remaining coverage limit
- A claim above an adjuster's authority requiring escalation
- Repeated processing that must not decrement coverage twice

## What interviewers look for

Whether the candidate separates eligibility, computation, and authority instead of tangling them, and keeps the coverage balance as an invariant that reconciles with the sum of payouts. Full marks show the one-time decrement handled deliberately so re-adjudication or retries can't double-spend a policy's limit, and approval rules enforced as data rather than scattered conditionals.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/insurance-claim-adjudication-system
