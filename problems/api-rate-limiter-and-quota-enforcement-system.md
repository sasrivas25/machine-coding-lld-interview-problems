# API Rate Limiter and Quota Enforcement System

Difficulty: Hard. Core topic: token-bucket concurrency.

The rate-limiter round. Token-bucket semantics with an injected clock, refill-then-decide ordering, atomic single-token consumption, strict per-bucket isolation, and a hard guarantee that concurrent callers to one bucket never admit more requests than capacity.

## Scenario

You are given a backend for an API rate limiter that already knows what a bucket is: a capacity, a refill rate, and a token count keyed by scope and client. The structure is sound but the decisions are wrong. Refills happen at the wrong moment so a request is judged against stale token counts, the bucket sometimes drifts above capacity or below zero, buckets for different clients bleed into each other, and under concurrent calls to a single bucket more requests are admitted than the capacity allows. The service layer needs correct token-bucket logic and proper per-bucket locking.

## Requirements

- Refill the bucket based on the injected timestamp before every allow/deny decision.
- Cap tokens at capacity; never let the count exceed it or fall below zero.
- On an allowed request, atomically consume exactly one token.
- Isolate buckets strictly by scope plus client key.
- Keep decisions deterministic when timestamps are equal.
- Never admit more than capacity under concurrent calls to the same bucket.

## Edge cases to handle

- Two requests carrying the identical timestamp
- A long-idle bucket that would refill past capacity
- The first request against a bucket that has no prior state
- Concurrent admits racing on the last remaining token
- Distinct clients under the same scope that must not share tokens

## What interviewers look for

Whether refill, check, and consume are one indivisible operation guarded per bucket, not three steps a competing thread can wedge itself between. Strong answers scope the lock to the individual bucket so unrelated clients stay parallel, derive available tokens purely from the injected clock, and hold the capacity and non-negativity invariants under the worst interleaving. The test is simple to state and hard to fake: capacity is a ceiling no burst of concurrency can breach.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/api-rate-limiter-and-quota-enforcement-system
