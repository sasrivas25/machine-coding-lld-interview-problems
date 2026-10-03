# A Read Returned Data the Cluster Had Already Replaced

Difficulty: Hard. Core topic: quorum reads, read repair.

The distributed-systems round. Writes are acknowledged at two of three replicas, but reads do not gather a quorum and never repair what they find stale — so the lagging replica serves replaced data and never catches up.

## Scenario

A replicated key-value store runs three replicas with no primary. A coordinator sends every write to all three and acknowledges once two have applied it, giving a write quorum of two out of three. Reads go through the same coordinator, which decides which replicas to ask, how to turn their answers into a single value, and what to do about a replica found to be behind.

One replica stopped receiving writes for a period. Writes continued to be acknowledged correctly, because two replicas were still applying them.

Reads began returning data the cluster had already replaced. Some keys were reported absent even though they had been written and acknowledged well before the read. When the replica started receiving writes again, the keys it had missed stayed missing, and the replicas never came back into agreement.

The evidence records which replicas answered each read and what each replica ends up holding, the version returned by each read against the version the cluster had acknowledged, every write sent to every replica with whether it landed, and the coordinator's log.

## Requirements

- Never return a version older than a write the cluster has acknowledged.
- Repair a replica found to be behind during a read, so it does not stay behind.
- Keep the cluster available: two of three replicas are enough to serve a read.
- Keep serving reads while one replica is unreachable.
- Do not modify the tests.

## Edge cases to handle

- A single replica answering a read while it is the stale one
- A key absent on some replicas and present on others
- Choosing the winning version among differing answers
- Repair failing on a replica that is unreachable
- One replica permanently down, with reads still served

## What interviewers look for

Whether reads gather enough replicas to intersect the write quorum, and whether repair is performed on the read path rather than deferred to a sweeper that does not exist. A full-marks answer satisfies R + W > N, reconciles by version, repairs the laggard, and keeps availability with one replica down.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/quorum-read-without-repair
