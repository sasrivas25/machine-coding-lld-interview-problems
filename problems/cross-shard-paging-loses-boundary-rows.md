# A Paged Export Is Silently Returning a Third of the Orders

Difficulty: Hard. Core topic: cross-shard pagination, merge boundaries.

The distributed-systems round. A paged listing merges rows from four shards and keeps only the first ten, then advances every shard's cursor past rows it never returned — so a 120-order walk returns 30 and ends cleanly.

## Scenario

An orders API serves a paged listing over four shards. Each page asks every shard for its next rows, merges them in sort order, returns the first ten, and hands back a cursor the caller passes to the next request.

A finance team reconciling a month of orders found their export short — not by a few rows: a walk that should return a hundred and twenty orders returns thirty, and the walk ends cleanly with no error. Exports from a single shard reconcile perfectly, which is why this went unnoticed for two release cycles.

The capture in `artifacts/` walks the whole result set one page at a time and records, for each page, the first and last sort key it returned and how many rows it read from each shard.

## Requirements

- Make a full walk return every row exactly once.
- Keep each page reading at most one page worth of rows from any shard — reading whole shards per page is not the fix.
- Keep the merged ordering correct across shards.
- Keep the cursor opaque and the page size honoured.
- Do not modify the tests.

## Edge cases to handle

- Unreturned rows that must be reconsidered on the next page
- Rows with equal sort keys spanning a shard boundary
- A shard that runs out of rows before the others
- The last page and the end-of-walk signal
- A shard returning fewer rows than requested

## What interviewers look for

Whether you realise the cursor must remember the merge boundary, not each shard's read position. A full-marks answer advances every shard only to the last key actually returned, keeps per-page reads bounded, and can prove completeness by reconciling the walk against the full set.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/cross-shard-paging-loses-boundary-rows
