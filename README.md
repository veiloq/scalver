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
MAJOR   = non‑zero digit *digit
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

**Date‑Only‑Grows (DOG):** within any single MAJOR line the DATE can stay the same length or grow, but **never shrink**.¹

¹ **Y10K note** – ScalVer comparisons remain valid with years \> 9999\. The reference grammar caps `YYYY` at four digits for ISO‑8601 readability and maximum tool compatibility, **but ScalVer is formally 100 % compatible** if you decide to extend the field or restart at `YYYY = 0000` after a MAJOR bump.

---

## **4\. Release‑Cadence Examples**

| Cadence | Example | Meaning |
| ----- | ----- | ----- |
| Yearly | `1.2025.0` | First 2025 stable release |
| Yearly | `1.2025.3` | Fourth 2025 **patch** release |
| Monthly | `1.202503.0` | First March 2025 release |
| Monthly | `1.202503.2` | Third March 2025 release |
| Daily | `1.20250301.0` | First 1 Mar 2025 release |
| Daily | `1.20250301.7` | Eighth 1 Mar 2025 release |

Progression path: `YYYY` → `YYYYMM` → `YYYYMMDD`.

---

## **5\. Extended Semantics**

*Pre‑release identifiers* follow SemVer precedence:

* `1.202503.0-alpha.1` \< `1.202503.0-beta.1` \< `1.202503.0-rc1` \< `1.202503.0`

*Build metadata* (ignored in precedence):

* `1.202503.0+linux.amd64`

* `1.202503.0+sha.42ab1ef`

---

## **6\. End‑of‑Life (EOL) Markers**

* `1.999999.0` – freezes the *entire* 1.x line (year upper‑bound \= 9999; 999 999 acts as sentinel)

* `2.2026.0` – resumes normal dating under MAJOR 2

---

## **7\. Migration Strategies**

* **Greenfield project** – start with `0.YYYY.0` internally; switch to `1.<current‑year>.0` at first public release

* **Existing SemVer project** – translate `X.Y.Z` → `X.<current‑year>.Z`; keep MAJOR intact

* **Accelerating cadence** – extend DATE length (DOG‑compliant) without changing MAJOR

* **Tooling** – have CI derive DATE in **UTC** and fail builds that regress the DATE segment

---

## **8\. Automation Snippet (Git Hook)**

```
# Generate the next ScalVer tag for the current branch
next_scalver() {
    today=$(date -u +%Y%m%d)
    IFS='.' read -r major date patch <<< "$1"

    if [[ "$date" == "$today"* ]]; then
        echo "${major}.${date}.$((patch + 1))"
    else
        echo "${major}.${today}.0"
    fi
}

# Guard against shrinking DATE without MAJOR bump
git_tag_guard() {
    latest=$(git describe --tags --abbrev=0)
    next=$(next_scalver "$latest")
    IFS='.' read -r _ prevDate _ <<< "$latest"
    IFS='.' read -r _ newDate  _ <<< "$next"
    [[ ${#newDate} -lt ${#prevDate} ]] && {
        echo "DATE shrink detected; bump MAJOR" >&2
        exit 1
    }
    echo "$next"
}
```

*Hook ignores pre‑release/build identifiers; append them afterwards if needed.*

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
   1\. **Y10K perspective** – ScalVer comparisons remain correct with years \> 9999; there’s no intrinsic cap. For interoperability, the reference grammar sticks to four‑digit `YYYY`. Teams needing post‑9999 dating may **(a)** extend the `YYYY` field, or **(b)** bump MAJOR and restart at `YYYY = 0000`.  
   2\. **ISO‑8601 perspective** – By default we enforce four‑digit years for maximum tooling compatibility. Under this rule `20250225` unambiguously parses to **2025‑02‑25**. A later yearly tag (e.g. the future “**`20250225`**” year example) can’t shrink `DATE` within MAJOR 1
