# Customer Support Ticket SLA System

Difficulty: Medium. Core topic: SLA tracking, injected clock.

The service-layer time round. First-response and resolution deadlines per priority, elapsed time measured from an injected clock, and a breach that must be strictly late — with the twist that closing a ticket freezes some breaches but not others.

## Scenario

You inherit the backend of a support ticket SLA system. Each priority maps to a policy with first-response and resolution limits; first response is recorded once on an open ticket; resolution happens once after creation; elapsed time is computed from creation using an injected `now`. The shape is right and the boundaries are wrong: breaches fire at the wrong instant, first response gets recorded more than once, and resolved tickets keep accruing breaches they should no longer be able to.

The core rule is deceptively precise: a breach is true only when elapsed time is strictly greater than the configured limit — not at it. And a resolved ticket keeps its historical first-response breach but can never newly accrue a resolution breach.

## Requirements

- Map each priority to its SLA policy (first-response and resolution limits).
- Record first response exactly once on an open ticket.
- Record resolution exactly once, after creation.
- Compute elapsed time from creation using the injected `now`, never a wall clock.
- Flag a breach only when elapsed time is strictly greater than the limit.
- A resolved ticket retains a historical first-response breach but cannot newly accrue a resolution breach.

## Edge cases to handle

- Elapsed time exactly equal to the limit (not a breach).
- A second first-response attempt on a ticket that already has one.
- Resolution recorded before any first response.
- Evaluating breaches on an already-resolved ticket.
- Different priorities resolving to different limits for the same elapsed time.

## What interviewers look for

Whether the service layer treats "strictly greater than" and "recorded once" as invariants it defends, not conditions it happens to check. A full-marks answer pins breach evaluation to the injected clock, makes first-response and resolution idempotent, and reasons clearly about which state transitions freeze which breaches — so a resolved ticket's history stays honest without letting it drift into new violations.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/customer-support-ticket-sla-system
