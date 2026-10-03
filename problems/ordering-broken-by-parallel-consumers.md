# Account Updates Land Out of Order Under Load

Difficulty: Hard. Core topic: per-key ordering, parallel dispatch.

The distributed-systems round. Three workers apply updates in parallel and the dispatcher picks a worker without regard to the account, so a degraded host leaves accounts resting on an older version than one already applied.

## Scenario

A pool of three workers applies account updates in parallel. Each update names the account it affects and carries a version that increases by one per account. Updates to different accounts are independent and may run simultaneously. Updates to the same account are not: each overwrites that account's stored state, so they must take effect in version order.

The dispatcher answers exactly one question per update — which worker should run it — and the pool does the rest.

One worker then began running several times slower than the others after its host degraded. Reduced throughput was expected. What followed was not: several accounts settled on an older version than the newest update already applied to them, and stayed there.

Nothing failed. Every update was applied, none rejected, none applied twice. The evidence holds the dispatch order with the chosen worker, the completion order, the pool's log, and a timeline of which worker was occupied by which update. Comparing dispatch order against completion order for a single account is the shortest route to the defect.

## Requirements

- Restore per-account ordering under a degraded pool.
- Never have two updates for one account in flight at once.
- End every account at its highest version.
- Keep throughput: work must keep reaching every worker and a clean run must stay inside its makespan budget — routing everything through one worker will not do.
- Do not modify the tests.

## Edge cases to handle

- Two updates for one account dispatched back to back
- Unrelated accounts that must still run concurrently
- A slow worker holding the account whose next update has arrived
- An account with a single update
- Makespan budget on a healthy pool after the change

## What interviewers look for

Whether ordering is enforced by routing on the key, not by serialising the pool or sorting after the fact. A full-marks answer maps an account deterministically to one worker (or gates per-account concurrency), keeps all workers fed, and can explain why parallelism is safe exactly where keys differ.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/ordering-broken-by-parallel-consumers
