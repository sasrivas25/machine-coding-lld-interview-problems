# Kanban Board with WIP Limits and Card Flow

Difficulty: Medium. Core topic: ordering, WIP limits.

The board-mechanics round. Columns cap how many cards they hold, cards keep a deterministic order as they flow, transitions respect adjacency, and one board's state must never leak into another. Each rule is simple; getting all four right at once is the test.

## Scenario

You are working on a partially implemented Kanban board backend. Boards own an ordered set of columns, each with a work-in-progress limit; cards flow between columns while keeping a stable order. The models and repositories are in place, but several visible tests fail because some service and repository logic is incomplete or wrong.

The symptoms cluster: a column accepts more cards than its limit allows, card order drifts or leaves gaps after a move, transitions jump non-adjacent columns, and operations on one board occasionally see or touch cards belonging to another. You must fix the implementation without rewriting it or modifying the tests.

## Requirements

- Each column enforces its WIP limit — a move into a full column is rejected.
- Cards within a column keep a contiguous, deterministic order.
- Cards transition only between adjacent columns in the board's order.
- Cards and columns from one board never interfere with another (isolation).
- Existing public method contracts are preserved.

## Edge cases to handle

- Moving a card into a column already at its limit.
- Reordering after a card leaves the middle of a column (no gaps).
- A transition that skips a column or moves backward.
- Two boards with columns of the same name.
- Moving a card to the column it already occupies.

## What interviewers look for

Whether invariants are enforced at the service boundary rather than assumed by callers. A full-marks answer checks the WIP limit before committing a move, maintains contiguous ordering as a maintained property rather than an accident of insertion, validates adjacency against the board's own column order, and scopes every lookup to its board so isolation is structural, not coincidental.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/kanban-board-with-wip-limits-and-card-flow