# Incident Debugging: Poison Message Kills the Queue Worker

Difficulty: Medium. Core topic: poison messages, dead-lettering.

The 3 AM incident round. One malformed message throws out of the run loop, the worker dies mid-batch, the supervisor restarts it onto the same message, and queue depth climbs past its alarm — a textbook poison-message crash loop you have to read the logs to unwind.

## Scenario

PagerDuty fired at 03:41 UTC: the `billing-events` ingest worker is down and queue depth is climbing. On-call notes and the captured production log sit in `logs/incident.log` inside each language directory — that is where you start. The worker is a small, fully in-process project: an injected in-memory queue, a `Worker` whose `run(queue)` pulls each message and `process()`es it (parse a `key=value;key=value` payload, apply a `set`/`incr` update to an in-memory store), and an injected dead-letter queue the runbook says poison messages belong in — but nothing ever lands there.

The bundled tests reproduce the incident deterministically: a batch of valid messages interleaved with several poison ones. On `main` they FAIL the way production did — the first poison message throws out of `run()`, the worker dies, valid messages behind it are never processed, and the DLQ stays empty.

## Requirements

- `run()` completes without throwing even when poison messages are present.
- Every valid message is processed exactly once, in order.
- Each poison message is routed to the dead-letter queue with a classified reason: `malformed_payload`, `missing_field`, `invalid_value`, or `unknown_op`.
- The queue is fully drained regardless of how many poison messages appear.
- An empty queue is a clean no-op.

## Edge cases to handle

- A poison message as the very first item in the batch.
- Consecutive poison messages back to back.
- A payload that parses but carries a non-integer value or an unknown op.
- A message missing a required field.
- Draining to completion after the last poison message.

## What interviewers look for

Whether error handling is scoped to the unit of work, not the loop. A full-marks answer isolates each message so one failure cannot abort the batch, classifies the failure precisely enough to dead-letter it with a reason, and keeps the store consistent — valid updates applied, poison ones quarantined. The instinct to read `logs/incident.log` before touching code, and to make the fix reproducible against the captured incident, is exactly what the round is testing.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-poison-message-worker-debugging