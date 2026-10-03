# Duplicate Charges After a Consumer Restart

Difficulty: Hard. Core topic: at-least-once delivery, idempotent side effects.

The distributed-systems round. The queue guarantees delivery, not uniqueness. The consumer treats every delivery as new work, so a restart mid-window charges a batch of customers twice — and its own logs look perfect.

## Scenario

A payment consumer reads charge commands from a partitioned queue and applies each charge to a customer's balance. The queue may deliver a command again, and a repeat arrival carries a higher delivery-attempt number than the first, exactly as a real broker increments a delivery-count header when it hands the same work back.

During a recent incident a batch of customers were charged twice. The consumer process was restarted part-way through the affected window. Nothing in the consumer's own output suggests a problem: it logged every charge it applied, and its committed position advanced smoothly from the start of the run to the end, with no gaps and no rewinds.

The captured evidence holds the delivery trace, the ledger of charges applied, the record of committed positions, and the consumer's application log. Reconciling what the link delivered against what the consumer did is the shortest route to the defect.

## Requirements

- Apply each distinct command exactly once, however many times it is delivered.
- Keep that property across a process restart.
- End every customer's balance equal to the sum of their distinct commands.
- Keep the already-passing tests passing — they describe behaviour that must not regress.
- Do not modify the tests.

## Edge cases to handle

- A redelivery whose attempt number is higher than the original
- A duplicate arriving after the consumer restarted and lost in-memory state
- Commands for the same customer arriving across different partitions
- A crash between applying a charge and recording that it was applied
- Distinct commands that happen to have identical amounts

## What interviewers look for

Whether deduplication state is durable and keyed by command identity rather than kept in memory or inferred from the committed offset. A full-marks answer makes apply-and-record atomic, survives restart, and can explain why advancing an offset proves nothing about side effects.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/at-least-once-duplicate-side-effects
