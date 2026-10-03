# Two Nodes Ran the Same Schedule

Difficulty: Hard. Core topic: leader election, quorum, fencing by term.

The distributed-systems round. An eight-hundred-millisecond partition left two nodes both believing they held the schedule, and the shared runner accepted jobs from both.

## Scenario

Three nodes take turns running a batch schedule, and exactly one may dispatch jobs at any moment. Leadership is won by election: a node campaigns for a new term, asks the other two to vote, and takes the schedule if it wins enough votes. Once it holds the schedule it sends heartbeats so the others stay quiet. Jobs come from a shared pointer so a replacement leader resumes where its predecessor stopped, and every job reaches a shared runner tagged with the term of the leader that submitted it.

One module makes two decisions: whether a campaign has won, and whether the runner should accept a job carrying a given term.

The network then split for roughly eight hundred milliseconds, isolating one node from the other two before healing. During the split two nodes both believed they held the schedule and both dispatched work. Nothing crashed, no message was corrupted, and every job ran to completion.

The evidence contains every vote request and whether the network carried it, every job the runner accepted with its submitting term, each node's log including how it tallied the votes it received, and a timeline of when each node believed it was leader. Reading the timeline and then the tally line belonging to the node that should never have won is the shortest path to the defect.

## Requirements

- Make it impossible for two nodes to hold the schedule simultaneously.
- Refuse work submitted by a leader that has since been superseded.
- Keep availability: when a leader genuinely dies, a replacement takes over and finishes the remaining jobs.
- Never elect on fewer votes than a majority — refusing to elect anyone is not a solution.
- Do not modify the tests.

## Edge cases to handle

- A campaign where vote requests were never carried by the network
- Votes counted that include the candidate's own
- A job submitted under an old term arriving after a new leader exists
- The partition healing, with the isolated node learning of a higher term
- A genuine leader death requiring a prompt replacement

## What interviewers look for

Whether the vote tally requires a strict majority of the configured cluster, and whether the runner fences on term. A full-marks answer rejects stale terms at the runner, elects only on quorum, and keeps a real failover working — the two halves of the same defect.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/split-brain-under-network-partition
