# Job Queue: Delayed Jobs & Priority Ordering

Difficulty: Medium. Core topic: scheduling, priority queue.

The extend-without-breaking round. A working FIFO queue needs two new powers — jobs that only become dispatchable at a future time, and a priority order among the jobs that are ready — without disturbing the plain FIFO behavior that already passes.

## Scenario

You are given a small, correct in-memory FIFO job queue with `enqueue(job)`, `dequeue(now) -> job`, and `size()`. Your task is to extend it with delayed jobs and priority ordering, all inside the single stubbed `dequeue` method per language, while the existing FIFO tests keep passing.

Jobs may carry a `run_at` time and an integer `priority` (default `0`). A delayed job is only dispatchable once `now >= run_at`; `dequeue(now)` must skip jobs that are not yet due and signal empty when nothing is currently dispatchable — but a not-yet-due job still counts toward `size()`. Among the due jobs, a higher priority number wins, ties break by insertion order, and a high-priority job that is not yet due must never jump ahead of a due lower-priority one. Time is injected via `now` everywhere, so the bundled tests run with no wall clock and no sleeping.

## Requirements

- `dequeue(now)` never returns a job whose `run_at` is still in the future.
- Signal empty (raise/throw) when no due job exists, even if future jobs remain.
- Among due jobs, return the highest priority first (higher number is higher priority).
- Break priority ties by insertion order — stable FIFO.
- Not-yet-due jobs still count toward `size()`.
- Preserve the original FIFO behavior for plain jobs.

## Edge cases to handle

- A high-priority job that is not yet due sitting behind a due low-priority job
- Several due jobs at the same priority requiring stable FIFO order
- `now` exactly equal to a job's `run_at`
- A queue holding only future jobs — dispatchable-empty but non-zero size

## What interviewers look for

Whether due-ness is checked before priority so scheduling can never be undercut by a ranking, whether ties stay stable rather than depending on container quirks, and whether the empty signal distinguishes "nothing dispatchable now" from "truly nothing left". Clean answers keep the injected `now` as the single source of time and leave the original FIFO path intact.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/job-queue-delayed-and-priority-extension