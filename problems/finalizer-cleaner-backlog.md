# Finalizer and Cleaner Backlog in a Scratch File Service

Difficulty: Hard. Core topic: deterministic resource release, native handles.

The runtime-diagnostics round. A thumbnail service delegates all resource release to finalization, so descriptors stay open and scratch files stay on disk long after the jobs that created them returned.

## Scenario

A rendering service produces thumbnails. Each job opens a scratch file, writes a page payload into it, reads it back, computes a digest, and finishes. The scratch file wrapper closes the descriptor and deletes the file when it is destroyed, so the resource is always released eventually.

Eventually is the problem. Under load the service opens scratch files faster than the runtime reclaims the wrappers — and the wrapper participates in a reference cycle, so release depends entirely on the collector. Descriptors stay open and files stay on disk past the completion of the job that owned them.

Captured runtime evidence is in `artifacts/`, with `artifacts/CAPTURE.md` recording how it was taken. Read it before changing code.

## Requirements

- Make release deterministic: a job that has returned has already released everything it opened.
- Keep digests, byte counts, and job results unchanged.
- Leave zero handles open after the workload, without forcing a collection.
- Leave no scratch files on disk.
- Release no handle via the finalizer — it may remain as a safety net only.
- Add no third-party dependencies, and do not edit the tests.

## Edge cases to handle

- A job that raises part-way through, before the digest is computed
- The reference cycle keeping the wrapper alive past its scope
- Forcing a collection at the end of the workload — handles are counted before that
- Double release when both the scope exit and the safety net fire
- Concurrent jobs holding their own scratch files

## What interviewers look for

Whether release becomes a scoped, explicit lifecycle rather than a hope about collector timing. A full-marks answer closes and deletes at the end of the job on every path, breaks the cycle so the safety net never has work to do, and can explain why finalizers are unsuitable for bounded resources like file descriptors.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/finalizer-cleaner-backlog
