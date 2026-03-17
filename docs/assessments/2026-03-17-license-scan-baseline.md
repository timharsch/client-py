# License Scan Baseline Assessment

**Date:** 2026-03-17
**Branch:** main
**Commit:** 2438a85c195eecc87371d78d8871062e22464892
**Scanner:** FOSSology nomos (via `fossology/fossology:scanner` Docker image)
**Scan scope:** Full repository, excluding `.venv/`, `.git/`, `__pycache__/`, `.egg-info/`

## Result: CLEAN

All source code in the repository passes the license allowlist check.

## Scan Summary

| License | Files | Status |
|---------|-------|--------|
| Apache-2.0 | 7 | Allowed |
| See-doc.OTHER | 5 | Informational |
| CC0-1.0 | 2 | Allowed |
| NoWarranty | 2 | Informational |
| Public-domain | 1 | Allowed |
| Non-commercial | 1 | False positive (excluded) |

**Total files scanned:** 1,006

## Allowlist

```
Apache-2.0
MIT
BSD
BSD-2-Clause
BSD-3-Clause
ISC
Unlicense
CC0-1.0
CC0
PSF-2.0
Python
0BSD
No_license_found
Public-domain
NoWarranty
See-URL
See-doc.OTHER
```

## Excluded File

`tests/data/examples/measure-cms146-example.json` is excluded from license
enforcement. This FHIR specification example file contains NCQA measure
copyright text referencing "noncommercial purposes" in its metadata. This is
not a source code license -- it is embedded healthcare measure copyright
language from the FHIR R4 specification examples. The file itself is
distributed as part of the HL7 FHIR standard test data.

## Notes

- The `.venv/` directory contained 124 AGPL-flagged files from installed
  Python packages. These are runtime dependencies and not part of the
  repository source. The GitHub Action will not scan `.venv/` since it
  runs against a fresh checkout.
- The `See-doc.OTHER` and `See-URL` detections are informational markers
  from nomos indicating it found a reference to an external license
  document, not a specific license identification.

## Scanner Configuration

```bash
docker run --rm --platform linux/amd64 \
  -v "$REPO:/opt/repo" \
  --entrypoint fossologyscanner \
  fossology/fossology:scanner \
  nomos scan-dir --dir-path /opt/repo --report TEXT
```
