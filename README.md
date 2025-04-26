# **Scalable Calendar Versioning Specification v1.2025.6**

TLDR: `1.2025.5 < 1.20250323.0 < 2.2025.0 < 2.202503.1 < 2.20250125.1`

## **1\. Purpose & Core Concept**

ScalVer is a **calendar‑aware, SemVer‑compatible and extendable versioning scheme** expressed as

```
<MAJOR>.<DATE>.<PATCH>
```

where the `DATE` segment may lengthen over time **within a MAJOR line**: `YYYY` → `YYYYMM` → `YYYYMMDD` (`YYYY[MM[DD]]`)

* **`<MAJOR>`** — mirrors SemVer’s MAJOR component; incremented for *breaking-change* releases **or** whenever the `DATE` segment would otherwise need to contract.

* **`<DATE>`** — expressed as `YYYY`, `YYYYMM`, or `YYYYMMDD` in **UTC**; it may *stay the same width* or *expand* (year → month → day) as release cadence accelerates, and it resets to its initial width when the next MAJOR version begins.

* **`<PATCH>`** — mirrors SemVer’s PATCH component; a monotonically increasing counter for backward-compatible updates released within the same `DATE` window.

---

## **2\. Motivation**

 ScalVer provides the time-based clarity of CalVer (knowing when something was released) while needing the compatibility guarantees and tooling support of SemVer (knowing if an update breaks things).

* **Adjustable cadence** : ScalVer allows projects to adjust their release frequency (yearly, monthly, daily) and reflect this in the versioning without breaking the logical version order. 

* **SemVer Compatibility**: every ScalVer tag is syntactically valid SemVer, so existing tooling (CI/CD, package managers, release dashboards) works unchanged.

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

### **3\.1 Difference with SemVer Backus–Naur Form Grammar**

```diff
--- SemVer2.0.bnf
+++ SemVer2.0.bnf
@@
-<version core> ::= <major> "." <minor> "." <patch>
+<version core>  ::= <major> "." <date> "." <patch>
@@
-<minor> ::= <numeric identifier>
+
+<date>  ::= <year>
+          | <year> <month>
+          | <year> <month> <day>
+
+<year>  ::= <positive digit> <digit> <digit> <digit>
+<month> ::= "01" | "02" | "03" | "04" | "05" | "06" \
+          | "07" | "08" | "09" | "10" | "11" | "12"
+<day>   ::= "01" | "02" | "03" | "04" | "05" | "06" | "07" | "08" | "09" \
+          | "10" | "11" | "12" | "13" | "14" | "15" | "16" | "17" | "18" | "19" \
+          | "20" | "21" | "22" | "23" | "24" | "25" | "26" | "27" | "28" | "29" \
+          | "30" | "31"
```

### **3.2 Full ScalVer BNF Grammar**

The complete ScalVer grammar is available in the [BNF](./BNF) file for reference and implementation.

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

> Progression path: `YYYY` → `YYYYMM` → `YYYYMMDD`.

---

- MAJOR = 0 ⇒ Alpha, Experiment, PoC (volatile)
- MAJOR ≥ 1 ⇒ Stable Release (guarantees apply; breakage = new major)

---

## **5\. Transitions Examples**

### **Table A – Allowed transitions *within the same MAJOR***

| From | To | Δ DATE | Allowed? | Rationale |
| ----- | ----- | ----- | ----- | ----- |
| `1.2025.2` | `1.202503.0` | \+MM | ✓ Yes | Yearly → Monthly – DATE grows (`YYYY` → `YYYYMM`) |
| `1.202507.3` | `1.20250701.0` | \+DD | ✓ Yes | Monthly → Daily – DATE grows (`YYYYMM` → `YYYYMMDD`) |
| `1.20250301.4` | `1.20250301.5` | \= | ✓ Yes | Daily → Daily – PATCH \+1, DATE unchanged |
| `1.202510.0` | `1.202510.1` | \= | ✓ Yes | Monthly → Monthly – PATCH \+1 |
| `1.2025.0` | `1.2025.1` | \= | ✓ Yes | Yearly → Yearly – PATCH \+1 |

### **Table B – Transitions that shrink DATE or require a MAJOR bump**

| From | To | Δ DATE | Allowed without bump? | Correct fix / note |
| ----- | ----- | ----- | ----- | ----- |
| `1.20250301.4` | `1.2026.0` | –DD/MM | ✗ No | Bump MAJOR → `2.2026.0` |
| `1.20250301.4` | `2.2026.0` | reset | ✓ Yes | MAJOR bump resets cadence |
| `2.20271225.6` | `2.202712.7` | –DD | ✗ No | Keep daily cadence or bump MAJOR |

