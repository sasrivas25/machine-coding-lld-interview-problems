# Debug Batched Classification Metrics

Difficulty: Hard. Core topic: classification metrics, batch pooling.

The evaluation-harness round. Precision and recall look plausible but contradict the trusted reference, and — the giveaway — the headline scores change when the exact same predictions are shuffled into different batches. A metric that depends on batch layout is a broken metric.

## Scenario

You inherit a production model-evaluation harness that computes weighted binary-classification metrics over many prediction batches and a caller-supplied threshold sweep. The emitted confusion counts look reasonable, yet precision and recall disagree with the trusted reference. And when the same examples are repartitioned into a different batch layout, the curves move — proof that batching is leaking into the math.

The public API and the bundled tests are fixed; you repair the implementation underneath. The result must preserve threshold order, honor sample weights and the defined zero-denominator behavior, and produce byte-identical reports for every partition of identical prediction data.

## Requirements

- Metrics are computed by pooling raw counts across batches, not by averaging per-batch metrics.
- Precision, recall, and F1 match the trusted reference at every threshold.
- Sample weights are applied consistently in every confusion count.
- Threshold order is preserved in the emitted sweep.
- Zero-denominator cases follow the defined convention rather than producing NaNs by accident.
- The public API and tests remain unchanged.

## Edge cases to handle

- Identical predictions split into 1 batch vs many batches — same report.
- A threshold at which a denominator is zero.
- Batches of uneven size, including empty batches.
- Non-uniform sample weights.
- The extreme thresholds where everything is predicted positive or negative.

## What interviewers look for

Whether you spot that aggregating already-normalized ratios across batches is not the same as aggregating counts and normalizing once. A full-marks answer accumulates weighted TP/FP/FN/TN per threshold across all batches first, then derives precision and recall from those pooled totals — making the report invariant to batch layout by construction, with a deliberate, documented rule for the zero-denominator boundary.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/ml-metric-computation-pooling-bug-coding-problem