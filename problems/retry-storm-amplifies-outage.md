# A Struggling Dependency Received Three Times Its Normal Load

Difficulty: Hard. Core topic: retry budgets, backoff, failure classification.

The distributed-systems round. Retrying everything, immediately, means the dependency gets more traffic precisely because it is failing — and fewer requests succeed than if nobody had retried at all.

## Scenario

A checkout API calls a pricing service. Requests arrive one every ten milliseconds. Most failures from pricing are transient and worth another attempt; some are outright rejections of a malformed request that will fail identically however many times they are sent. A retry policy decides whether a failed call is attempted again and how long to wait first.

The pricing service then had a period of unavailability. It recovered on its own, but the outage lasted far longer than the underlying fault, and throughout it pricing received substantially more traffic than the checkout API was actually asked to serve.

Fewer requests succeeded during the outage than would have succeeded during a comparable period of plain unavailability with no retrying at all — the opposite of what retrying is for.

The evidence records calls reaching the dependency per 100 ms window and how many calls each request generated, each request with its attempt count and duration, every call and retry sent, and the client's log.

## Requirements

- Ensure a dependency in trouble does not receive more load because it is in trouble.
- Keep generated traffic inside the retry budget the policy declares.
- Keep retrying effective: a transient failure is retried and succeeds.
- Make every request eventually settle.
- Do not modify the tests.

## Edge cases to handle

- A permanent rejection that must not be retried at all
- A transient failure that clears on the second attempt
- Many requests failing in the same window and retrying together
- Backoff with jitter so retries do not re-synchronise
- The budget boundary, where further retries must be refused

## What interviewers look for

Whether failures are classified before they are retried, and whether retry volume is capped as a fraction of traffic rather than per request. A full-marks answer applies capped backoff with jitter, skips retries that cannot succeed, and can explain why retry amplification makes recovery slower than no retries at all.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/retry-storm-amplifies-outage
