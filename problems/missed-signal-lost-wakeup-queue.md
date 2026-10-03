# Missed Signal in a Handoff Queue

Difficulty: Hard. Core topic: condition variables, lost wakeups, shutdown.

The runtime-diagnostics round. An ingestion pipeline never finishes shutting down: consumers sit parked on a condition variable nobody wakes, and some consumers walk away from a queue that is still open.

## Scenario

An archive service ingests scanned documents. A parser stage turns each document into pages and hands them to a pool of indexer workers through a shared handoff queue. The queue is the only synchronisation point between the stages: the parser publishes pages, the indexers take pages, and a close signal tells the indexers no more pages are coming.

The pipeline is correct with one indexer and correct while pages are flowing. It breaks at the edges of the handoff. Shutdown never completes, because most indexers stay parked on the condition variable long after the queue was closed. Under a multi-indexer workload, some indexers also stop consuming while the queue is still open, so a later burst of pages has fewer workers than configured.

The supplied artifact is a thread dump taken while the fault was live, with the queue counters read at the same instant: the queue closed, thirty-one of thirty-two indexers still parked inside the wait, and exactly one wakeup delivered.

## Requirements

- Index every published page exactly once.
- Let no consumer leave while the queue is still open.
- Make closing the queue stop every consumer, with no forced interrupt.
- Keep consumers blocked while idle — a timed poll is not a repair; the suite counts wakeups and rejects spinning.
- Do not modify the tests.

## Edge cases to handle

- A page published between a consumer's predicate check and its wait
- Close signalling only one waiter instead of all of them
- A spurious wakeup leading a consumer to conclude the queue is finished
- The last page racing the close signal
- A consumer waking to an empty but still-open queue

## What interviewers look for

Whether you fix the protocol rather than the symptom: predicate re-checked in a loop under the lock, the right broadcast on publish and on close, and no reliance on timing. A full-marks answer keeps consumers genuinely blocked, accounts for every page exactly once, and can explain why a single signal is a correctness bug and not a performance nit.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/missed-signal-lost-wakeup-queue
