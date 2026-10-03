# Off-Heap Buffer Churn and Full-GC Storm

Difficulty: Hard. Core topic: buffer pooling, explicit collection.

The off-heap diagnostics round. Every message allocates a fresh native buffer and explicitly asks the runtime to collect so its tiny managed handle might release the region.

## Scenario

A messaging gateway serializes outbound payloads through 256 KiB off-heap buffers. The starter serializer allocates one buffer per message and relies on handle collection for reclamation, then calls explicit GC after every message. Captures show thousands of full cycles reclaiming almost no managed heap while external memory climbs.

Deleting the collection call removes the pause storm but leaves unbounded native memory. Releasing per message bounds memory but still performs an expensive map/unmap for every message.

## Requirements

- Allocate a fixed pool of reusable off-heap buffers.
- Return each buffer deterministically on success and failure.
- Remove explicit collection from the normal path.
- Release all pooled native memory when the serializer closes.
- Preserve every message and workload digest.

## Edge cases to handle

- Serialization failure after acquiring a buffer
- Batches larger than the pool
- Concurrent callers waiting for a buffer
- Close with buffers checked out
- Preventing use-after-return or stale payload bytes

## What interviewers look for

Whether you solve both halves: bounded pooling and deterministic release. GC is not a native-resource manager, and reducing buffer size alone does not remove allocation churn. Ownership and pool lifecycle must be explicit.

---

Practice this in a real repo with a failing test suite → https://gronex.org/offheap-buffer-churn-full-gc-storm-coding-problem
