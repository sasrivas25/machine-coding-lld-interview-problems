# Database Overload Under a Traffic Spike

Difficulty: Hard. Core topic: indexing, query planning, N+1 queries.

The query-efficiency round. An order-history endpoint looks fine in development but turns one request into thousands of statements and scans an entire table before returning a ten-row page.

## Scenario

A storefront retrieves a customer's recent orders and computes item counts and totals. The starter path fetches the customer's full history without a SQL limit, issues another aggregate query for every order, and truncates the result in application code. The `orders` table also lacks an index matching the customer filter and recency order.

Under promotional traffic, database CPU is dominated by sequential scans, statement count scales with total history rather than page size, and heavy customers materialize thousands of unused rows.

## Requirements

- Return exactly the requested page size in recency order.
- Execute a bounded number of statements independent of history length.
- Compute item counts and totals with set-based SQL.
- Add a forward index supporting the primary filter and order.
- Eliminate the sequential scan from the primary query plan.

## Edge cases to handle

- Customers with thousands of orders
- Orders with zero or many line items
- Stable ordering when timestamps collide
- Very small and maximum allowed page sizes
- Avoiding a query rewrite that still performs per-row subwork

## What interviewers look for

Whether you move limiting and aggregation into the database, remove the N+1 loop, and verify the planner can satisfy `WHERE customer_id = ? ORDER BY placed_at DESC LIMIT ?` from a suitable composite index. Full marks reason from statement and row counts, not local latency.

---

Practice this in a real repo with a failing test suite → https://gronex.org/database-overload-traffic-spike-coding-problem