*Date-Only-Grows: DATE cannot shrink inside the same MAJOR; a MAJOR bump resets the cadence.*

---

## **6\. Extended Semantics**

*Pre‑release identifiers* follow SemVer precedence:

* `1.202503.0-alpha.1` \< `1.202503.0-beta.1` \< `1.202503.0-rc1` \< `1.202503.0`

*Build metadata* (ignored in precedence):

* `1.202503.0+linux.amd64`

* `1.202503.0+sha.42ab1ef`

---

## **7\. Migration**

Because every ScalVer tag is syntactically valid SemVer, most projects can keep their existing tooling unchanged or with only minimal tweaks.

---

### 7\.1  SemVer → ScalVer Example

| Format                         | New Variant      | Length¹ | Δ vs `1.23.5` | Conversion                                             |
|--------------------------------|------------------|---------|--------------|---------------------------------------------------------|
| `xMAJOR.YYYY.xPATCH`           | `1.2025.5`       | 8       | +2           | Major & Patch preserved; **Year replaces Minor**        |
| `xMAJOR.YYYYMM.xPATCH`         | `1.202504.5`     | 10      | +4           | Major & Patch preserved; **Year‑Month replaces Minor**  |
| `xMAJOR.YYYYMMDD.xPATCH`       | `1.20250421.5`   | 12      | +6           | Major & Patch preserved; **Y‑M‑D replaces Minor**       |
| `xMAJOR.YYYY.xMINORxPATCH`     | `1.2025.235`     | 10      | +4           | Major preserved; **Minor & Patch concatenated**         |
| `xMINOR.YYYY.PATCH`            | `23.2025.0`      | 9       | +3           | **Minor promoted to leading segment; Major dropped**    |
| `xMINOR.YYYY.PATCH`            | `23.2025.5`      | 9       | +3           | **Minor promoted to leading segment; Major dropped; Patch preserved**    |

¹ **Log & storage overhead example** (**assuming (1) one‑byte UTF‑8 characters and (2) no compression/deduplication**): a +6 variant could inflates log lines by 6 MB per million tags and consumes 6 MB more disk per million stored records.


### 7\.2 Playbook

**7.2.1 Quick path (most projects):**

1. **Choose calendar width** — `YYYY`, `YYYYMM`, or `YYYYMMDD`.  
2. **Reset PATCH** to `0`.  
3. **Keep MAJOR** unless you also break the API.  
4. **Publish** `MAJOR.DATE.0`.

**7.2.2 Guard against legacy *minor* overflows:**

1. `maxMinor = max(X in MAJOR.X.PATCH)`  
2. Compute today’s `DATE` (`YYYYMMDD` or `YYYYMM` or `YYYY`).  
3. Compare  
   * **DATE > maxMinor** → tag `MAJOR.DATE.0`.  
   * **DATE ≤ maxMinor** → **Increment MAJOR** and tag `newMAJOR.DATE.0`.

> **Note** Option B (incrementing MAJOR) may be unacceptable in ecosystems where MAJOR is tightly coupled to compatibility promises or installer heuristics. Prefer widening the DATE slot whenever possible; choose a MAJOR bump only when all stakeholders agree it won’t disrupt dependency resolution policies.

---

## **8\. FAQ**

* **Can I use ScalVer with Cargo / Maven / npm / Go / Python?** – Yes; all treat `<DATE>` as MINOR, so caret (`^`) and tilde (`~`) ranges still work unchanged.

* **What if we tag `1.202503.0` for March 2025 and later jump to `1.20250225.0`, which looks like the year 202 502 25 A.D.?**  
   *Within a MAJOR line, the DATE segment may grow but must never shrink; you can shorten it only after bumping to a new MAJOR version (see 5); nonetheless:*  
   1\. **Y10K perspective** – ScalVer comparisons remain correct with years \> 9999; there’s no intrinsic cap. For interoperability, the reference grammar sticks to four‑digit `YYYY`. Teams needing post‑9999 dating may **(a)** extend the `YYYY` field (i.e, `YYYYYYYY`), or **(b)** bump MAJOR and restart at `YYYY = 0000`.  
   2\. **ISO‑8601 perspective** – By default we enforce four‑digit years for maximum tooling compatibility. Under this rule `20250225` unambiguously parses to **2025‑02‑25**. A later yearly tag (e.g. the future “**`20250225`**” year example) can’t shrink `DATE` within MAJOR 1

* **Is ScalVer an extension of SemVer?** – No. ScalVer tags are a syntactic subset of SemVer, yet they intentionally diverge semantically by repurposing the MINOR field as a calendar date.
