# A Stalled Writer Overwrote the New One

Difficulty: Hard. Core topic: leases, fencing, pause tolerance.

The distributed-systems round. A writer stalls for half a second, loses its lease, resumes, and writes anyway — the store knows nothing about leases and applies whatever it is handed.

## Scenario

Two writers maintain a single shared document, and only the lease holder may write it. A lease service hands the lease to one writer at a time for a fixed duration, stamping every grant with an increasing epoch. The holder renews periodically; if it stops renewing the lease lapses and the other writer may take it. The document store is a plain key-value store that knows nothing about leases.

Each writer runs a guard that is told when a lease is granted and when it is lost. The guard decides whether a revision may be issued, and whether a finished revision may be applied.

One writer stalled for half a second during a long garbage collection pause. Its lease lapsed while it was stopped, the other writer took over, and the handover looked correct from outside. When the stalled writer resumed it wrote anyway: two revisions from the new holder were overwritten by older content, and the document came to rest holding a revision from a writer that had not held the lease for hundreds of milliseconds.

The evidence contains the lease timeline with epochs and windows, every revision applied to the document in order, every revision as issued, and each writer's log.

## Requirements

- Let no revision reach the document unless its issuing writer still holds the lease when the write lands.
- Keep a healthy writer making progress.
- Keep a stall shorter than the lease from costing a writer its work.
- After a long stall, let the other writer take over and continue.
- A guard that refuses everything is not a solution.

## Edge cases to handle

- A revision issued under epoch N landing after epoch N+1 was granted
- A stall shorter than the lease duration
- The handover instant, with both writers believing they hold the lease
- Renewal failing while work is in flight
- Repeated takeovers across several epochs

## What interviewers look for

Whether you fence the write with the epoch at the store rather than trusting a local guard's view of lease state. A full-marks answer rejects writes carrying a stale epoch, keeps both writers productive, and can explain why a pause makes every local "do I hold the lease?" check untrustworthy by the time the write lands.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/expired-lease-write-after-pause
