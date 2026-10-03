# CDC Search Index Synchronization

Difficulty: Hard. Core topic: change data capture, transaction visibility, idempotent projections.

The database-engineering round. A projector consumes an append-only change log into a search index, but it ignores transaction visibility, drops deletes, and duplicates rows on replay — so the projection drifts from the source and never repairs itself.

## Scenario

You inherit a document management service that keeps a searchable projection in the same database. Writes go to `documents`, and every write appends a row to the append-only `document_changes` log. A background projector consumes that log in order, maintains `search_index`, and records progress in `projection_cursor`.

Support has three reports. Some documents never appear in search even though they exist, and re-running the projector never repairs them — the cursor has advanced past change rows that were not yet visible when it read. Deleted documents keep appearing in search results. And after an operator replayed the log from a reset cursor to rebuild the projection, several documents appeared multiple times.

The projector runs on demand: tests call `poll(batch_size)` explicitly, so the fix must be deterministic — no sleeps, no wall-clock timeouts.

## Requirements

- Consume and apply every committed change eventually, with no permanently skipped rows.
- Reflect deletions in the projection.
- Make the projection idempotent: replay from a reset cursor reproduces the same final state with no duplicates.
- Keep `poll(batch_size)` bounded and its public shape unchanged.
- Change only code under `src/` — the base migration, seed data, tests, and `verify.sh` are fixed.

## Edge cases to handle

- A change row committed by a transaction that started before the one already consumed
- Gaps in the change sequence that later fill in
- Delete followed by re-create of the same document id
- Replay from cursor zero over a projection that already holds rows
- A batch that ends mid-transaction

## What interviewers look for

Whether you understand that a monotonic id is not a visibility guarantee — rows can appear in the log behind a cursor that has already moved past them. A full-marks answer advances the cursor only past changes that are definitely visible, makes each apply an upsert or delete keyed by document id, and proves replay-safety instead of asserting it.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cdc-search-index-synchronization
