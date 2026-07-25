# API Contract Debugging: Input Validation & Error Envelope

Difficulty: Medium. Core topic: API contract, validation.

The contract round. A handler that returns 200 with a garbage body when it should return 400 with a documented error envelope, and sets the wrong Content-Type on the paths that matter most. Honoring the request/response contract to the byte is the whole task.

## Scenario

You are handed an in-process Orders API modeled as one function: a request struct with `method`, `path`, `headers`, and `body` goes in, a response with `status`, `headers`, and `body` comes out. No server, no network, fully deterministic. The single route `POST /orders` has a documented contract — a valid body yields a 200 with a created-order object; an invalid body yields a 400 with a standard error envelope listing every offending field; unknown paths and wrong methods yield 404 and 405 with the same envelope shape.

The shipped handler breaks that contract. On invalid input it wrongly returns 200 with an empty or garbage body instead of the 400 envelope, and on some error paths it sets the wrong Content-Type. The bundled contract tests assert status, body shape, and headers for the valid case and each invalid case, and they fail on `main`.

## Requirements

- Valid requests return 200, `application/json`, and the exact created-order body.
- Invalid requests return 400 with the `VALIDATION_ERROR` envelope and a `fields` array naming every field that failed.
- Missing fields, wrong-typed fields, and out-of-range `quantity` all count as invalid.
- Unknown paths return 404 `NOT_FOUND`; wrong methods on a known path return 405 `METHOD_NOT_ALLOWED`.
- Every response, success or error, carries `Content-Type: application/json`.
- The error envelope shape is identical across all failure codes.

## Edge cases to handle

- A body missing several fields at once (all must appear in `fields`)
- `quantity` present but zero, negative, or non-integer
- A correctly shaped body sent with the wrong method
- A well-formed request to a path the API does not expose
- Empty or non-object request bodies

## What interviewers look for

Whether validation is a single pass that collects all failures before responding, rather than bailing on the first bad field or, worse, letting invalid input fall through to a success path. A full-marks answer keeps one envelope builder for every error code, validates against the documented types and ranges, and never leaks a 200 for input it could not honor. The discipline is treating the written contract as the specification and matching it exactly — status, body, and headers.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/api-validation-and-error-shape-debugging
