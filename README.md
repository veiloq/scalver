# Scalable Calendar Versioning (ScalVer) v1.2025.3

**TL;DR — Start yearly, grow to monthly, grow to daily while staying SemVer‑safe** `MAJOR.YYYY[MM[DD]].PATCH`

Ordering stays numeric

* 1.2025.0 < 1.2025.1 < 1.2025.2
* 1.202503.0 < 1.202503.1 < 1.202503.2
* 1.2025.0 < 1.202503.0 < 1.20250301.0
* 1.2025.0 < 1.2026.1 < 2.2026.0
* 1.20250410.0 < 2.2026.1 < v3.20270310.0
* v1.2025.0 < v1.2025.1 < v1.2025.2 (with prefix, tag)

Format progression → `MAJOR.YYYY.PATCH` → `MAJOR.YYYYMM.PATCH` → `MAJOR.YYYYMMDD.PATCH`

## 1. Problem Statement

Projects often accelerate from annual releases to monthly sprints and daily hot‑fixes.  
[CalVer](https://calver.org/) hides breaking changes, [SemVer](https://semver.org/) hides when a build was shipped.

ScalVer fuses both:

* **Date context** — anyone sees *when* at a glance.  
* **API context** — MAJOR still signals breaking changes.  
* **Elasticity** — one scheme from day 1 through high‑tempo CI/CD.

## 2. SemVer Compatibility

| Segment | SemVer role | Rules & tips |
|---------|-------------|--------------|
| **MAJOR** | `MAJOR` | Increment for breaking changes. |
| **DATE**  | `MINOR` | Accepts `YYYY`, `YYYYMM`, or `YYYYMMDD` (UTC). Numeric comparison ensures 202503 > 2025. |
| **PATCH** | `PATCH` | Increment for every additional stable build within the current DATE. |

Note: If an old tag would make the next DATE smaller, simply bump MAJOR.

## 3. Yearly Format

* `1.2025.0` → first stable release in 2025  
* `1.2025.1` → subsequent release in 2025  

Best for projects with only a few releases each year.

## 4. Monthly Format

* `1.202503.0` → first March 2025 release  
* `1.202503.1` → second March 2025 release  

When cadence shifts to monthly, 202503 already sorts after 2025, so nothing breaks.

## 5. Daily Format

* `1.20250301.0` → first build on 1 Mar 2025  
* `1.20250301.1` → follow‑up build on the same day  

Choose daily precision when multiple releases per day are realistic.

## 6. Pre‑release Suffixes

* `1.202503.0-alpha.1` < `1.202503.0-beta.1` < `1.202503.0` 
* `1.202503.0-rc1` < `1.202503.0`

Pre‑release tags carry lower precedence than the final build, per SemVer.

## 7. Build Metadata

* `1.202503.0+linux.amd64`  
* `1.202503.0+build.42ab1ef`

Everything after + is ignored for precedence by Go, npm, Cargo, etc.

## 8. Multiple Releases in the Same Period

* **Yearly** → `1.2025.0`, `1.2025.1`, `1.2025.2` 
* **Monthly** → `1.202503.0`, `1.202503.1`, `1.202503.2` 
* **Daily** → `1.20250301.0`, `1.20250301.1`, `1.20250301.2` 

Just increment PATCH.

## 9. Optional “No‑Longer‑Supported” Marker

* `v1.999999.0` → declares MAJOR 1 as frozen / EOL  
* `v2.2026.0` → next MAJOR continues normally  
