# Connector Record Reconciliation Sync

Difficulty: Hard. Core topic: reconciliation, sync idempotency.

The forward-deployed sync round. A connector pulls customer records from several upstream systems, reconciles them against local rows, and must never let one source clobber another — while a cursor that under-advances quietly replays work forever.

## Scenario

You maintain a connector that syncs customer records from multiple external systems into a local account table. Each run receives paged connector snapshots, reconciles them against existing local rows, emits upserts and deletes, and returns a cursor to persist for next time. Simple create and update cases pass, so it looks healthy.

Production tells a different story. Records from different connector identities can overwrite each other, because identity is being collapsed to a shared key. And some runs replay tombstone-only or unchanged pages because the cursor does not move far enough after a fully scanned page — the next run re-reads ground it already covered, re-emitting changes that were already applied.

## Requirements

- Reconciliation preserves connector identity: a record from one source never overwrites a distinct record from another.
- For each remote record, the newest event wins; older events for the same record are ignored.
- Emit exactly the upserts and deletes implied by the reconciled snapshot against local state.
- Tombstones (deletes) reconcile correctly and do not resurrect or drop the wrong rows.
- The cursor advances deterministically past every fully scanned page so no page is replayed.

## Edge cases to handle

- Two connectors carrying the same remote id under different identities.
- A page containing only tombstones or only unchanged records.
- Out-of-order events for one record within a snapshot.
- An empty page and an already-fully-synced run.
- A record deleted upstream then re-created.

## What interviewers look for

Whether identity and recency are modelled explicitly rather than assumed. A full-marks answer keys records by (connector identity, remote id), resolves conflicts by event recency deterministically, and treats the cursor as a commitment: once a page is fully processed the cursor must move past it so the run is idempotent under replay. Sync jobs live or die on whether re-running them is a no-op.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/connector-record-reconciliation-sync-coding-problem