# Humongous Allocation Region Pressure

Difficulty: Hard. Core topic: allocation shape, large-object thresholds, chunking.

The runtime-diagnostics round. A rendering service asks for one contiguous buffer sized to the whole payload, every buffer crosses the collector's large-object threshold, and cycles fire repeatedly at low heap occupancy.

## Scenario

A document rendering service materialises scanned documents. A request names a document and its page count; the service builds the page payload, digests it, and returns the digest with the byte count. Rendering is pure computation, no I/O. `RenderService` is the entry point, and `BufferAllocator` is the only source of buffers — it records how many it handed out, how large they were, and peak bytes outstanding.

The pipeline's output is correct. The allocation shape is wrong. Every request asks for a single contiguous buffer sized to the entire payload, and those buffers routinely cross the large-object threshold. Crossing it takes the allocation off the normal path: the request becomes a hunt for a contiguous run of regions, and the collector runs repeatedly even though almost nothing is live.

`artifacts/` was captured while the fault was live: every allocation crosses the threshold, and peak footprint tracks the largest payload rather than the data actually in flight.

## Requirements

- Keep rendered output exactly as it is — every document digest is pinned.
- Let no allocation reach the large-object threshold.
- Keep peak buffer footprint within budget.
- Keep the volume allocated above the threshold within budget.
- Treat the threshold as a property of the service, not of the runtime; `verify.sh` is fixed.

## Edge cases to handle

- Documents whose payload is just under versus just over the threshold
- The last chunk of a payload that does not divide evenly
- Digesting incrementally so chunking does not change the result
- Buffers released promptly so peak footprint reflects work in flight
- Single-page documents

## What interviewers look for

Whether you fix the shape of the allocation rather than the size of the heap. A full-marks answer chunks the payload below the threshold, streams the digest across chunks so output is untouched, and can explain why one 8 MB buffer is worse for a region-based collector than many small ones.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/humongous-allocation-region-pressure
