# Zero-Downtime Column Split Loses Rows

Difficulty: Hard. Core topic: expand-contract migration, online backfill.

The online-schema-migration round. A background job splits `full_name` into new columns while production writes continue, then announces completion even though old and newly inserted rows still contain nulls.

## Scenario

The expand phase has added nullable `first_name` and `last_name`. A bounded backfill migrates existing users and stores progress in a single-row cursor, while `UserWriter` keeps accepting traffic. The starter job uses unstable progress logic and the live write path still populates only the legacy column.

The contract phase cannot safely remove `full_name` until every row is migrated and all new writes maintain the expanded representation.

## Requirements

- Dual-write new and legacy columns during the migration window.
- Visit every pre-existing row exactly as needed in bounded batches.
- Avoid skipping rows when concurrent inserts occur.
- Report completion only when no row still needs migration.
- Make rerunning a completed backfill a safe no-op.

## Edge cases to handle

- Rows inserted before, during, and after a batch
- Gaps in monotonically increasing IDs
- A crash after updating rows but before recording progress
- Names with one or several components
- A stale progress cursor that claims the scan is finished

## What interviewers look for

Whether you apply expand–migrate–contract discipline: make the application forward-compatible first, scan with a stable key, keep batches transactional and retryable, and verify the actual null predicate before declaring completion. Timing the deployment is not a consistency strategy.

---

Practice this in a real repo with a failing test suite → https://gronex.org/zero-downtime-database-migration-coding-problem
