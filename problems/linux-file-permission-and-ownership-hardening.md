# File Permission & Secret Hardening

Difficulty: Easy. Core topic: file permissions, hardening.

The Linux fundamentals round. A service refuses to start because its config tree ships with sloppy permissions. Fix the provisioning script so every file and directory lands in a secure, exact mode — no world-readable secrets, no group-writable surprises.

## Scenario

`solution.sh` provisions an app config tree under a target directory passed as an argument. As written it leaves the tree in an unsafe state: the secret file is readable or writable by too many, directories carry loose bits, and some paths are group- or other-writable — enough for the service to refuse to start. The tests create a temporary target directory, run the script against it, and assert the resulting mode bits.

Your job is to correct the script so that, after it runs, every path in the created tree has precisely the mode it should, with no writable bits leaking to group or other.

## Requirements

- The secret file ends at mode `600`.
- The config file ends at mode `644`.
- Every directory in the created tree ends at mode `755`.
- No file or directory under the tree is group-writable.
- No file or directory under the tree is other-writable.
- The script works against whatever target directory argument the tests pass.

## Edge cases to handle

- Nested directories that must each end at `755`, not just the top one
- A secret file that must not be readable by group or other
- Umask or default modes that leave stray writable bits
- Re-running against a fresh temporary target on each test invocation
- Distinguishing file modes from directory modes when applying bits

## What interviewers look for

Whether the candidate sets modes explicitly and deliberately rather than trusting the umask, and treats the secret's `600` as a hard requirement separate from ordinary config. A clean answer applies file and directory modes distinctly, verifies nothing under the tree is group- or other-writable, and stays correct no matter which target path the harness supplies.

---

Practice this in a real repo with a failing test suite (free) → https://gronex.org/problems/linux-file-permission-and-ownership-hardening
