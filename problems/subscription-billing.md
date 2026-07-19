# Subscription Billing and Proration Engine

Difficulty: Hard. Core topic: proration math, lifecycle state machines.

Where fintech interviews go to test money handling and state machines at once. A subscription sounds like a monthly charge; in practice it is a lifecycle — trial, active, past-due, cancelled — plus the most bug-prone computation in SaaS: proration, the credit-and-charge math of a customer changing plans mid-cycle.

## Scenario

You maintain a billing engine: subscriptions move through their lifecycle, invoices are generated per period, mid-cycle upgrades and downgrades prorate, and failed payments walk a dunning path with retries and grace. The engine produces numbers, nearly all of them wrong: upgrades charge full price instead of the prorated difference, downgrades mint negative invoices, regenerating a period bills the customer twice, cancelled subscriptions keep invoicing, and payment failures jump straight to cancelled with no dunning.

## Requirements

- Proration: credit the unused remainder of the old plan, charge the same window on the new plan, exact to the minor unit.
- Integer money math with rounding applied at defined points, once.
- Invoicing is idempotent, keyed by subscription and period: a retry returns the existing invoice.
- Lifecycle: trial, active, past-due, cancelled — legal transitions only.
- Payment failure enters dunning (scheduled retries, grace window); only exhausted retries plus expired grace may terminate.
- Billing cycles stay anchored across plan changes.

## Edge cases to handle

- A downgrade mid-cycle (must not produce a negative invoice)
- Invoice generation rerun after a crashed job or redelivered webhook
- Payment recovery mid-dunning (subscription restored cleanly)
- A plan change on the cycle boundary itself
- Cancelled subscriptions receiving late events

## What interviewers look for

Whether the math is done in integer minor units against day counts of the actual cycle — floating point and rounded intermediates are the standard wrong answers, and grading checks totals to the exact unit. And whether the clock is injected as a dependency: billing logic that reads wall time inline cannot be tested. Every bug in this problem has a real invoice attached.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/subscription-billing-and-proration-engine-coding-problem
