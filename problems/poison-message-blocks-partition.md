# One Bad Record Freezes a Partition

Difficulty: Hard. Core topic: poison messages, retry budgets, dead-lettering.

The distributed-systems round. A record the downstream will never accept is retried forever, and because the dispatcher advances only when the current record settles, everything behind it in that partition stops.

## Scenario

A dispatcher consumes an ordered stream split across three partitions. Records within a partition must be applied in order, so the dispatcher works on one record at a time per partition and advances only when the current record is settled. For each record it calls a downstream ledger service and decides what to do next: settle it and move on, ask for it again after a short backoff, or set it aside and move on. Rejections are routine and most succeed on a later attempt.

A producer then emitted a record the ledger service will never accept, because its schema version is not recognised. No number of further attempts will change that answer.

Two of the three partitions drained normally. The third stopped making progress, and the records queued behind the unacceptable one were never applied. The dispatcher raised no alarm and was never idle — from its own point of view it was busy the whole time.

The evidence contains every record offered, every record actually processed, the dispatcher's log, and a per-partition summary of how far each advanced. Counting log lines per record is the shortest route to the defect.

## Requirements

- Make a single unacceptable record cost only that record.
- Keep applying the records behind it.
- End every record either applied or set aside — never neither.
- Keep anything given up on retrievable, with its payload.
- Keep retrying working: a record rejected twice that would be accepted on a third attempt must still be accepted.
- Keep no partition stalled past the retry budget.

## Edge cases to handle

- A permanent rejection versus a transient one
- The attempt count that separates the two
- Ordering of the records that follow a set-aside record
- Two partitions healthy while one is stuck
- A record set aside and later replayed

## What interviewers look for

Whether a retry budget exists and dead-lettering is a real outcome rather than a code path nobody reaches. A full-marks answer caps attempts per record, preserves the payload for replay, keeps in-order progress for the rest of the partition, and treats "busy forever with no alarm" as its own observability defect.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/poison-message-blocks-partition
