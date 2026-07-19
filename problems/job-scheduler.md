# Distributed Job Scheduler with Singleton Execution

Difficulty: Hard. Core topic: distributed coordination, leases.

A senior-round staple. The part that actually gets graded is narrow: when several workers race to claim the same scheduled job, exactly one must run it, and a worker that dies mid-run must not lock the job forever.

## Scenario

You are given a system that simulates a fleet of workers sharing a job store. Jobs become due, workers poll and claim them, run them, and report completion. The coordination code exists — and under concurrent workers it falls apart. Two workers claim and execute the same job, a crashed worker leaves its job claimed forever, expired leases get honoured as if still valid, and re-run attempts double-execute work that already completed.

## Requirements

- Claiming a job is atomic: a worker observes "due and unclaimed" and transitions it to "claimed by me with a lease deadline" in one step.
- Leases expire, so a dead worker's job becomes claimable again.
- A worker must hold a valid lease at execution and at completion, not just at claim time.
- Completion is exactly-once in effect, even when attempts overlap.
- Job lifecycle: due → claimed → running → done, with no illegal transitions.

## Edge cases to handle

- N workers polling the same due job simultaneously — exactly one winner
- A worker crashing after claim but before completion
- A slow worker whose lease expires mid-run, racing the reaper that reclaims it
- A reclaimed job completing twice (original worker and the new claimant)
- The reaper reclaiming a job that just completed

## What interviewers look for

Whether you treat the claim as the critical section (check-and-take as one operation, never check then take), and whether expiry and completion are defensive: a worker finishing a job proves it still holds a valid lease before recording the result. Thinking of the lease as a fencing token checked at every effect is what separates a passing solution from one that merely looks right. The honest phrasing of "exactly once" is "at least once plus idempotency."

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/distributed-job-scheduler-and-singleton-execution-system-coding-problem
