# Monthly CalVer

This repository explains how to do **monthly** calendar versioning (CalVer) in a way that remains fully compatible with [Semantic Versioning (SemVer) 2.0](https://semver.org/) and Go’s module versioning rules.

---

## Why CalVer?

- **Clear timeline**: The version number includes the year and month (`YYYYMM`), so it’s obvious when a release happened.
- **Still SemVer**: We only swap the “minor” version for a date. This is perfectly valid as long as we keep the `MAJOR.MINOR.PATCH` format.

### Base Format

vMAJOR.YYYYMM.PATCH

1. **MAJOR**: Increment only for breaking changes (in Go, that means also updating `module .../v2` in `go.mod`).  
2. **YYYYMM**: Year and month combined. Example: `202503` for March 2025.  
3. **PATCH**: Start at `0` each month and bump it for new stable releases during that month.

**Example**:

- `v1.202503.0` → First stable March 2025 release  
- `v1.202503.1` → Second stable March 2025 release

---

## Pre-release Suffixes (Alpha, Beta, RC, etc.)

Add a hyphen and the pre-release label after `PATCH`:

vMAJOR.YYYYMM.PATCH-rc1

Some common suffixes:
- `-alpha.1` / `-alpha.2`
- `-beta.1` / `-beta.2`
- `-rc1`, `-rc2`
- `-nightly.20250301` for a daily snapshot

SemVer rules treat any pre-release version as earlier than the final `vMAJOR.YYYYMM.PATCH`. For example:

v1.202503.0-rc1 < v1.202503.0

---

## Build Metadata

Optionally, append build info after a plus sign:

vMAJOR.YYYYMM.PATCH[-prerelease]?+BUILDINFO

- `v1.202503.0+build.1234`
- `v1.202503.0-rc1+linux.amd64`

Go’s module resolution ignores everything after the `+`, so these versions act like the same one in dependency terms. Use it for reference only.

---

## Multiple Releases per Month

If you patch or update multiple times within the same month:
1. `v1.202503.0`
2. `v1.202503.1`
3. `v1.202503.2`  
…and so on.

For pre-releases, just add the suffix:  
`v1.202503.2-beta.1`, leading up to `v1.202503.2` when stable.


## Example Tagging Script

Here’s a small shell script you can adapt to automatically tag a new release each month:

```bash
#!/usr/bin/env bash
# Usage: ./tag_release.sh <major> <patch>
# e.g., ./tag_release.sh 1 0 -> v1.202503.0

MAJOR=$1
PATCH=$2
YYYYMM=$(date +'%Y%m')

NEW_TAG="v${MAJOR}.${YYYYMM}.${PATCH}"
echo "Creating tag: ${NEW_TAG}"
git tag "${NEW_TAG}"
git push origin "${NEW_TAG}"
```

---

FAQ

> What happens if I use build metadata like v1.202503.0+build.999?

Go ignores the metadata for version sorting; it’s effectively the same as v1.202503.0 in terms of module resolution.

> Is this compatible with standard SemVer?

Yes — `MAJOR.MINOR.PATCH` remains intact, just that “MINOR” is the `YYYYMM` integer and “PATCH” is the monthly patch count.
