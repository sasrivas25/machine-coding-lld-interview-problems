# Notification Preferences and Delivery System

Difficulty: Medium. Core topic: policy evaluation, idempotent delivery.

One of the most asked backend questions anywhere, because every product has a notification system and almost every implementation has shipped the same three bugs: a user notified on a channel they muted, a promotion delivered at 2 a.m., and the same message sent twice because a retry fired.

## Scenario

You maintain the notification service of a consumer backend: events come in, user preferences and quiet hours are consulted, and notifications go out on email, SMS, or push. Everything is wired and much of it misbehaves: muted channels still receive sends, quiet hours suppress critical alerts and let promotions through, redelivered events notify users twice, and suppressed notifications vanish without a trace instead of being recorded.

## Requirements

- Preferences: per-user, per-notification-type, per-channel opt-ins, enforced in one policy point.
- Quiet hours suppress delivery inside the user's do-not-disturb window — including windows that cross midnight.
- Critical-priority notifications legitimately pierce quiet hours.
- Delivery is idempotent: one notification per (user, event), retries included.
- Suppression is an outcome, not a silent drop: record why a notification did not go out.
- Multi-channel fan-out for a single event is deterministic.

## Edge cases to handle

- A 22:00–07:00 quiet window (a naive start < now < end check silently never matches)
- The same event redelivered by infrastructure
- A critical alert arriving inside quiet hours
- An event for a user who muted some channels but not others
- Time itself — the clock should be injectable so behaviour is testable

## What interviewers look for

Whether the three decisions — dedupe, preferences, quiet hours with the priority exception — compose in a fixed order in one place. Deduping last means a retried event re-runs the policy and can double-send; preferences after quiet hours records the wrong suppression reason. It looks like plumbing and grades like design.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/notification-preference-and-delivery-system-coding-problem
