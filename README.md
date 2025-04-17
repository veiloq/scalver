# Scalable Calendar Versioning (ScalVer) v1.2025.3

**TL;DR:** `MAJOR.YYYY[MM[DD]].PATCH`

* 1.2025.0 < 1.2025.1 < 1.2025.2  
* 1.202503.0 < 1.202503.1 < 1.202503.2  
* 1.2025.0 < 1.202503.0 < 1.20250301.0  
* 1.2025.0 < 1.2026.1 < 2.2026.0  
* 1.20250410.0 < 2.2026.1 < v3.20270310.0  
* v1.2025.0 < v1.2025.1 < v1.2025.2 (with `v` tag)  

Format progression → `MAJOR.YYYY.PATCH` → `MAJOR.YYYYMM.PATCH` → `MAJOR.YYYYMMDD.PATCH`

## 1. Problem Statement
Projects accelerate from annual releases to monthly sprints and daily hot‑fixes.  
[CalVer](https://calver.org/) hides breaking changes, [SemVer](https://semver.org/) hides when a build shipped.

**ScalVer** fuses both while remaining **fully SemVer‑compatible**:

* **Date context** — see *when* at a glance.  
* **API context** — MAJOR still signals breaking changes.  
* **Elasticity** — one scheme from day 1 through high‑tempo CI/CD.

## 2. SemVer Compatibility

| Segment | SemVer role | Rules & tips |
|---------|-------------|--------------|
| **MAJOR** | `MAJOR` | Increment for breaking changes **or whenever a shrink in DATE would be required**. |
| **DATE**  | `MINOR` | Accepts `YYYY`, `YYYYMM`, or `YYYYMMDD` (UTC). **Must never shrink.** Numeric comparison ensures `202503` > `2025`. |
| **PATCH** | `PATCH` | Increment for every additional stable build within the current DATE. |

## 3. Yearly Format
* `1.2025.0` → first stable release in 2025  
* `1.2025.1` → subsequent release in 2025  

## 4. Monthly Format
* `1.202503.0` → first March 2025 release  
* `1.202503.1` → second March 2025 release  

## 5. Daily Format
* `1.20250301.0` → first build on 1 Mar 2025  
* `1.20250301.1` → follow‑up build the same day  

## 6. Pre‑release Suffixes
* `1.202503.0-alpha.1` < `1.202503.0-beta.1` < `1.202503.0`  
* `1.202503.0-rc1` < `1.202503.0`  

## 7. Build Metadata
* `1.202503.0+linux.amd64`  
* `1.202503.0+build.42ab1ef`  

## 8. Multiple Releases in the Same Period
* **Yearly** → `1.2025.0`, `1.2025.1`, `1.2025.2`  
* **Monthly** → `1.202503.0`, `1.202503.1`, `1.202503.2`  
* **Daily** → `1.20250301.0`, `1.20250301.1`, `1.20250301.2`  

## 9. Optional “No‑Longer‑Supported” Marker
* `1.999999.0` → declares MAJOR 1 as frozen / EOL  
* `2.2026.0` → next MAJOR continues normally  

## 10. Date‑Only‑Grows (DOG) Rule

Within a given **MAJOR** line, the **DATE** segment may **only stay the same length or grow**: `YYYY` → `YYYYMM` → `YYYYMMDD`.

> If you ever need to publish a version whose DATE would be *shorter*, **bump MAJOR first** and
> start over at the cadence you want. This guarantees monotonic numeric ordering and preserves
> full SemVer compatibility.

## 11. Scale Up

| From              | To               | Allowed? | Rationale                          |
|-------------------|------------------|----------|------------------------------------|
| `1.2025.2`        | `1.202503.0`     | ✅ Yes   | Yearly → Monthly (DATE grows)      |
| `1.202503.3`      | `1.20250301.0`   | ✅ Yes   | Monthly → Daily (DATE grows)       |
| `1.20250301.4`    | `1.2026.0`       | ❌ No    | Would shrink DATE inside MAJOR‑1   |
| `1.20250301.4`    | `2.2026.0`       | ✅ Yes   | MAJOR bump resets DATE to Yearly   |
| `2.2026.0`        | `2.202612.0`     | ✅ Yes   | Yearly → Monthly inside MAJOR‑2    |

With these rules you can *scale up* release granularity indefinitely while preserving clear history and complete **SemVer compatibility**.
