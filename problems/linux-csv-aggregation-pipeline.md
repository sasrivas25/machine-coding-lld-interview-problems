# CSV Group-By Aggregation in the Shell

Difficulty: Medium. Core topic: shell text processing.

The shell-fundamentals round. Group a CSV by one column, sum another, sort the result by a compound key — all with standard shell tools and not a line of pandas.

## Scenario

You are given `SHELL/fixtures/sales.csv` with the header `region,product,amount` and asked to write `SHELL/solution.sh` that reduces it to a per-region total. Each row's `amount` is an integer count of paise; your script sums those per region and emits one line per region as `region<TAB>total` on stdout. The header row is data, not a total, and must be skipped. The output is sorted by total descending, and where two regions tie on total, those rows break the tie by region ascending. The whole thing is a small pipeline of ordinary tools — no dataframe library allowed.

## Requirements

- Read the CSV and skip the header row.
- Group rows by `region` and sum the integer `amount` values.
- Emit one line per region as `region`, a tab, then the total.
- Sort output by total descending.
- Break ties on equal totals by region ascending.
- Use shell tools only; do not use pandas.

## Edge cases to handle

- A region appearing in non-adjacent rows across the file
- Two or more regions sharing the same total
- Integer sums that must not drift into floating point
- The header line never contributing to any region's total
- Exact `region<TAB>total` formatting, tab-separated, no stray columns

## What interviewers look for

Whether you reach for the right tool for a grouped sum and compose a clean pipeline instead of hand-rolling parsing. A strong answer gets the compound sort right — primary key descending, secondary key ascending — handles the header without a special case that leaks into the data, and keeps arithmetic in integers. It is a small task that rewards fluency: knowing which tool aggregates, which sorts, and how to chain them precisely.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/linux-csv-aggregation-pipeline
