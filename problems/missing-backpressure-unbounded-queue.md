# Indexing Latency Grew Without Bound During a Worker Slowdown

Difficulty: Hard. Core topic: backpressure, admission control.

The distributed-systems round. Nothing was lost and nothing failed — the intake simply accepted every event into an unbounded queue, so latency grew for as long as the slowdown lasted and every caller timed out.

## Scenario

An ingest API accepts events and hands them to a pool of four indexing workers. Events arrive one every five milliseconds, and each normally takes fifteen milliseconds to index, so four workers are comfortably sufficient. A single component decides what happens to an event when it is offered.

The workers slowed down, first moderately and then severely. They never stopped, and every event was eventually indexed, so nothing was lost. What happened instead is that end-to-end latency grew for as long as the slowdown lasted and kept growing: events arriving during the slow period were still being indexed long after submission, each one worse off than the last. Callers waiting on indexing timed out. Adding workers moved the point at which growth started without changing the fact that it grew.

The evidence records queue depth and busy workers sampled every 50 ms, each indexed event with its end-to-end latency, every event offered with the depth it saw, and the intake's log.

## Requirements

- Degrade in a bounded way: keep latency within budget while the workers are slow.
- Keep the backlog within the capacity the service is provisioned for.
- Absorb a brief slowdown rather than shedding during it.
- Shed load only when there is genuinely no room — never when there was room.
- Account for everything: nothing may disappear unexplained.

## Edge cases to handle

- A brief slowdown that the queue should absorb entirely
- A severe, sustained slowdown where shedding is correct
- The boundary where the queue is exactly full
- A rejected event reported to the caller rather than dropped silently
- Recovery, where the backlog must drain and admission resume

## What interviewers look for

Whether you bound the queue and make rejection explicit, rather than adding capacity. A full-marks answer distinguishes absorbing a transient from shedding a sustained overload, keeps latency bounded by queue depth times service time, and can explain why an unbounded queue converts an overload into an unbounded latency problem.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/missing-backpressure-unbounded-queue
