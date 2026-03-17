# Fork Maintenance Guide

## Overview

`timharsch/client-py` is a deliberate divergence from `smart-on-fhir/client-py`.
This fork adds features (e.g., multi-FHIR-version support) and enforces license
compliance that are not intended to be contributed back upstream.

## Remote Setup

| Remote | Repository | Purpose |
|--------|-----------|---------|
| `origin` | `git@github.com:timharsch/client-py.git` | Our fork (push target) |
| `upstream` | `git@github.com:smart-on-fhir/client-py.git` | Source repo (pull-only) |

## Syncing from Upstream

Upstream changes are pulled **manually** and **selectively**. There is no
automated sync.

```bash
git fetch upstream
git checkout main
git merge upstream/main    # or: git rebase upstream/main
# resolve conflicts, favoring our fork's changes where they diverge
git push origin main
```

### When to Sync

- When upstream publishes a new release
- When upstream fixes a bug relevant to our usage
- Before starting new feature work (to reduce future merge conflicts)

### Conflict Resolution

Our fork's additions take priority over upstream. Specifically:

- `fhirclient/models/` directory structure (R4 subdirectory layout) is ours
- `fhirclient/_version_registry.py` is ours
- `fhirclient/models/__init__.py` backward-compatible imports are ours
- `.github/workflows/license-check.yaml` does not exist upstream

## License Compliance

All code merged into `main` is scanned by FOSSology's `nomos` scanner via
GitHub Actions. Only allowlisted licenses (Apache-2.0, MIT, BSD, ISC, etc.)
are permitted. See `.github/license-allowlist.txt` for the full list.

This gate runs on every push to `main` and every PR targeting `main`. Code
with GPL, AGPL, SSPL, CC-BY-NC, or any other copyleft/non-commercial license
will be rejected.

**After syncing from upstream**, the license check runs automatically on push.
If upstream introduces a dependency or file with a non-allowed license, the
push will fail and must be resolved before merging.

## Rules

1. **Never push to `upstream`** -- origin only
2. **Never create PRs against `upstream`** from this fork
3. **All merges to `main` must pass the FOSSology license gate**
4. **Upstream syncs are manual** -- review changes before merging
