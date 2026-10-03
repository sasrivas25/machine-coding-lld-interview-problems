# Report Exports Are Pushing Checkouts Off the Gateway

Difficulty: Hard. Core topic: admission control, flow control by request class.

The distributed-systems round. One pool serves small constant checkouts and large bursty exports, and admission treats them identically — so a scheduled export batch refuses more than half the checkouts in the window.

## Scenario

A storefront gateway runs two kinds of work through one pool: customer checkouts, which are small and constant, and report exports, which are large and arrive in bursts when someone schedules a batch. The pool runs eight jobs at once and queues twenty-four more. Anything beyond that is refused.

When a large export batch is scheduled, checkouts start failing. Checkout traffic itself has not moved and the pool is doing as much work as it ever did, but more than half the checkouts in the window are turned away, and the ones that get through are slower.

The capture in `artifacts/` breaks arrivals, completions, and refusals into 200 ms windows, split by kind of work.

## Requirements

- Protect checkouts during an export burst: their admission must not collapse because exports arrived.
- Keep exports flowing — they are real work, and refusing them is not the fix.
- Keep total pool concurrency and queue depth as configured.
- Keep checkout latency close to its healthy value during a burst.
- Do not modify the tests.

## Edge cases to handle

- A burst that would fill the queue entirely with exports
- A window with exports only, where they should use the whole pool
- A window with checkouts only
- Exports starved indefinitely by continuous checkout traffic
- Admission decisions made per class without changing total capacity

## What interviewers look for

Whether you see that fair admission requires knowing the class of the request, and reserve capacity rather than enlarging it. A full-marks answer gives each class its own share of concurrency and queue, lets either class borrow idle capacity, and can explain why a single undifferentiated queue always lets the bursty workload win.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/admission-ignores-request-class
