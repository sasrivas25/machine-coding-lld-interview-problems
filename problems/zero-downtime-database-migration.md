# Zero-Downtime Column Split Loses Rows

Difficulty: Hard. Core topic: expand-contract migration, resumable backfill.

The database-engineering round. An expand-contract migration backfills a live table in batches, misses the rows written while it runs, and then reports success — handing the contract phase permission to drop a column that still has unmigrated data behind it.

## Scenario

Gronex is splitting `users.full_name` into `first_name` and `last_name`. The expand phase has already added the two nullable columns. A backfill job walks the table in batches while the service keeps serving traffic, and the contract phase — dropping `full_name` — runs once the backfill reports completion.

Staging runs leave rows behind. After the backfill reports that it is finished, some users still have `NULL` in both new columns, in two patterns: users created while the backfill was running are never populated, and users that existed before the run are occasionally missed when the table changes between batches. The completion check reports success regardless, so contract would drop `full_name` while data is still unmigrated.

## Requirements

- Every row present when the backfill finishes must have both new columns populated, including rows written during the run.
- Do not skip rows when the table is modified between batches.
- Report completion only when no rows remain to migrate.
- Keep each batch bounded, and keep progress durable across process restarts.
- Do not modify the tests or `verify.sh`; a single unbounded `UPDATE` is not acceptable.

## Edge cases to handle

- Inserts and updates landing behind the batch cursor mid-run
- Offset-based batching shifting when rows are deleted
- A restart resuming from stale progress
- The final batch and the emptiness check that gates contract
- Names that do not split cleanly into exactly two parts

## What interviewers look for

Whether the write path is fixed alongside the backfill — a dual-write or a default that keeps new rows migrated — instead of only chasing the existing rows. A full-marks answer batches on a stable key rather than an offset, makes completion a real emptiness check, and treats resumability as a requirement, not a nicety.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/zero-downtime-database-migration
