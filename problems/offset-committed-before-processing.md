# Settlements Vanish After a Consumer Restart

Difficulty: Hard. Core topic: commit ordering, exactly-once effects.

The distributed-systems round. The consumer commits its position before the work is applied, so a restart skips records the broker will never send again — and the obvious repair trades the loss for duplicates.

## Scenario

A settlement consumer reads records from a partitioned stream in batches of five and applies each to an account total. As it works it commits its position so a restarted process knows where to resume. The broker's contract is the usual one: after a restart it resumes from the committed position, and anything at or below that position is treated as handled and will never be sent again.

The settler was restarted during a deployment. Reconciliation afterwards came up short: several accounts are missing money the upstream system insists it sent. Nothing in the consumer's own account of itself looks wrong — the committed position advanced smoothly from start to end, with no gaps and no rewinds, and the log shows it settling records both before and after the restart.

The captured evidence holds the delivery trace, the ledger of settlements applied, the record of committed positions, and the settler's log. Reconciling the ledger against the position file is the shortest route to the defect.

## Requirements

- Apply every offered record exactly once across a restart: nothing lost, nothing applied twice.
- End every account total equal to the sum of its distinct records.
- Keep the already-passing tests passing.
- Note that moving the commit after processing alone introduces duplicates — the suite checks for both faults.
- Do not modify the tests.

## Edge cases to handle

- A crash between applying a record and committing its position
- A crash between committing and applying
- A redelivered record after the restart
- A partial batch interrupted mid-way
- Multiple partitions committing independently

## What interviewers look for

Whether you see that commit ordering alone cannot give exactly-once, and pair the later commit with durable per-record deduplication. A full-marks answer makes apply-and-record atomic, reconciles the ledger exactly, and can state which direction each ordering fails in.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/offset-committed-before-processing
