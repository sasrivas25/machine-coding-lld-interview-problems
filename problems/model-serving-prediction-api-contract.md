# Model Serving Prediction API Contract

Difficulty: Hard. Core topic: model serving, batch inference.

The ML-serving contract round. A batch prediction endpoint whose scores change after a client harmlessly reserializes equivalent JSON, and whose response records stop lining up with the inputs. Restoring the published contract without retraining is the whole task.

## Scenario

A production retention model is exposed through an in-process FastAPI batch endpoint. Two failures surface. First, valid requests receive different risk results after clients reserialize equivalent feature objects — the same features expressed in a different but equivalent JSON form produce a different score. Second, some batches return records in a different order than they were sent, or omit repeated entity identifiers so a caller can no longer map every input to its prediction.

You restore the published prediction contract without retraining or replacing the bundled model artifact. The endpoint must treat equivalent JSON objects consistently, validate the model's exact feature schema, and return exactly one prediction for every input occurrence, in request order. All verification runs over an in-process ASGI transport, so no server or network is involved.

## Requirements

- Equivalent JSON feature objects produce identical predictions.
- Validate the model's exact feature schema before scoring.
- Return exactly one prediction per input occurrence.
- Preserve request order in the response records.
- Handle repeated entity identifiers without collapsing or dropping them.
- Reuse the bundled model artifact — no retraining or replacement.

## Edge cases to handle

- The same features reserialized with reordered keys or differing numeric forms
- A batch containing the same entity id more than once
- Inputs missing a required feature or carrying an unexpected one
- A batch whose response order must match input order exactly
- A single-element batch and a large batch treated consistently

## What interviewers look for

Whether you find the serialization sensitivity — where equivalent objects are being treated as distinct before they reach the model — and normalize inputs so scoring is deterministic across JSON forms. A full-marks answer validates against the model's feature schema up front and builds the response as a strict one-to-one, order-preserving mapping over input occurrences, so repeated ids and batch order are never lost. The tell is byte-different-but-equivalent requests scoring identically, and every input finding its record.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/model-serving-prediction-api-contract-coding-problem
