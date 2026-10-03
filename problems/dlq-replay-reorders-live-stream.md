# Replayed Updates Roll Accounts Back to Old Values

Difficulty: Hard. Core topic: replay ordering, state convergence.

The distributed-systems round. Parking failed updates and replaying them later loses nothing and corrupts everything: the replayed version is older than what the account has already moved on to, and the projector applies it anyway.

## Scenario

A profile projector consumes a stream of account updates and keeps one current profile per account. When an enrichment dependency refuses an update, the projector parks it rather than dropping it, and replays the parked updates once the dependency recovers. Parking and replaying was added deliberately, and it works: nothing is lost.

Support has escalated three accounts whose tier and credit limit are wrong. All three were correct earlier in the day. Each now shows values published hours before the values customers last saw, and the projector's own log shows it applied those old values on purpose, minutes after the newer ones. There is no error, no retry storm, and no gap in the stream. Every update in the period was applied, and the projector never fell behind.

The accounts that broke are exactly the accounts that had an update parked. The accounts that never had one are all correct.

A capture of the affected period is in `artifacts/`.

## Requirements

- Stop a replayed update from overwriting newer state for the same account.
- Keep replaying parked updates once the dependency recovers.
- Lose no update.
- Keep accounts that never had a parked update unaffected.
- Do not modify the tests.

## Edge cases to handle

- A parked update that is still the newest for its account
- Several parked updates for one account replayed together
- A live update arriving during replay
- Per-account ordering while unrelated accounts proceed in parallel
- Replay running twice over the same parked work

## What interviewers look for

Whether convergence is enforced by comparing versions at apply time instead of trusting arrival order. A full-marks answer makes the projection monotonic per account, keeps the dead-letter path intact, and can explain why "nothing lost" and "correct final state" are different guarantees.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/dlq-replay-reorders-live-stream
