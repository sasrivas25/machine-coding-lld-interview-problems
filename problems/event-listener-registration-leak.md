# Event Listener Registration Leak

Difficulty: Hard. Core topic: heap retention, listener lifecycle.

The heap-dump round where closed collaboration sessions remain alive and every publish becomes slower. A process-wide event bus still owns callbacks that capture objects whose user-visible lifecycle has ended.

## Scenario

A document collaboration server opens and closes editing sessions throughout the day. Each session subscribes a listener to a long-lived `ChangeBus`, receives a subscription token, and owns a document buffer and revision history. The close path removes the session from the active-session map but never unregisters its callback.

Heap evidence shows closed sessions retained through bus subscribers. Delivery count grows much faster than publish count because stale listeners continue receiving changes.

## Requirements

- Unregister a session's listener when that session closes.
- Release document buffers and session state after lifecycle end.
- Deliver each change exactly once to each currently open session.
- Keep the bounded journal and document index behavior unchanged.
- Make retained state independent of total historical sessions.

## Edge cases to handle

- Closing a session twice
- Publishing concurrently with session closure
- A session that fails during initialization
- Reopening the same document in a new session
- Listener cleanup when an exception occurs

## What interviewers look for

Whether you follow the heap's retaining path to the long-lived publisher and treat subscription as a resource with explicit ownership. Forcing collection cannot reclaim reachable callbacks; lifecycle symmetry between subscribe and unsubscribe is the repair.

---

Practice this in a real repo with a failing test suite → https://gronex.org/event-listener-registration-leak-coding-problem
