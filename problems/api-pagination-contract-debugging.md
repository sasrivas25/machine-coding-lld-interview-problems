# API Contract Debugging: Cursor Pagination

Difficulty: Medium. Core topic: cursor pagination, contract testing.

The pagination-correctness round. A client walks pages by echoing back the cursor it was handed, and the contract is unforgiving: every item exactly once, stable order, no duplicates, no gaps, and a clean terminating signal. Off-by-one at the page boundary and iteration silently corrupts.

## Scenario

You inherit a small in-process API modelled as a pure `handle(request) -> response` function — no HTTP server, no network, no framework. It exposes one read endpoint, `GET /items?limit=&cursor=`, paginating over a fixed in-memory dataset sorted by a stable integer id. A client starts with no cursor and repeatedly calls back with the `next_cursor` it received; done right, this reconstructs the whole dataset in id order.

The shipped handler almost works, which is the trap. Depending on the language it double-counts the boundary item across two pages, drops an item at each boundary, or emits a `next_cursor` that never resolves to a terminal page so iteration loops forever. The bundled contract tests encode the invariants and currently FAIL on `main`.

## Requirements

- Iterating from no cursor through each returned `next_cursor` yields every item exactly once, in stable id order.
- No duplicates across page boundaries and no gaps between pages.
- Each page honors the requested `limit`.
- `has_more` is accurate on every page; the final page sets `has_more=false` and `next_cursor=null`.
- Cursors are opaque to the client but decoded deterministically by the handler.

## Edge cases to handle

- Empty dataset — one clean page, no cursor.
- A dataset smaller than `limit` — single-page result terminates immediately.
- An out-of-range or stale cursor pointing past the end.
- Invalid or non-positive `limit` values.
- The exact page boundary where the last item lands.

## What interviewers look for

Whether you reason about the cursor as a precise position claim rather than a fuzzy offset: does it mean "resume at this id" or "resume after this id," and does the slice boundary and the terminal condition agree with that meaning? A full-marks answer nails the half-open interval so the boundary item is neither dropped nor repeated, and proves termination — `has_more` and `next_cursor` derived from the same fact, never contradicting each other.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/api-pagination-contract-debugging