# Incident Debugging: Black Friday 503s

Difficulty: Hard. Core topic: Postgres MVCC, migrations, timeouts, cache expiry.

The production-incident round at full difficulty. Twenty seconds into the Black Friday sale the orders API starts shedding a third of its traffic with 503s, and app, database and cache metrics all look wrong at once. Originally set as a Gronex Engineering Cup #1 problem.

## Scenario

It is 18:00 IST on Black Friday. `orders-api` rolled out at the start of the sale, its deploy hook ran a database migration, and within half a minute the load shedder was rejecting most requests. When the pool finally drained, the order cache went cold and the API fell over a second time. On-call captured three logs from a replay with aligned timestamps: `logs/app.log` (worker pool, load shedder, latency), `logs/db.log` (statements, lock waits, autovacuum) and `logs/cache.log`. The limits and policies are in `docs/runbook.md`. The system is reproduced in-process and deterministically, including MVCC dead tuples and an xmin horizon autovacuum cannot cross.

## Requirements

- Correlate the three logs and find every root cause; there is more than one and they interact.
- Fix them in the code your team owns: the database client, the order cache, the settings and the migrations.
- Make the incident replay meet the runbook's SLOs.
- Do not edit the tests, the harness, the logs or the runbook, and run `./verify.sh`.

## Edge cases to handle

- A migration that takes an exclusive lock on a hot table during peak traffic
- Statements with no timeout queueing behind DDL until the worker pool saturates
- Dead tuples piling up while something holds the xmin horizon autovacuum cannot cross
- Cache entries expiring together, sending a read stampede to the primary
- Meeting every SLO at once: error rate, pool saturation, p99 latency, primary reads, dead-tuple ratio and lock time

## What interviewers look for

Whether you work from evidence across systems instead of guessing at the loudest metric. A strong answer builds a timeline across the three logs, names each root cause with the line that proves it, and fixes causes rather than raising limits until the symptoms go away.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/black-friday-503s-incident
