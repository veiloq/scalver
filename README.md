# **ScalVer Specification v1.2025.4**

## **1\. Purpose & Essence**

ScalVer is a **calendar‑aware, SemVer‑compatible and extendable versioning scheme** expressed as

```
<MAJOR>.<DATE>.<PATCH>
```

where the `DATE` segment may lengthen over time **within a MAJOR line**: `YYYY` → `YYYYMM` → `YYYYMMDD`.

* `<MAJOR>` – identical to SemVer MAJOR, bumped for breaking changes **or** whenever the DATE segment would need to shrink

* `<DATE>` – `YYYY`, `YYYYMM`, or `YYYYMMDD` in **UTC**; extends (never shrinks) as release cadence accelerates

* `<PATCH>` – incremental, backward‑compatible fixes within the same DATE window

### **Minimal Grammar (EBNF)**

```
MAJOR = digit *digit  ; 0 = volatile alpha, 1+ = stable API
DATE    = YYYY | YYYYMM | YYYYMMDD      ; UTC calendar date
PATCH   = non‑negative digit *digit
PRE     = ( "-" identifier *( "." identifier ) ) ; optional
BUILD   = ( "+" identifier *( "." identifier ) ) ; optional
version = MAJOR "." DATE "." PATCH [ PRE ] [ BUILD ]
```

