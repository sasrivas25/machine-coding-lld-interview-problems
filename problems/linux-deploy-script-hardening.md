# Hardening a Fragile Deploy Script

Difficulty: Medium. Core topic: shell reliability, error handling.

The shell-fundamentals round. A deploy script that reported success while leaving a half-broken release behind — it swallowed a failing precheck, hid a non-zero status inside a pipeline, and split a path with spaces into pieces. Making the script fail loudly and safely is the whole task.

## Scenario

You are on call after a deploy that claimed success but left the release broken. Reading the script, the failures are the classic shell traps: a failing precheck was ignored and execution continued, a pipeline masked the real non-zero exit status of an upstream command, and a release artifact whose path contained spaces was mishandled as multiple arguments. Fix `SHELL/solution.sh` so the deploy aborts on any command failure, aborts on unset required values, propagates non-zero status through pipelines, and treats filenames and paths with spaces as single arguments.

The tests run entirely in temporary directories, inject a failing helper command through `PATH`, and assert that a failed deploy never writes the final deployment marker and never performs the destructive cleanup step.

## Requirements

- Abort the deploy on any command failure rather than continuing.
- Abort when a required value is unset.
- Propagate a non-zero status through pipelines instead of masking it.
- Handle filenames and paths containing spaces as single arguments.
- Never write the final deployment marker after a failed step.
- Never run the destructive cleanup step when the deploy has failed.

## Edge cases to handle

- A precheck command that exits non-zero mid-script
- A pipeline whose first stage fails but last stage succeeds
- A release artifact path containing spaces
- A required variable that is empty or undefined
- The cleanup step reached only after a genuine success

## What interviewers look for

Whether you make the shell strict about failure instead of hoping each command succeeds: fail on error, fail on unset, fail through pipelines, and quote every expansion so word-splitting cannot silently mangle a path. A full-marks answer ensures the two dangerous side effects — the deployment marker and the destructive cleanup — are strictly gated behind a run that actually succeeded. The discipline is treating "the script kept going after a failure" as the bug, not a symptom.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/linux-deploy-script-hardening
