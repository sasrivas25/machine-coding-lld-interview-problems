# One Tenant's Volume Slows Every Tenant

Difficulty: Hard. Core topic: partition pruning, tenant isolation.

The noisy-neighbor database round. PostgreSQL has one event partition per large tenant plus an overflow partition, yet a small tenant's read cost grows with the largest tenant's data.

## Scenario

A multi-tenant event store is declaratively partitioned by `tenant_id`. The physical design is intended to isolate reads, but the repository's query shape prevents PostgreSQL from proving which partition contains the requested tenant. After one tenant grows to tens of thousands of events, queries for tenants with only hundreds of rows scan unrelated partitions and lose their latency isolation.

Tests inspect table statistics directly, so a superficially fast query over seed data does not pass.

## Requirements

- Make a tenant-scoped read touch exactly one partition.
- Read only rows belonging to the requested tenant.
- Preserve ordering, limits, and existing repository behavior.
- Leave schema, partitions, indexes, and seed data unchanged.
- Express the tenant restriction in a form PostgreSQL can prune.

## Edge cases to handle

- A tenant with its own dedicated partition
- A small tenant routed to the overflow/default partition
- Tenant identifiers passed through joins, casts, or expressions
- Empty tenants and bounded result limits
- Plans prepared with parameters rather than literals

## What interviewers look for

Whether you inspect the execution path and understand partition pruning rather than proposing more hardware or indexes. The strongest repair restores an explicit predicate on the partition key and proves isolation through touched-partition and scanned-row statistics.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/hot-partition-multi-tenant-database
