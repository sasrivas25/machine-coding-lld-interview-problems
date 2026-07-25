# Access Log Triage & Request Analysis

Difficulty: Easy. Core topic: shell log analysis.

The on-call shell round. A raw combined access log, core text tools, and a triage report whose exact output contract — sorting, tie-breaks, and status ranges — is the whole test.

## Scenario

You are handed an Apache/nginx combined access log at `SHELL/fixtures/access.log` and asked to implement `SHELL/solution.sh` so it prints an on-call triage report to stdout. The report has two parts: the top client IPs by request count, and a count of server errors. The bundled Bats tests encode the exact output contract, so the format is not negotiable — the difficulty is precision, not volume.

The traps are the usual log-parsing ones: getting the ranking stable when counts tie, pulling the right field out of the combined format, and counting only the status codes that actually mean a server error.

## Requirements

- Parse the combined-format access log at `SHELL/fixtures/access.log`.
- Print the top 5 client IPs by request count, one per line as `count<TAB>ip`.
- Sort by descending request count, with a deterministic ascending IP tie-break.
- Print a final line `5xx=N` counting only status codes 500 through 599.
- Read the status from the correct field of the combined log format.
- Match the exact output contract the Bats tests expect.

## Edge cases to handle

- Two or more IPs with identical request counts (tie-break by ascending IP).
- Fewer than five distinct IPs in the log.
- Status codes at the range boundaries (499 and 600 excluded; 500 and 599 included).
- Lines whose status field must not be confused with other numeric fields.
- Tab versus space as the field separator in the output.

## What interviewers look for

Whether you compose small, correct shell tools into a deterministic pipeline instead of reaching for something heavier. A full-marks answer nails the sort keys and tie-break explicitly, scopes the 5xx count to exactly 500–599, and treats the output format as an exact contract — because in triage, a report that's "roughly right" is one nobody can trust at 3 a.m.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/linux-log-triage-and-request-analysis
