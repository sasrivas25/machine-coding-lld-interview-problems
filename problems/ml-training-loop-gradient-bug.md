# Debug a Logistic Training Loop

Difficulty: Hard. Core topic: gradients, training loop.

The ML-debugging round. A compact NumPy logistic-regression trainer that emits plausible loss curves yet fails gradient checks and drifts under equivalent mini-batch layouts — the failure mode that looks fine until you compare it to ground truth.

## Scenario

A model training service runs a NumPy logistic-regression loop for fast CPU-only experiments before larger jobs are scheduled. The recent runs look healthy on the surface — loss values that decrease and read as reasonable — but the learned parameters are unstable across batch layouts that should be equivalent, and they fail checks against a trusted objective. The gradient the loop computes does not match the gradient of the objective it claims to optimize, so the trainer converges to the wrong place, or to different places depending on how the batches were arranged. Your job is to repair the training implementation without touching its public API or its bundled tests.

## Requirements

- Compute gradients that match the configured objective under a gradient check.
- Produce results stable across equivalent mini-batch layouts.
- Keep parameter histories finite and deterministic.
- Preserve the intercept's semantics exactly.
- Converge consistently on the supplied synthetic classification workloads.
- Do not change the public API or the bundled tests.

## Edge cases to handle

- Batch layouts that partition the same data differently
- The intercept term, which must not be regularized or scaled like a weight
- Numerically extreme inputs that could produce non-finite values
- Determinism across repeated runs with the same seed
- A loss that looks reasonable while the gradient is subtly wrong

## What interviewers look for

Whether you diagnose against a trusted objective rather than trusting a loss curve that merely trends downward. A full-marks answer verifies the gradient with a finite-difference check, isolates where the update diverges from the objective's true derivative, and reasons carefully about intercept handling and batch aggregation. The lesson is that a plausible loss is not a correctness proof; the gradient must be the gradient of the thing you are actually optimizing.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/ml-training-loop-gradient-bug-coding-problem
