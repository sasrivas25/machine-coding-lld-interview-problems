# Lock Leaked on an Exception Path

Difficulty: Hard. Core topic: exception safety, critical sections.

The thread-dump round where one malformed request is rejected correctly and every request after it hangs forever. The failing path exits without releasing the service's serialization lock.

## Scenario

A settlement service uses one lock because downstream ledger operations must never overlap. Normal requests acquire, validate, apply, and release. A validation exception returns the correct error to its caller but jumps past the explicit unlock.

The captured dump shows subsequent workers blocked at lock acquisition while no active worker is making progress. Restarting recreates the lock and temporarily clears the symptom.

## Requirements

- Release the lock after every acquired path.
- Continue rejecting invalid or unbalanced requests.
- Preserve exactly-once settlement accounting.
- Avoid allowing two settlements into the critical section.
- Keep the lock held only for the operations that require serialization.

## Edge cases to handle

- Validation throwing immediately after acquisition
- Application failure halfway through settlement
- Cleanup when acquisition itself did not succeed
- Multiple callers arriving after the first failure
- Error reporting that must preserve the original exception

## What interviewers look for

Whether you connect the dump to exception-unsafe lock ownership and replace paired manual calls with structured locking—`finally`, a context manager, synchronized scope, or RAII. Adding timeouts converts a permanent hang into repeated failure without repairing ownership.

---

Practice this in a real repo with a failing test suite → https://gronex.org/problems/lock-leaked-on-exception-path
