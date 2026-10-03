# Rule Bundle Reloads Retain Previous Generations

Difficulty: Hard. Core topic: classloader lifecycle, hot reload.

The hot-reload heap round. Rule evaluation stays correct, but each deployment leaves an isolated module or classloader generation reachable, producing stepwise memory growth tied to reload count.

## Scenario

A risk engine compiles each published rule bundle into a fresh isolated scope, installs it as active, and keeps shutdown hooks, audit history, and rule statistics. After forty reloads, heap evidence shows old scopes and rule objects reachable through lifecycle registries rather than through live traffic.

Full collection releases nothing because the old generations remain strongly referenced. Doubling the reload count doubles retained state.

## Requirements

- Keep only the active generation and intentionally bounded metadata reachable.
- Remove lifecycle hooks and registrations belonging to retired generations.
- Execute generation shutdown exactly as required.
- Preserve scoring and activation of newly published rules.
- Keep retention constant as reload count increases.

## Edge cases to handle

- Reload failure before the new generation becomes active
- Shutdown hooks that capture generation objects
- Statistics keyed by generation-owned rule instances
- Concurrent scoring during an atomic generation swap
- Closing the registry after many reloads

## What interviewers look for

Whether you trace the oldest retained rule back to the owning registry and clean every generation-keyed side structure during replacement. Clearing only the active pointer is insufficient when hooks, caches, or metrics still retain the old scope.

---

Practice this in a real repo with a failing test suite → https://gronex.org/plugin-reload-classloader-leak-coding-problem
