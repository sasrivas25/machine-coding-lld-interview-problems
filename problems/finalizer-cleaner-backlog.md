# Finalizer and Cleaner Backlog

Difficulty: Hard. Core topic: deterministic resource release.

The native-resource lifecycle round. Thumbnail jobs finish, but scratch-file descriptors and files remain until delayed finalizers or cleaners happen to run.

## Scenario

Each thumbnail job opens a scratch file, writes and reads a page payload, computes a digest, and returns. Its wrapper has a destructor/finalizer safety net, but the normal path never closes it explicitly. Under load, wrappers are produced faster than the runtime reclaims them, exhausting descriptors and filling disk despite little managed-heap pressure.

Tests inspect open handles before forcing collection, so requesting a GC at workload end cannot pass.

## Requirements

- Close every scratch handle before its job returns.
- Delete every scratch file deterministically.
- Preserve payload bytes, digest, and job result.
- Keep finalization only as an emergency safety net.
- Report zero normal releases performed by the finalizer.

## Edge cases to handle

- Failure while writing or reading the scratch file
- Cleanup when construction only partially succeeds
- Close called more than once
- Many concurrent jobs
- Preserving the original exception if cleanup also fails

## What interviewers look for

Whether you treat file handles as scope-owned resources and use try-with-resources, context management, `defer`, or RAII. A finalizer is nondeterministic and often serialized; it cannot be the normal lifecycle for throughput-sensitive native resources.

---

Practice this in a real repo with a failing test suite → https://gronex.org/finalizer-cleaner-backlog-coding-problem
