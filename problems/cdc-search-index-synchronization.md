# CDC Search Index Synchronization

Difficulty: Hard. Core topic: CDC ordering, idempotent projections.

The database-engineering round where a green projector silently loses committed changes. A search index is rebuilt from an append-only PostgreSQL change log, but transaction visibility, serial IDs, deletes, and replay semantics make a cursor-only consumer unsafe.

## Scenario

A document service stores authoritative rows in `documents` and appends every mutation to `document_changes`. A projector polls changes in `change_id` order, updates `search_index`, and advances a durable cursor.

In production, a slow transaction allocates a lower serial ID while a faster transaction allocates a higher ID and commits first. The projector sees the higher committed row, advances past it, and never returns for the lower row when it eventually commits. Deletes are ignored, and resetting the cursor exposes non-idempotent insert/update handling.

## Requirements

- Consume every committed change without advancing across an invisible transaction gap.
- Apply inserts, updates, and deletes to the search projection.
- Advance projection state atomically with the applied batch.
- Make replay safe and converge on the newest document state.
- Keep polling bounded by the requested batch size.

## Edge cases to handle

- Transaction commit order differing from sequence-allocation order
- A delete followed by replay from an older cursor
- Multiple changes for the same document in one batch
- A crash between applying rows and advancing the cursor
- Concurrent projectors observing the same eligible changes

## What interviewers look for

Whether you understand that PostgreSQL sequences are not commit-order clocks. Strong answers introduce a safe transaction visibility watermark, make each projection write idempotent, handle tombstones explicitly, and keep the cursor and projection in one transaction.

---

Practice this in a real repo with a failing test suite → https://gronex.org/cdc-search-index-synchronization-coding-problem
