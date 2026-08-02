# Missed Signal in a Handoff Queue

Difficulty: Hard. Core topic: condition variables, shutdown.

The condition-variable round. Every page is eventually indexed, yet consumers disappear during the run and remaining threads stay parked after shutdown because wakeups are treated as durable state.

## Scenario

A parser hands pages to a pool of indexer threads through `HandoffQueue`. Consumers wait when the queue is empty; producers signal when work arrives; shutdown closes the queue. The starter implementation checks queue state and waits with incorrect synchronization semantics, allowing a notification to occur between the check and park or waking too few consumers when closing.

Thread dumps and queue counters show indexers stuck in the wait path despite available terminal state.

## Requirements

- Keep every consumer alive until explicit shutdown.
- Deliver every submitted page exactly once.
- Make waiting predicate-based and safe against spurious wakeups.
- Wake all parked consumers when the queue closes.
- Ensure shutdown joins cleanly without process hangs.

## Edge cases to handle

- A producer signaling just before a consumer waits
- Several consumers parked on an empty queue
- Shutdown with an empty or non-empty queue
- Spurious wakeups
- Submit attempts after closure

## What interviewers look for

Whether you understand that condition notifications are not queued permits. Correct solutions guard the predicate and wait atomically under the same mutex, loop after wakeup, and broadcast terminal state so every consumer can observe closure.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/missed-signal-lost-wakeup-queue
