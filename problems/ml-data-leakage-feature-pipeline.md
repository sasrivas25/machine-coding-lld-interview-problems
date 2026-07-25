# Debug Data Leakage in a Retention Feature Pipeline

Difficulty: Hard. Core topic: data leakage, feature engineering.

The too-good-to-be-true round. Offline validation looks spectacular, the production canary performs at baseline, and rebuilding the same cohort after an unrelated table update quietly changes the training matrix. Every symptom points at information crossing a boundary it should never cross.

## Scenario

A subscription retention team trains a churn classifier from account snapshots. The bundled pipeline validates required fields, builds numerical and categorical model inputs, preprocesses them, and returns aligned training and validation matrices with an audit trail. Recent runs report implausibly strong offline validation while a production canary performs near baseline — and rebuilding the same cohort after routine validation-table updates can change the training matrix, even though the training snapshots themselves did not change.

Your task is to repair `PYTHON/src/feature_pipeline.py` so the returned artifact respects the supplied cohort boundary and contains only information that would be available at the moment a retention score is produced. The failure is not a single line; it spans independent stages of feature preparation, and a partial repair must not make verification pass. Preserve deterministic account ordering, a single shared feature schema, unknown-category handling, missing-value handling, label alignment, and the fit audit. Do not change the public API or the bundled tests.

## Requirements

- Respect the cohort boundary: training must not draw on information from outside it.
- Include only features available at scoring time — no future or validation-side signal.
- Preserve deterministic account ordering so runs are reproducible.
- Use one shared feature schema across training and validation.
- Handle unknown categories and missing values consistently across both splits.
- Keep labels aligned to their rows and preserve the fit audit trail.

## Edge cases to handle

- A validation-table update that must not alter the training matrix
- A category present at scoring time but unseen during fit
- Missing values in fields that also drive preprocessing statistics
- Preprocessing statistics fit on the wrong slice of data
- Row order that must stay stable for labels to align

## What interviewers look for

Whether you can trace leakage across stages rather than patching one symptom: statistics fit on the wrong split, features that encode the outcome, and a cohort boundary that is honored in name but not in the numbers. The strongest fixes make the artifact reproducible from the training snapshots alone and identical regardless of later changes to unrelated tables.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/ml-data-leakage-feature-pipeline-coding-problem