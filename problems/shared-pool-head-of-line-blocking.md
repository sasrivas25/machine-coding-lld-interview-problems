# One Slow Dependency Made Every Dependency Slow

Difficulty: Hard. Core topic: bulkheads, resource isolation.

The distributed-systems round. Three dependencies share six connections, so a 300 ms stall in one of them takes the whole pool hostage and pushes 20 ms catalog lookups past a second.

## Scenario

A checkout API calls three dependencies through a shared pool of six connections. A call takes a connection before it starts and returns it when it finishes, waiting if none is free. Six is the provisioned capacity of the service, not a tuning parameter.

A recommendation service had a bad deploy and its calls went from 20 ms to 300 ms for about six hundred milliseconds. That was expected to make recommendations slow. Instead checkout latency collapsed across the board: catalog lookups, answering in 20 ms throughout, took up to 1110 ms, and pricing reached 1120 ms. Neither service was touched and both were healthy for the entire incident.

The evidence in `artifacts/` samples pool occupancy at every acquire, broken down by dependency, so the three columns can be followed from before the stall into it.

Four tests pass and four fail. The passing ones describe behaviour that must stay correct: every call completes when all dependencies are healthy, a brief stall is absorbed without affecting anyone, and the pool never exceeds its provisioned capacity.

## Requirements

- Keep one slow dependency from degrading calls to the others.
- Keep total pool capacity at six — raising it is not a fix.
- Avoid restricting each dependency to a single connection, which makes the healthy case worse.
- Keep every call completing when all dependencies are healthy.
- Keep a brief stall absorbed without affecting anyone.

## Edge cases to handle

- All six connections demanded by the slow dependency
- A healthy dependency arriving while the slow one holds most of the pool
- A dependency idle while another could use its share
- The brief stall that should be absorbed, not isolated away
- Fairness when two dependencies are busy and one is not

## What interviewers look for

Whether you partition the resource per dependency with limited borrowing, rather than enlarging it or splitting it evenly and rigidly. A full-marks answer caps any one dependency's share while letting idle capacity be used, keeps the healthy case as fast as before, and names the pattern as a bulkhead.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/shared-pool-head-of-line-blocking
