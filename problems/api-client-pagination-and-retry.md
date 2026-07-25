# API Client Pagination and Retry

Difficulty: Hard. Core topic: paginated sync, retry idempotency.

The forward-deployed-engineer incident round. A sync job that walks a vendor API page by page and retries transient failures, where the two hardest failure modes — stopping too early and replaying too much — both show up as corrupted customer data.

## Scenario

You own a customer sync job that pulls paginated account records from a vendor API through an injected transport. During a live incident, operators noticed the damage was tenant-specific: some tenants imported only their first page and nothing after it, while others ended up with duplicate records right after the vendor returned a burst of `429` responses. The transport is fine; the client's handling of opaque cursors and retryable failures is not. The repo ships the incident log and a deterministic fake transport so you can reproduce both symptoms without touching the network.

Somewhere between "follow the next cursor" and "retry on a `429`" the client loses the vendor's contract. It either treats a missing-versus-empty cursor field as an end-of-stream signal when it isn't, or it re-issues a page in a way that re-imports rows it already accepted.

## Requirements

- Walk every page by following the vendor's opaque cursor until the vendor signals the stream is genuinely exhausted.
- Distinguish "no more pages" from "cursor absent on this response" — do not stop early on the first page.
- Retry only retryable failures (such as `429`), with a bounded number of attempts.
- A retried page must not duplicate records already emitted; re-fetching is safe, re-importing is not.
- Preserve the exact vendor API contract encoded by the fake transport's responses.
- Behaviour is fully deterministic under the bundled fake transport.

## Edge cases to handle

- A first page whose cursor field is empty versus missing versus a real terminal marker.
- A transient `429` mid-stream, then a successful retry of the same page.
- Repeated `429`s that exhaust the retry budget.
- A cursor that points back at an already-seen page.
- An empty result set for a tenant with zero records.

## What interviewers look for

Whether you separate the two concerns cleanly: pagination termination (when is the stream truly done?) and retry idempotency (how do I re-attempt a page without double-counting its rows?). A full-marks answer reads the incident log first, reproduces each symptom against the fake transport, and fixes the contract rather than papering over one tenant's behaviour — because the same bug that drops page two for one tenant duplicates rows for another.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/api-client-pagination-and-retry-coding-problem
