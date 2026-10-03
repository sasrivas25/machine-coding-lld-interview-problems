# Database Overload Under a Traffic Spike

Difficulty: Hard. Core topic: indexing, bounded query work, N+1.

The database-engineering round. An order-history endpoint is fast for one request and fatal for a thousand: sequential scans on `orders`, a statement count that grows with the customer's history, and a result set with no ceiling.

## Scenario

You inherit the endpoint behind a storefront's "my orders" screen. It returns a page of a customer's most recent orders, each with its line-item count and order total, from PostgreSQL. `customers` holds accounts, `orders` holds one row per order with `customer_id`, `status`, and `placed_at`, and `order_items` holds one row per item with `order_id`, `sku`, `quantity`, and `unit_price_cents`.

During a promotion the database became the bottleneck and the endpoint became unusable. Database CPU was dominated by sequential scans over `orders`. The number of statements per request grew with the size of the customer's order history instead of the size of the requested page. A customer with a long history could return an unbounded result set on its own.

Single-threaded requests are fast. The defect only shows up under load.

## Requirements

- Return byte-identical data to the buggy version — the response shape must not change.
- Execute a bounded number of statements per request, independent of history size.
- Keep row scans within a small multiple of the page size N.
- Return exactly N results when N are requested, never more.
- Eliminate sequential scans on `orders` from the plan for the primary operation.
- You may add a new forward migration; do not rewrite the existing one, the seed data, or the tests.

## Edge cases to handle

- The per-order aggregate that currently issues one query per row
- A customer whose history is far longer than one page
- Ties in `placed_at` making the page boundary ambiguous
- An index that covers the filter but not the sort, leaving a sort step behind
- Customers with no orders

## What interviewers look for

Whether you separate the two defects — the missing index and the N+1 — instead of fixing one and declaring victory. A full-marks answer bounds both statement count and rows scanned, backs the claim with a plan rather than a stopwatch, and keeps the output identical so the fix is provably behaviour-preserving.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/database-overload-traffic-spike
