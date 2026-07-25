# Incident Debugging: Connection Pool Leak (Service Outage Under Load)

Difficulty: Medium. Core topic: resource leak, error handling.

The production-incident round. A bounded connection pool, a request path that borrows and returns connections, and a spike of failing queries that quietly bleeds the pool dry until every request — valid or not — dies waiting for a connection.

## Scenario

At 09:14 UTC the `orders-api` service began timing out in production. Within two minutes every request was failing, the load balancer pulled the instance, and the process fell into a crash loop that bought only minutes at a time. You are handed the service code and `logs/incident.log` from the outage. The log tells the story: at 09:14:02 an upstream started sending malformed order ids, failing queries pile up, the pool's in-use gauge climbs, `acquired_total` steadily outruns `released_total`, and the pool finally exhausts — after which even valid requests can no longer get a connection. The service is reproduced in-process with deterministic fakes, so the incident replays exactly on your machine. Something in the request path loses a connection whenever a query fails.

## Requirements

- Read the log first and confirm where acquired outpaces released.
- Fix the request path so it can never lose a connection on any outcome.
- Ensure every valid request still succeeds across the full workload.
- Ensure a failing query surfaces the query error, never pool exhaustion.
- Leave the pool's in-use count at zero once the workload finishes.
- Do not edit the tests; make the bundled workload pass and run `./verify.sh`.

## Edge cases to handle

- A query that raises for an unknown order id mid-request
- A pool at its limit where `acquire()` raises immediately rather than blocking
- Interleaved valid and failing requests across a long run
- Acquired-versus-released totals that must reconcile exactly
- Distinguishing the real error from the downstream pool-exhaustion symptom

## What interviewers look for

Whether the connection is released on every path out of the request, success or failure, rather than only on the happy path. A strong answer reads the log to locate the leak before touching code, recognizes that the failing query short-circuits the release, and guarantees the borrow is always returned regardless of outcome. The invariant is the whole point: a connection lent out is always given back, so a spike of failures degrades gracefully instead of exhausting the pool.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-connection-pool-leak-debugging
