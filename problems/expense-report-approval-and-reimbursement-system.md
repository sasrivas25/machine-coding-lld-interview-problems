# Expense Report Approval and Reimbursement System

Difficulty: Hard. Core topic: approval routing, budget state.

The multi-level approval round. The amount on the report decides how many approvers it needs, each of them signs in order, a department budget is drawn down exactly once when the last approval lands, and the reimbursement must be issued a single time — never twice, never after a rejection.

## Scenario

You inherit a partially implemented expense report approval and reimbursement backend. The models, repositories, and services exist, and the visible tests describe threshold-based routing, ordered sequential approvals with role checks, atomic budget consumption on final approval, and one-time reimbursement with rollback on rejection.

The behavior is subtly wrong in the places that cost money. Reports route to the wrong number of approvers for their total, approval steps can be taken out of order or by the wrong role, the department budget is decremented at the wrong moment or more than once, and reimbursement can be issued twice or survive a rejection. Several visible tests fail until the routing, sequencing, and budget logic are correct.

## Requirements

- Route each report to an approval chain determined by its total amount.
- Enforce sequential, ordered approval steps with role-based access at each step.
- Consume the department budget atomically, only on final approval.
- Issue exactly one reimbursement per fully approved report.
- Roll back cleanly on rejection: no budget drawn, no reimbursement issued.
- Preserve the existing public method contracts; do not modify the tests.

## Edge cases to handle

- A report that sits exactly on a routing threshold boundary
- An approver attempting a step out of sequence or above their role
- Final approval racing against a concurrent budget change
- A rejection arriving after some approvals already landed
- A repeated final-approval or reimbursement call

## What interviewers look for

Whether routing, sequencing, and settlement stay separate concerns, and whether the budget decrement and reimbursement are treated as a single atomic effect of the final approval rather than side effects sprinkled through the flow. Full marks require the money-moving steps to be exactly-once and fully reversible when the report is rejected.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/expense-report-approval-and-reimbursement-system