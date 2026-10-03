# Cache Hit Rate Collapses Every Time a Node Is Added

Difficulty: Hard. Core topic: consistent hashing, membership change cost.

The distributed-systems round. Growing a four-node cache to five remapped almost every key at once, so the hit rate fell to nothing and the database sat at its connection ceiling for the rest of the morning.

## Scenario

A session cache is spread across four nodes. Each node can work out which node owns a given key on its own, without asking anyone, which is what keeps a lookup cheap. The scheme is even: every node holds close to a quarter of the keys and nothing has ever been unbalanced.

The cluster was grown to five nodes to add headroom before a campaign. The change itself was uneventful and the cluster stayed balanced. The twenty minutes that followed were not: the cache hit rate fell to almost nothing, every miss became a database read, and the database spent the morning at its connection ceiling. The same thing happened when a node was drained for patching a week earlier, and it will happen again the next time capacity changes.

Adding a node should cost roughly the new node's share of traffic while it fills, and nothing else. The keys that already had a home should keep it. Instead almost every key ended up somewhere new, so almost every cached entry became unreachable at once.

The captured evidence shows which node owned which key before and after the change, how many keys changed hands, and how many of those moved between two nodes that were both already in the cluster. The distribution is even before and after, so this is not a balance problem.

## Requirements

- Make a membership change cost only what it has to: keys move to or from the changed node, not between existing ones.
- Keep keys spread evenly across nodes.
- Keep ownership computable by any node alone — no shared table, no coordination.
- Keep lookup cost unchanged.
- Do not modify the tests.

## Edge cases to handle

- Adding a node, and the share of keys it should take
- Removing a node, and where its keys should go
- Balance quality with a small number of nodes
- Deterministic agreement on ownership across all nodes
- Repeated membership changes in sequence

## What interviewers look for

Whether you recognise modulo-by-node-count as the defect and replace it with a scheme whose movement is proportional to the change. A full-marks answer keeps distribution even (virtual nodes, or an equivalent), proves that existing nodes do not trade keys with each other, and preserves coordination-free lookup.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/key-remapping-on-cluster-resize
