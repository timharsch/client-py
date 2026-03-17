# FOSSology License Gate -- Design

**Date:** 2026-03-17
**Status:** Proposed

## Goal

Prevent any GPL, copyleft, or non-commercial licensed source code from entering
`main` on `timharsch/client-py` by running FOSSology's `nomos` scanner as a
GitHub Actions check.

## Context

This fork diverges from `smart-on-fhir/client-py` and periodically pulls
upstream changes. Since upstream may introduce dependencies or code with
incompatible licenses, an automated gate is needed to catch violations before
they land on `main`.

## Approach

Run FOSSology's `nomos` license scanner (via Docker) against the full source
tree on every push to `main` and every PR targeting `main`. Detected licenses
are compared against an allowlist. Any non-allowlisted license fails the check.

### Why FOSSology / nomos

- Scans actual file contents, not just dependency metadata
- Detects license text embedded in source files (copy-pasted GPL headers, etc.)
- Regex-based, fast enough for CI (~1-3 min for this repo)
- Industry-standard tool used by Linux Foundation projects

### Why Allowlist (not Blocklist)

An allowlist is safer: unknown or obscure licenses are rejected by default.
A blocklist risks missing novel copyleft variants.

## Architecture

```
Push to main / PR opened
  -> checkout code
  -> pull fossology/fossology:scanner Docker image
  -> run `nomos -r /scan -J` against repo
  -> parse JSON output
  -> compare each detected license against .github/license-allowlist.txt
  -> PASS: all licenses allowlisted -> check succeeds
  -> FAIL: violations found -> check fails, summary posted as PR comment
```

## Allowlist

Stored in `.github/license-allowlist.txt`, one SPDX identifier per line:

```
Apache-2.0
MIT
BSD-2-Clause
BSD-3-Clause
ISC
Unlicense
CC0-1.0
PSF-2.0
No_license_found
```

`No_license_found` is allowlisted because most source files in this repo do not
contain embedded license headers. Only positively identified non-allowed licenses
should block.

## Workflow

File: `.github/workflows/license-check.yaml`

**Triggers:**
- `push` to `main`
- `pull_request` targeting `main`

**Steps:**
1. Checkout code
2. Run `nomos` scanner via `fossology/fossology:scanner` Docker image
3. Parse JSON report, filter out binary file extensions
4. Compare detected licenses against allowlist
5. On failure: exit non-zero and (for PRs) post a comment listing violations

**Concurrency:** Cancel in-progress runs for the same branch.

## Edge Cases

| Case | Behavior |
|------|----------|
| Binary files (.png, .jpg, .whl) | Skipped by extension filter |
| Dual-licensed file | Fails if any detected license is not allowlisted |
| File with no license detected | Passes (`No_license_found` is allowlisted) |
| New allowlist entry needed | Add to `.github/license-allowlist.txt`, re-run |
| Upstream sync introduces violation | Push to main fails; must resolve before merging |

## Files to Create

| File | Purpose |
|------|---------|
| `.github/workflows/license-check.yaml` | GitHub Actions workflow |
| `.github/license-allowlist.txt` | Allowed SPDX license identifiers |

## Open Questions

- Should we cache the FOSSology Docker image between runs to speed up CI?
- Should we scan only changed files (faster) or full repo (safer)? Current
  design scans the full repo.