*identifier* follows [SemVer 2.0 rules](https://semver.org/).

---

## **2\. Why ScalVer?**

\~ It’s a simple adaptation of **CalVer** that remains fully compatible with **SemVer**, but lets you switch release frequencies without messing up version ordering.

* **Temporal context** → every tag reveals its release window at a glance

* **Break‑age transparency** → MAJOR still flags incompatible API shifts

* **Single trajectory** → teams glide from yearly to monthly to daily without inventing new numbering schemes

---

## **3\. Segments & Core Rules**

| Segment | SemVer Equivalent | ScalVer Behaviour |
| ----- | ----- | ----- |
| **MAJOR** | MAJOR | Bump for **breaking changes** *or* when DATE would otherwise shrink |
| **DATE** | MINOR | May stay the same length or grow from `YYYY` → `YYYYMM` → `YYYYMMDD` |
| **PATCH** | PATCH | Increment for safe, backward‑compatible releases within the same DATE |

**SemVer 2.0 Compatible:** Each ScalVer tag is a *syntactically valid* SemVer 2.0 version; the date sits in the position that vanilla SemVer calls “MINOR”, so standard parsers (`golang.org/x/mod/semver`, `npm‑semver`, Python `packaging.version`, Maven’s `ComparableVersion`, Cargo, etc.) order ScalVer releases correctly without modification¹

**Date‑Only‑Grows (DOG):** within any single MAJOR line the DATE can stay the same length or grow, but **never shrink**.

¹**Y10K note:** ScalVer’s ordering logic continues to work for years beyond 9999. The reference grammar intentionally limits YYYY to four digits for ISO‑8601 clarity and broad tooling support, but the scheme is formally 100% compatible with longer year fields and can be widened whenever ecosystems catch up.

---

## **4\. Release‑Cadence Examples**

| Cadence | Example tag | Meaning |
| ----- | ----- | ----- |
| Yearly | `0.2025.0` | First 2025 alpha |
| Yearly | `1.2025.0` | First 2025 stable release |
| Yearly | `1.2025.3` | Fourth 2025 patch |
| Monthly | `1.202503.0` | First March 2025 release |
| Monthly | `1.202503.2` | Third March 2025 release |
| Daily | `1.20250301.0` | First 1 Mar 2025 release |
| Daily | `1.20250301.7` | Eighth 1 Mar 2025 release |

Progression path: `YYYY` → `YYYYMM` → `YYYYMMDD`.

---

MAJOR = 0 → Alpha Track (volatile)
MAJOR ≥ 1 → Stable Track (guarantees apply; breakage = new major)

---

## **5\. Transitions Examples**

### **Table A – Allowed transitions *within the same MAJOR***

| From | To | Δ DATE | Allowed? | Rationale |
| ----- | ----- | ----- | ----- | ----- |
| `1.2025.2` | `1.202503.0` | \+MM | ✓ Yes | Yearly → Monthly – DATE grows (`YYYY` → `YYYYMM`) |
| `1.202503.3` | `1.20250301.0` | \+DD | ✓ Yes | Monthly → Daily – DATE grows (`YYYYMM` → `YYYYMMDD`) |
| `1.20250301.4` | `1.20250301.5` | \= | ✓ Yes | Daily → Daily – PATCH \+1, DATE unchanged |
| `1.202503.0` | `1.202503.1` | \= | ✓ Yes | Monthly → Monthly – PATCH \+1 |
| `1.2025.0` | `1.2025.1` | \= | ✓ Yes | Yearly → Yearly – PATCH \+1 |

### **Table B – Transitions that shrink DATE or require a MAJOR bump**

| From | To | Δ DATE | Allowed without bump? | Correct fix / note |
| ----- | ----- | ----- | ----- | ----- |
| `1.20250301.4` | `1.2026.0` | –DD/MM | ✗ No | Bump MAJOR → `2.2026.0` |
| `1.20250301.4` | `2.2026.0` | reset | ✓ Yes | MAJOR bump resets cadence |
| `2.20261224.6` | `2.202612.7` | –DD | ✗ No | Keep daily cadence or bump MAJOR |

*Date-Only-Grows: DATE cannot shrink inside the same MAJOR; a MAJOR bump resets the cadence.*

---

## **6\. Extended Semantics**

*Pre‑release identifiers* follow SemVer precedence:

* `1.202503.0-alpha.1` \< `1.202503.0-beta.1` \< `1.202503.0-rc1` \< `1.202503.0`

*Build metadata* (ignored in precedence):

* `1.202503.0+linux.amd64`

* `1.202503.0+sha.42ab1ef`

---

## **7\. End‑of‑Life (EOL) Markers**

* `1.999999.0` – freezes the *entire* 1.x line (year upper‑bound \= 9999; 999 999 acts as sentinel)

* `2.2026.0` – resumes normal dating under MAJOR 2

---

## **8\. Migration Strategies**

* **Greenfield project** – start with `0.YYYY.0` internally; switch to `1.<current‑year>.0` at first public release

* **Existing SemVer project** – translate `X.Y.Z` → `X.<current‑year>.Z`; keep MAJOR intact

* **Accelerating cadence** – extend DATE length (DOG‑compliant) without changing MAJOR

* **Tooling** – have CI derive DATE in **UTC** and fail builds that regress the DATE segment

---

## **9\. Common Pitfalls & Remedies**

* Shrinking DATE inside a MAJOR line → **forbidden**; bump MAJOR first

* Dropping back from daily to monthly cadence → bump MAJOR to reset DATE granularity

* Using local timezone in CI → inject DATE via `date -u` or similar UTC source

* Patching a *previous* date after cadence acceleration → use a maintenance branch, tag on the frozen DATE (e.g. `1.202401.5`)

---

## **10\. FAQ**

* **Can I use ScalVer with Cargo / Maven / npm / Go / Python?** – Yes; all treat `<DATE>` as MINOR, so caret (`^`) and tilde (`~`) ranges still work unchanged.

* **What if we tag `1.202503.0` for March 2025 and later jump to `1.20250225.0`, which looks like the year 202 502 25 A.D.?**  
   *Date cannot be shrunk within a MAJOR line; nonetheless:*  
   1\. **Y10K perspective** – ScalVer comparisons remain correct with years \> 9999; there’s no intrinsic cap. For interoperability, the reference grammar sticks to four‑digit `YYYY`. Teams needing post‑9999 dating may **(a)** extend the `YYYY` field (i.e, `YYYYYYYY`), or **(b)** bump MAJOR and restart at `YYYY = 0000`.  
   2\. **ISO‑8601 perspective** – By default we enforce four‑digit years for maximum tooling compatibility. Under this rule `20250225` unambiguously parses to **2025‑02‑25**. A later yearly tag (e.g. the future “**`20250225`**” year example) can’t shrink `DATE` within MAJOR 1
