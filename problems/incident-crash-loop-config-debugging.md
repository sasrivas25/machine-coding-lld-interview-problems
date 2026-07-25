# Incident Debugging: Crash Loop from a Bad Config

Difficulty: Easy. Core topic: config validation, crash loop.

The 3am pager round. A service is crash-looping in production right after a release, the supervisor has given up and marked it FATAL, and the root cause is a config field the deploy legitimately omits. Reading the log and fixing boot-time validation is the whole task.

## Scenario

Your pager goes off at 03:11 UTC: `checkout-gateway` is down and crash-looping. A release rolled out minutes earlier, and since then the supervisor spawns the service, it dies during boot, gets restarted, and dies again. The production log captured during the incident is bundled at `logs/incident.log` in each language project — you read it the way you would on call, finding the boot attempt, the stack trace, and the restart loop.

The service reads its deploy config from an injected map, so everything is deterministic: no real environment, no network, no processes. The contract is simple — `PORT` is required and integer, `DB_URL` is required and non-empty, `CACHE_TTL` is optional and defaults to 300. The current boot code enforces none of it safely: it treats the optional key as required (which is what took prod down), parses `PORT` with no error handling, and blindly dereferences `DB_URL`.

## Requirements

- Boot succeeds and reports healthy with the exact production config from the incident.
- `PORT` is required and must parse as an integer.
- `DB_URL` is required and must be non-empty.
- `CACHE_TTL` is optional and falls back to 300 when omitted.
- Genuinely invalid configs fail with a single clear `ConfigError` (`ConfigException` in Java) naming the offending key.
- No raw `KeyError`/`ValueError`, NPE, or `std::out_of_range` ever escapes boot.

## Edge cases to handle

- The optional `CACHE_TTL` absent (the production case)
- `PORT` present but not parseable as an integer
- `DB_URL` missing or empty
- A config that is valid in every field (must boot healthy)
- An invalid field surfacing exactly one named error, not a raw exception

## What interviewers look for

Whether you diagnose from the log before touching code, then make boot-time validation explicit: required keys checked, optional keys defaulted, and every failure funneled into one typed error that names what was wrong. A full-marks answer distinguishes "optional and absent" from "required and missing," and never lets a low-level parse or lookup exception be the thing that pages someone. The recovery signal is the suite going green because the service boots, not because you suppressed an error.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/incident-crash-loop-config-debugging
