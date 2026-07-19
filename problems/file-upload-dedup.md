# File Upload Deduplication and Multipart Finalization

Difficulty: Hard. Core topic: content-addressed storage, reference counting.

The Dropbox / Drive-style question, reduced to its hardest 20%: finalizing multipart uploads correctly and deduplicating identical content without ever deleting bytes someone still references.

## Scenario

You maintain the backend of an upload service. Clients start multipart uploads, send parts, and finalize; the service checksums content, deduplicates identical files, and cleans up storage when files are deleted. All the pieces exist, and all of them are broken: finalize succeeds with missing or corrupt parts, two concurrent uploads of identical content create duplicate blobs (or one loses data), deleting a deduplicated file strands or destroys the shared bytes, and abandoned uploads leak storage forever.

## Requirements

- Finalize verifies every part is present and its checksum matches before assembly; anything else is rejected.
- Blobs are keyed by content hash; two users uploading the same file store it once.
- Two identical uploads finalizing concurrently converge to one blob with two references.
- Deletion decrements a reference count; bytes are physically removed only at zero references.
- Finalize is idempotent: a retried completion cannot corrupt or duplicate state.
- Abandoned-upload cleanup reclaims storage without ever touching live data.

## Edge cases to handle

- Finalize called with a part missing or corrupted
- The dedup race: same content, two finalizes, one stored copy
- A delete racing a new upload of the same content
- Retried finalize after a success
- Cleanup running while an upload is still in progress

## What interviewers look for

Finalize as a validation gate followed by a commit, with the content hash as the identity. Cleanup is where careless designs lose data: the blob's reference count must be the single source of truth, incremented atomically on link, decremented on delete, with physical removal only at zero. The delete-racing-upload interleaving is the case that always gets probed.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/file-upload-deduplication-and-multipart-finalization-system-coding-problem
