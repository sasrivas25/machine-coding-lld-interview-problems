# Airflow Revenue DAG: Early Publish and Retry Inflation

Difficulty: Hard. Core topic: DAG dependencies, idempotent load.

The on-call data round. A daily revenue pipeline that publishes before all its inputs are visible, and a warehouse model that grows every time you replay the same business date — even when nothing upstream changed. Fan-in ordering and idempotency are the whole exam.

## Scenario

You inherit a deterministic, in-process DAG that loads one business-date partition and publishes a revenue summary. The source inputs are complete snapshots for that partition. Two things are subtly wrong. First, the summary task sometimes runs before every upstream task it depends on has actually finished, so it publishes against a partial view of the interval. Second, replaying the same date under a fresh attempt appends rather than replaces: warehouse row counts climb even though the source snapshots are byte-for-byte identical.

The incident is captured in `PYTHON/logs/incident.log`, and the DAG plus warehouse model live in `PYTHON/src/daily_pipeline.py`. Your job is to make fan-in honest and loads convergent, without touching the tests or the log.

## Requirements

- Every fan-in task observes all of its declared upstream work before it runs.
- A replay of the same table and partition converges to the latest complete snapshot, not an accumulation.
- Corrections and removals present in the newer snapshot are reflected in the loaded partition.
- Unrelated tables and unrelated dates remain untouched by a replay.
- The published summary matches the loaded order and customer snapshots exactly.
- Loading is deterministic given the same inputs and business date.

## Edge cases to handle

- The summary task scheduled before a slow upstream branch completes
- A second attempt for a date that already has loaded rows
- A newer snapshot that drops rows the previous one contained
- Two tables sharing a business date, only one being replayed
- Duplicate delivery of the same snapshot for one partition

## What interviewers look for

Whether you separate "has all my upstream work completed?" from "what does this load do to the partition?" A full-marks answer makes fan-in wait on the true dependency set and makes each partition load a replace-by-key operation keyed on table and business date, so replay is a no-op when inputs are unchanged and a clean overwrite when they are not. The tell is idempotency: run the DAG twice, get identical warehouse state and an identical published summary.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/airflow-dag-dependency-and-idempotent-load-coding-problem
