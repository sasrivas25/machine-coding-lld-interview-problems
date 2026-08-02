# Catalog Pagination Skips and Repeats Products

Difficulty: Hard. Core topic: keyset pagination, composite indexes.

The large-table pagination round. Offset pages become slower with depth and mutate underneath the reader, causing duplicates and skipped products while a catalog is being published and removed.

## Scenario

A catalog contains 50,000 published products ordered newest first. The repository returns an opaque cursor, but internally relies on offset-style traversal. Inserts before the current offset shift previously seen rows onto later pages; deletes shift unseen rows backward. Deep pages must scan and discard everything before the requested offset.

The API must remain cursor based, but the cursor needs to identify a stable position in the ordering rather than a row count.

## Requirements

- Use deterministic keyset/seek pagination.
- Avoid duplicates when newer products are inserted mid-session.
- Avoid skipping unseen products when earlier rows are deleted.
- Keep work per page independent of pagination depth.
- Add a supporting index through a forward migration when needed.

## Edge cases to handle

- Multiple products sharing the same publication timestamp
- Empty, final, and partially filled pages
- A cursor whose referenced row has been deleted
- Inserts between consecutive page requests
- Invalid or malformed opaque cursors

## What interviewers look for

Whether you define a total ordering such as `(published_at, id)`, encode both values in the cursor, apply the corresponding strict seek predicate, and back it with a matching composite index. Rebranding an offset as a cursor does not solve correctness or complexity.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/large-table-pagination-failure
