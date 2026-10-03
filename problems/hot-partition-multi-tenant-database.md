# One Tenant's Volume Slows Every Tenant

Difficulty: Hard. Core topic: partition pruning, tenant isolation.

The database-engineering round. An `events` table is partitioned by tenant, but every query touches every partition, so the one tenant holding a hundred times more data than the rest sets the latency for all of them.

## Scenario

`events` is partitioned by tenant, and one tenant holds roughly a hundred times the volume of the others. A repository serves three per-tenant reads: a recent-events feed, an event count, and a usage report.

Small tenants report that their feeds are slow, and the slowness tracks the largest tenant's growth rather than their own. Reading twenty events for a tenant with two hundred events performs work proportional to the entire table. Partition pruning is simply not happening — the planner cannot narrow the query to one partition, so each read fans out across all of them.

The partitioning scheme itself is fine. The defect is in how the queries address it.

## Requirements

- A read for one tenant must touch only that tenant's partition.
- Work done for a small tenant must not scale with the largest tenant's row count.
- Keep results correct and strictly scoped to the requested tenant.
- Keep the caller-facing API unchanged: tenants are identified by their external key.
- Do not modify the tests or `verify.sh`, and treat the partitioning scheme as fixed.

## Edge cases to handle

- Translating an external tenant key into the value the partition key is defined on
- Aggregates (count, usage report) pruning as well as the row feed does
- A tenant with no events at all
- Queries whose predicates defeat pruning because the key is wrapped in an expression
- Correct ordering and limits once the query is pruned to one partition

## What interviewers look for

Whether you can read a plan and see that pruning failed, then trace it back to the predicate shape rather than blaming data volume. A full-marks answer makes the partition key visible to the planner on every read path, proves isolation by showing small-tenant work is independent of the hot tenant, and never gets there by denormalising or caching around the problem.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/hot-partition-multi-tenant-database
