# Pandas Order Reconciliation: Join Fan-out and Deduplication

Difficulty: Hard. Core topic: joins, change-data-capture.

The silent undercount round. A reconciliation job joins orders against append-only child streams, the output looks plausible, and then orders with several line items come out understated while a late correction gets ignored entirely. Every bug hides inside a join that fans out or a duplicate that never got collapsed.

## Scenario

A warehouse job combines canonical orders with append-only line-item and payment-event streams to produce one reconciliation row per merchant and order. Each child identity may arrive more than once — with revisions, ingestion-sequence tie breakers, and line tombstones — so the current record for each identity has to be resolved before anything is aggregated. The report must preserve all legitimate active lines and captured payment events, retain orders that have no child activity at all, and stay stable when delivery order changes.

The starter produces output that looks right at a glance, but orders carrying multiple child records are understated and late corrections can be dropped. The symptoms point at a join that fans out one order across its children before aggregation and a deduplication step that does not consistently keep the current record per identity. Repair the pandas transformation without changing its public function, output schema, or bundled tests.

## Requirements

- Emit exactly one reconciliation row per merchant and order.
- Resolve each child identity to its current record using revisions and the ingestion-sequence tie breaker.
- Honor line tombstones so retired lines drop out of the totals.
- Preserve all legitimate active lines and captured payment events in the aggregates.
- Retain orders that have no child activity.
- Produce identical output regardless of input delivery order.

## Edge cases to handle

- An order with several active lines that must not fan out its own fields
- Two records for one identity separated only by ingestion sequence
- A tombstone that arrives after the line it retires
- A late revision that should override an earlier version
- An order with no children that must still appear

## What interviewers look for

Whether you collapse each child stream to its current-per-identity record before joining, so aggregation counts each line and payment exactly once, and whether the join direction avoids fanning an order's own fields across its children. The strongest fixes are order-independent by construction and keep childless orders in the output rather than dropping them through an inner join.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/pandas-join-fanout-deduplication-coding-problem