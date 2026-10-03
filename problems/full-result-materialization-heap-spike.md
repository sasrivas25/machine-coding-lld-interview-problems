# Full Result Materialization Heap Spike

Difficulty: Hard. Core topic: streaming, bounded peak memory.

The runtime-diagnostics round. A CSV export builds the entire result set in memory before writing a byte, so peak heap scales with the table instead of the chunk size.

## Scenario

A reporting service exports large query results to CSV for download. Operators report that a single export of a large table drives the service heap to its ceiling and holds it there for the whole export, while a small export of the same table is untroubled. The heap returns to normal once the export finishes — so nothing leaks; the peak is simply proportional to the data.

A heap capture taken part-way through a large export ships with the code. The export is already chunked in configuration; the code just does not honour it end to end.

## Requirements

- Bound peak live memory by the configured chunk size.
- Keep peak memory flat as the dataset grows.
- Produce byte-identical CSV output, including header and row order.
- Do not change the tests, the workload sizes, or the configured chunk size.

## Edge cases to handle

- An intermediate list or string buffer that re-materialises the full result
- A generator consumed into a collection before being written
- Datasets smaller than one chunk
- The final partial chunk
- Flush behaviour so the writer does not accumulate the whole body

## What interviewers look for

Whether you trace the data path from query to response and find every place it is fully realised, rather than fixing the first one and assuming the rest. A full-marks answer streams from cursor to writer with a constant-size working set and proves peak memory is independent of dataset size.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/full-result-materialization-heap-spike
