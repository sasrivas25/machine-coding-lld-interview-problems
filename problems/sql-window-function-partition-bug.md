# SQL Window Isolation in Fulfillment Event History

Difficulty: Hard. Core topic: window functions, partitioning.

The data-engineer round. A multi-tenant, versioned event table meets window functions, and a mis-scoped `PARTITION BY` quietly bleeds one tenant's history into another and lets one line's transitions corrupt the next. The report reads plausible right up until identifiers collide.

## Scenario

A fulfillment warehouse reconstructs meaningful state transitions from a versioned, multi-tenant event table. The report must keep the latest event revision visible at an ingestion cutoff, collapse consecutive duplicate scanner observations, and emit deterministic transition metadata for every order line. Production reconciliation now shows missing tenant histories and impossible line transitions — exactly the symptoms of window frames and revision logic that don't respect identity when identifiers collide or several lines advance together.

Repair the SQL-backed Python report without changing the fixture or tests. The query must preserve tenant isolation, respect event revision identity even when corrected payload fields shift, and keep each physical line's history independent.

## Requirements

- Keep the latest event revision visible at the ingestion cutoff.
- Collapse consecutive duplicate scanner observations into one.
- Emit deterministic transition metadata for every order line.
- Preserve tenant isolation across all windowed computations.
- Respect event revision identity even when corrected payload fields move.
- Keep each physical line's history independent of every other line's.

## Edge cases to handle

- Identifiers that collide across tenants and must not merge
- Multiple lines advancing together in the same batch
- Consecutive duplicate observations that must dedupe without dropping real transitions
- A corrected revision whose payload fields changed but whose identity did not
- The cutoff boundary where the latest revision must remain visible

## What interviewers look for

Whether the candidate partitions windows by the full identity — tenant and physical line — rather than a convenient subset, and separates revision identity from payload content so corrections don't masquerade as new events. Full marks produce a deterministic, tenant-isolated result where no line's transitions leak across a partition boundary, proving the fix restores correct window scoping rather than patching a single symptom.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/sql-window-function-partition-bug-coding-problem
