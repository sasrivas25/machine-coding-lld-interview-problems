# Resumable Multipart Upload

Difficulty: Hard. Core topic: multipart upload, dedup.

The architecture-extension round. A content store already dedups whole-file uploads by hash; your job is to grow it into resumable multipart upload — parts arriving out of order, retried idempotently, resumed after an interruption, then assembled, verified, and deduped exactly like a single put.

## Scenario

`FileStore` is an in-memory content store. It already supports whole-file uploads via `put(name, bytes) -> fileId` and `get(fileId)`, and it deduplicates identical content by hash — uploading the same bytes twice returns the same id and stores the content once. That base behavior is complete and its tests pass.

Large files cannot arrive in one `put`, so you extend the store with multipart sessions. The multipart methods are stubbed and their tests fail. The challenge is holding the session invariants: parts may land out of order, the same part number may be retried, a client may need to resume by asking which parts are present, and completion must assemble in ascending part order, verify the total size, and then fold into the existing hash-based dedup rather than storing a redundant copy.

## Requirements

- `init_upload(name, total_size) -> uploadId` starts a session.
- `upload_part(uploadId, partNumber, bytes)` accepts parts out of order; re-uploading a part number overwrites it (idempotent retry).
- `list_parts(uploadId)` returns the part numbers received so far, so a client can resume.
- `complete_upload(uploadId) -> fileId` assembles parts in ascending order, fails if any part is missing or the assembled size != `total_size`, and on success stores and dedups exactly like `put`.
- `abort_upload(uploadId)` discards the session.

## Edge cases to handle

- Parts arriving out of order and a retried part overwriting the previous bytes.
- Completing with a gap in part numbers, or with an assembled size mismatch.
- Resuming after interruption via `list_parts`.
- Completing a file whose content already exists — dedup returns the existing id.
- Operating on an aborted or unknown `uploadId`.

## What interviewers look for

Whether the extension reuses the existing hashing and dedup path instead of duplicating it, and whether session state is kept isolated and cleanable. A full-marks answer treats `complete_upload` as a verified reduction — order the parts, concatenate, check the size contract, then hand the assembled bytes to the same dedup logic `put` uses — so multipart and whole-file uploads converge on one content-addressed store.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/resumable-multipart-upload-extension