# Two Workers Render the Same Document at Once

Difficulty: Hard. Core topic: visibility timeouts, lease renewal.

The distributed-systems round. The queue hides a job for a fixed period, larger uploads made renders outlast it, and the only thing keeping two workers off a document quietly stopped holding.

## Scenario

A render fleet takes document jobs from a queue. When a worker takes a job, the queue hides it from the other workers for a fixed period, so exactly one worker is responsible for it at a time. If the worker dies, the job reappears and someone else takes it. This is the only mechanism keeping two workers off the same document, and it has been in place since the fleet was built.

Customers on large documents have started seeing renders that flicker between two versions, and the storage team has raised the cost of the bucket twice this quarter. Instrumentation shows the fleet publishing more renders than there are jobs, all of them for the biggest documents. No job is ever lost, no worker crashes, no error is logged, and the queue never reports a failure.

The fleet was fine until the product team started accepting larger uploads.

A capture of one busy period is in `artifacts/`, including a timeline of which worker held which job and when.

## Requirements

- Publish every job exactly once.
- Keep a job whose worker dies picked up by another worker.
- Keep large documents rendering successfully, not just safely.
- Keep the queue's own contract unchanged — the fix belongs in how the worker holds the job.
- Do not modify the tests.

## Edge cases to handle

- A render that takes longer than the visibility period
- A worker dying mid-render, where the job must reappear
- Renewal failing part-way through a long render
- The instant a second worker takes a job the first still believes it owns
- Publishing guarded so a superseded owner cannot write

## What interviewers look for

Whether the worker extends its claim while it still holds it, and whether the publish is fenced so a lapsed owner cannot write. A full-marks answer renews within the visibility window, abandons work when renewal fails, keeps genuine crash recovery, and can explain why a fixed timeout tuned for small documents is a correctness bug once input size grows.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/visibility-timeout-shorter-than-handler
