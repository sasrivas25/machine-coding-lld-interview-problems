# API Contract Debugging: Idempotent POST /orders

Difficulty: Medium. Core topic: idempotency, HTTP contracts.

The retry-safety round. A payment button gets double-tapped, a mobile request times out and fires again, and the same intent arrives twice. The header that is supposed to make that safe is sitting there, ignored, and every retry mints a fresh order.

## Scenario

You inherit a small order API modeled entirely in process — no HTTP server, no framework, no network. A `Request` carries a `method`, `path`, `headers`, and a `body`; a `Response` carries a `status`, `headers`, and a `body`. `OrderService.handle(request)` routes the request, and `POST /orders` creates an order and returns `201` with the new order's `id`. The tests call the handler directly and assert on both the response and the stored order count, so behavior is deterministic.

The endpoint is meant to be idempotent by the `Idempotency-Key` request header, so a client can safely retry a create. The current handler never reads the header: it creates a new order on every call. A timed-out request that retries, or a double-tapped "Pay", quietly produces a second order — a double charge. The no-key and distinct-key tests already pass; the idempotency tests fail until you honor the key.

## Requirements

- Same key plus the same payload returns the same order id and does not create a duplicate.
- Same key plus a different payload is a defined conflict: respond `409`, create nothing new.
- A request with no key creates normally; each such call is its own order.
- Distinct keys produce distinct orders.
- The stored order count reflects exactly the orders that were truly created.
- Preserve the existing `Request`/`Response` shapes and routing.

## Edge cases to handle

- A replayed key arriving after the first response was already returned
- A reused key whose body differs in a field that changes the order
- Interleaved requests that share a key versus requests with no key at all
- A retry that carries the key but was never seen before

## What interviewers look for

Whether you treat the idempotency key as a first-class part of the contract rather than an afterthought: a stored mapping from key to the original outcome, a defined answer when the same key returns with a different payload, and a create path that stays untouched when no key is present. Full marks come from making a replay return the recorded result verbatim instead of re-running the side effect.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/api-idempotency-contract-debugging