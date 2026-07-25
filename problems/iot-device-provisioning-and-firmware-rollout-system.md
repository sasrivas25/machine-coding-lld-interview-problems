# IoT Device Provisioning and Firmware Rollout System

Difficulty: Hard. Core topic: device lifecycle, staged rollout.

The fleet-management round. Device state machines, deterministic staged rollouts, an automatic halt-and-rollback when firmware starts failing, and hard tenant isolation — where a single leaky boundary can ship a bad update to the wrong fleet.

## Scenario

You inherit a partially implemented IoT device provisioning and firmware rollout backend. It moves devices through registration, provisioning, activation, and retirement; runs staged firmware rollouts that target a deterministic percentage of a tenant's active devices; halts and rolls back automatically when the observed failure rate crosses a rollout's threshold; and isolates tenants and cohorts so a rollout never touches another tenant's devices. Several visible tests fail because some repository and service logic is incomplete or incorrect.

The failures cluster around the hard parts: a lifecycle transition that shouldn't be legal, a "deterministic percentage" that isn't reproducible, a failure-rate check that halts too late or not at all, and an isolation boundary that lets a rollout reach across tenants or cohorts.

## Requirements

- Move devices through the lifecycle — registration, provisioning, activation, retirement — allowing only legal transitions.
- Select a deterministic percentage of a tenant's active devices for each staged rollout.
- Halt a rollout and roll back when the observed failure rate exceeds its threshold.
- Enforce strict multi-tenant and cohort isolation so a rollout never touches devices outside its scope.
- Preserve the existing public method contracts; do not modify the tests.
- Fix the implementation so all visible tests pass without rewriting the app.

## Edge cases to handle

- An illegal lifecycle transition (e.g. activating a retired device).
- A rollout percentage that must select the same devices given the same inputs.
- A failure rate landing exactly on versus just over the halt threshold.
- A rollout that must ignore another tenant's or cohort's active devices.
- Rollback after a partial rollout that already updated some devices.

## What interviewers look for

Whether you read the models, repositories, and tests to recover the intended behaviour before changing anything. A full-marks answer treats the lifecycle as an explicit state machine, makes staged selection deterministic and reproducible, and defends the tenant/cohort boundary as an invariant rather than a filter that's easy to forget — because in a fleet system, an isolation leak isn't a failing test, it's a bad firmware push to the wrong customer.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/iot-device-provisioning-and-firmware-rollout-system
