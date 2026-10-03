# Off-Heap Buffer Churn and the Full GC Storm

Difficulty: Hard. Core topic: off-heap memory, buffer pooling, explicit collection.

The runtime-diagnostics round. A gateway allocates a fresh 256 KiB off-heap buffer per message and calls for a collection after each one, so 92% of wall clock is spent in full collections that reclaim about a megabyte each.

## Scenario

A messaging gateway serialises outbound messages through a 256 KiB off-heap buffer before they reach the wire, because the regions are large and must not sit in the managed heap.

The serializer allocates a new off-heap buffer per message. The region behind it is only reclaimed when the small on-heap handle owning it is collected, and the pressure from a few small handles is nowhere near enough to trigger that. The code compensates by asking the runtime to collect after every message. The captured JVM evidence shows 1024 messages producing 1024 full collections, each attributed to an explicit request:

```
GC(0) Pause Full (System.gc()) 3M->1M(124M) 1.714ms
time spent collecting      : 1126 ms of 1217 ms wall clock
```

The C++ capture shows the same defect with no collector to soften it: 1024 buffers allocated, none released, 256 MiB of mappings still outstanding at the end of the workload.

Serialized output is correct throughout — the workload digest is stable and every message digest matches. The fault is entirely in how buffers are obtained and returned.

## Requirements

- Reuse a fixed pool of buffers instead of allocating per message.
- Release buffers deterministically, so off-heap memory stops tracking message count.
- Remove the explicit collection request — but note that removing it alone is not enough.
- Keep output digests, the digest algorithm, and payload generation unchanged.
- Do not change the tests, `verify.sh`, or the pinned runtime flags.

## Edge cases to handle

- A message larger than the pooled buffer size
- Returning a buffer on the exception path
- Resetting buffer state between uses so no bytes leak between messages
- Concurrent serialisation drawing from the same pool
- Pool exhaustion behaviour

## What interviewers look for

Whether you see two coupled defects — the churn and the explicit collection used to paper over it — and fix the cause. A full-marks answer pools and resets buffers with deterministic release, keeps allocation count flat as messages scale, and can explain why off-heap regions are invisible to heap pressure heuristics.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/offheap-buffer-churn-full-gc-storm
