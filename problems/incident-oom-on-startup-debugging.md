# Incident Debugging: OOM Crash-Loop on Startup

Difficulty: Medium. Core topic: memory budget, lazy loading.

The 3am pager round. A service is crash-looping in production, memory climbs while it logs the same line over and over, and it dies before serving a single request. The captured log tells you exactly where it dies — the skill is reading it, and choosing a boot path that fits the budget.

## Scenario

At 03:41 UTC you are paged: `catalog-svc` is crash-looping and never becomes healthy. The health probe returns 503 in a loop, memory climbs while the service logs `preloading full catalog into memory`, and at roughly 65% of the preload the process dies — OOM-killed, `OutOfMemoryError`, or `std::bad_alloc` depending on language — having served zero requests. The captured production log is bundled at `logs/incident.log` in each language folder; read it first.

The service is modeled in-process and deterministically, with no real large allocation. `CatalogService(source, budget)` receives a catalog source — a factory that opens a fresh streaming scan over 100,000 records (ids `sku-000001` through `sku-100000`), which you may open as often as you like — and a `MemoryBudget` that simulates memory: the service calls `allocate` before retaining a record and `release` when it drops one, and exceeding the limit raises `OutOfMemoryBudget`. Production runs a 32 MiB budget against a ~51.2 MB catalog. It does not fit. On `main`, `./verify.sh` fails everywhere because the boot path blows the budget before serving anything.

## Requirements

- `start()` completes within the budget with no `OutOfMemoryBudget` on the full 100k catalog.
- `health()` reports `healthy` after startup.
- Lookups return correct records anywhere in the catalog, including the very last ids.
- Unknown ids return a not-found result.
- Registered memory stays within the budget across a sequence of lookups, even under a deliberately tiny budget.
- Do not edit the tests, the log, `MemoryBudget`, or the catalog source.

## Edge cases to handle

- A lookup for one of the final ids that a partial preload never reached
- A budget far smaller than production forcing tighter retention
- Repeated lookups that must not accumulate unreleased memory
- Opening fresh scans on demand instead of holding the whole catalog

## What interviewers look for

Whether you let the log point you at the exact failing step instead of guessing, and whether you replace an eager whole-catalog preload with an access path that pays memory only for what it actually needs — releasing what it no longer holds. The strongest fixes serve correct lookups across the entire id range while keeping registered memory bounded under any budget.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-oom-on-startup-debugging