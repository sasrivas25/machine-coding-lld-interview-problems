# Catalog Pagination Skips and Repeats Products

Difficulty: Hard. Core topic: keyset pagination, stable cursors.

The database-engineering round. A fifty-thousand-row catalog is paged with `LIMIT`/`OFFSET` while the catalog keeps changing, so clients get the same product twice, never see others at all, and crawl slower the deeper they page.

## Scenario

`catalog_items` holds fifty thousand published products. A repository returns one page of items, newest first, plus an opaque cursor the caller passes back for the next page. Clients walk the whole catalog by following cursors while publishers continue inserting and removing rows.

Three complaints arrive from feed consumers. The same product appears on two consecutive pages. A product that was never returned is missing from the feed entirely. And the feed gets slower the further a client pages, with deep pages reading the entire table.

The cursor is opaque to callers, so its encoding is yours to choose — but invalid cursors must still be rejected.

## Requirements

- A product published mid-walk must never be returned twice.
- A product removed mid-walk must not cause an unseen product to be skipped.
- Work to serve one page must not grow with how deep the client has paged.
- Keep ordering newest-first and the page size honoured exactly.
- Reject malformed or tampered cursors cleanly.
- Do not modify the tests or `verify.sh`.

## Edge cases to handle

- Rows sharing the same timestamp, making the sort key non-unique
- The first page, where no cursor exists yet
- The final page and the end-of-feed signal
- A cursor pointing at a row that has since been deleted
- Inserts landing before the client's current position

## What interviewers look for

Whether you recognise that offset pagination is unstable by construction, not merely slow. A full-marks answer moves to a keyset cursor on a unique, indexed sort key, ties the tiebreaker into the cursor so pages cannot overlap, and demonstrates that page cost is constant with depth.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/large-table-pagination-failure
