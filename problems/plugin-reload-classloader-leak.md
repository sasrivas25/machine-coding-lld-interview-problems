# Rule Bundle Reloads Never Release the Previous Generation

Difficulty: Hard. Core topic: hot-reload lifecycle, generation retention.

The runtime-diagnostics round. A risk scoring service hot-reloads rule bundles and keeps every generation ever loaded reachable, so memory grows with the number of publishes rather than with traffic.

## Scenario

A risk service scores card transactions. Scoring rules live in versioned bundles that operations publishes several times a day, and the service hot-reloads them without dropping traffic. Each reload builds a fresh isolated bundle scope: a new module namespace is created, the bundle source is compiled into it, and the rules it defines are handed to `RuleRegistry`, which makes them the active generation.

Alongside the active bundle, the registry keeps a table of shutdown hooks so a generation can release its resources, a bounded audit log of recent reloads, and a per-rule statistics map.

The service is healthy after a restart and degrades over days. Memory climbs in steps that line up with bundle publishes rather than traffic. On a busy week, forty-plus publishes force a recycle before the weekend. Scoring stays correct and latency is flat until the very end. Forcing a full collection releases nothing.

`artifacts/` holds evidence captured from this code running the reload workload the tests describe.

## Requirements

- Make a superseded generation unreachable once the new one is active.
- Keep memory growth tied to traffic, not to reload count.
- Keep scoring results identical and keep a reload activating the newly published generation.
- Do not change anything under `tests/`, the reload count, the audit log capacity, or the number of rules a bundle defines.

## Edge cases to handle

- Shutdown hooks keyed per generation and never removed
- A per-rule statistics map keyed by objects from the old generation
- The audit log holding whole bundles instead of identifiers
- A reload that fails part-way, leaving a half-registered generation
- Scoring in flight against the generation being replaced

## What interviewers look for

Whether you follow the retaining chain from the evidence to the registry's own bookkeeping, rather than assuming the module namespace is the leak. A full-marks answer deregisters every per-generation structure on activation, keeps the bounded log genuinely bounded in bytes as well as entries, and shows memory flat across many reloads.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/plugin-reload-classloader-leak
