# Scalable Calendar Versioning v1.2025.1

We can flexibly “stretch” calendar versioning (CalVer) to meet different release frequencies. You can start with yearly (`vMAJOR.YYYY.PATCH`), then switch to monthly (`vMAJOR.YYYYMM.PATCH`), and even go daily (`vMAJOR.YYYYMMDD.PATCH`). This approach remains compatible with SemVer and Go’s module rules, as long as you treat the date part (`YYYY`, `YYYYMM`, or `YYYYMMDD`) as your “MINOR” number and handle `PATCH` normally.

## 1. Problem Statement
Sometimes you need a CalVer format that adapts to changing release frequencies. For example, you might begin with yearly releases (e.g., `v1.2025.0`) but later require more frequent updates—monthly (e.g., `v1.202503.0`), daily (e.g., `v1.20250301.0`), etc. Having an adaptable versioning scheme is crucial for managing different release cadences.

## 2. SemVer Compatibility
- **MAJOR**: Increment only for breaking changes. In Go, this means also updating the module path (e.g., `module example.com/v2`).
- **DATE**: Replaces the conventional “MINOR” component and can be formatted as:
  - **Yearly**: `YYYY` (e.g., `2025`)
  - **Monthly**: `YYYYMM` (e.g., `202503`)
  - **Daily**: `YYYYMMDD` (e.g., `20250301`)
  
  SemVer treats this field as a plain integer. All formats remain compatible with [Semantic Versioning 2.0](https://semver.org/) and Go’s module versioning.
- **PATCH**: Increments for each stable release within the chosen date period.
  > **Typical Scenario**: Going from `v1.2025.1` (yearly) to `v1.202503.0` (monthly) is usually fine, because `202503` is larger than `2025`. But if you ever have a conflict (e.g., a weird prior version like `v1.999999.0`), bumping MAJOR might be the simplest fix.

## 3. Yearly Format
- `v1.2025.0` → The first stable release in 2025.
- `v1.2025.1` → A subsequent release in 2025.
- **Usage**: Best for projects with infrequent major updates where the year alone sufficiently identifies the release period.

## 4. Monthly Format
- `v1.202503.0` → The first stable release in March 2025.
- `v1.202503.1` → The second stable release in March 2025, etc.
- **Note**: When transitioning from a yearly to a monthly format, verify that the new date value (e.g., `202503`) numerically exceeds the previous yearly version (`2025`). Under typical circumstances, it will, but if a bizarre old version number makes it smaller, you may need to bump the MAJOR to keep your version ordering valid.

## 5. Daily Format
- `v1.20250301.0` → The first stable release on March 1st, 2025.
- `v1.20250301.1` → A subsequent release on the same day.
- **Usage**: Ideal for projects requiring extremely frequent updates, as this format clearly specifies the exact release date.

## 6. Pre-release Suffixes
- Append identifiers such as `-alpha`, `-beta`, `-rc`, or even `-nightly.<date>`.
  - Examples: `v1.202503.0-alpha.1` or `v1.202503.0-rc1`
- **SemVer Rule**: Pre-release versions are considered lower precedence than their final release counterparts (e.g., `v1.202503.0-rc1 < v1.202503.0`).

## 7. Build Metadata
- Append build metadata after a plus sign. For example:
  - `v1.202503.0+buildinfo`
  - `v1.202503.0+linux.amd64`
- **Go Module Behavior**: Go ignores metadata for version comparisons, treating `v1.202503.0+buildinfo` as equivalent to `v1.202503.0`.

## 8. Multiple Releases in the Same Period
- **Yearly Releases**: `v1.2025.0`, `v1.2025.1`, `v1.2025.2`, etc.
- **Monthly Releases**: `v1.202503.0`, `v1.202503.1`, `v1.202503.2`, etc.
- **Daily Releases**: `v1.20250301.0`, `v1.20250301.1`, etc.
- **Increment**: Simply increase the PATCH number for each additional release within the period.

## 9. Script Example
Below is an enhanced shell script that demonstrates tagging a release using any of the three date formats:

```bash
#!/usr/bin/env bash
# Usage: ./tag_release.sh <major> <dateformat> <patch>
# - <major>: e.g. 1
# - <dateformat>: e.g. $(date +'%Y') for yearly, $(date +'%Y%m') for monthly, or $(date +'%Y%m%d') for daily
# - <patch>: e.g. 0 (for the first release of that period)
# Example: ./tag_release.sh 1 $(date +'%Y%m') 0 -> v1.202503.0

MAJOR=$1
DATEFORMAT=$2
PATCH=$3

NEW_TAG="v${MAJOR}.${DATEFORMAT}.${PATCH}"
echo "Creating tag: ${NEW_TAG}"
git tag "${NEW_TAG}"
git push origin "${NEW_TAG}"
```
