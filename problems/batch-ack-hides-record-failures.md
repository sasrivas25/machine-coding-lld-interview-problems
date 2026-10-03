# Some Ledger Records Never Reach the Ledger

Difficulty: Hard. Core topic: per-record acknowledgement, failure accounting.

The distributed-systems round. A settlement service acknowledges a whole batch after submitting it, so records the downstream refused are marked consumed and can never be replayed — and nothing alerts, because the service logged each one as a warning.

## Scenario

A settlement service reads accounting records from an ordered stream in batches of five, submits each record to a downstream ledger service, then tells the stream how far it has consumed so the stream can move on. It has run for months without complaint.

Finance has opened a discrepancy. Over the last reconciliation period, ninety-six records reached the ledger but one hundred and twenty were published. The missing twenty-four are spread across the whole period rather than clustered, and every one belongs to an account the ledger service was temporarily refusing while a review flag was set. The flags have since been cleared, but the records have not appeared, and replaying the stream does not bring them back: the stream considers them delivered.

The log is not silent. It contains a warning for every one of the twenty-four, naming the record and the reason the ledger gave. Nobody was paged, no error rate moved, and the consumer never fell behind.

A capture of one reconciliation period is in `artifacts/`.

## Requirements

- Stop discarding work the service did not complete.
- Keep settling records the ledger accepts.
- Keep making progress through the stream — stalling forever on one bad record is not a fix.
- Make refused records recoverable once the downstream accepts them again.
- Do not modify the tests.

## Edge cases to handle

- A batch where some records succeed and some are refused
- A refusal that is permanent versus one that clears later
- The consumed position advancing past an incomplete record
- Ordering requirements within an account after a retry
- Repeated refusals of the same record

## What interviewers look for

Whether you tie the acknowledgement to actual completion rather than to the attempt, and whether progress and durability survive together. A full-marks answer acknowledges per record or holds the position at the first incomplete one, keeps refused work somewhere replayable, and treats a warning-level log on lost money as its own defect.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/batch-ack-hides-record-failures
