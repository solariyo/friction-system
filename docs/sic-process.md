# SIC Code Process

How the system sources, stores, and uses UK Standard Industrial Classification (SIC) codes.

**Last updated:** 2026-09-16
**Related:** `BUILD.md` section 6, `README.md` section 15

---

## 1. Why SIC codes matter

Every friction in the system eventually needs a SIC code. The code lets us:

- Count active UK firms in that industry (Companies House API)
- Pull market context from ONS
- Score scalability (market size, ceiling, growth)
- Write a defensible business plan with traceable numbers

Wrong SIC code → wrong firm count → wrong scalability → wrong business plan.

So SIC codes must be sourced from the official list. Never guessed.

---

## 2. Source

**Official source:** ONS UK SIC 2026

**Page:** https://www.ons.gov.uk/methodology/classificationsandstandards/ukstandardindustrialclassificationofeconomicactivities/uksic2026

**Published:** 3 August 2026

**Licence:** Open Government Licence v3.0

We use **SIC 2026**, not SIC 2007. The 2007 version was the previous standard; 2026 is current.

---

## 3. Files

Two Excel files. Both downloaded from ONS.

| File | Purpose | Size | Rows |
|---|---|---|---|
| `uksic2026summaryofstructure03aug2026.xlsx` | Full hierarchy with descriptions | 112 KB | ~1,475 |
| `uksic2026index03aug2026.xlsx` | Alphabetical keyword index | 538 KB | ~17,862 |

**Local location:** `data/ons/sic/`

**Download URLs:**

- Summary: https://www.ons.gov.uk/file?uri=/methodology/classificationsandstandards/ukstandardindustrialclassificationofeconomicactivities/uksic2026/uksic2026summaryofstructure03aug2026.xlsx
- Index: https://www.ons.gov.uk/file?uri=/methodology/classificationsandstandards/ukstandardindustrialclassificationofeconomicactivities/uksic2026/uksic2026index03aug2026.xlsx

---

## 4. File structure

### 4.1 Summary of Structure

**Sheet:** `reworked structure`

**Columns:**

| Column | Meaning |
|---|---|
| Description | Human-readable name |
| Section | 1-letter code (A–U) |
| Division | 2-digit code |
| Group | 3-digit code |
| Class | 4-digit code |
| Subclass | 5-digit code |
| Most disaggregated level | The deepest code for this row |
| Level headings | Metadata (used for lookups) |

**Example rows:**

| Description | Section | Division | Group | Class | Subclass | Most disaggregated |
|---|---|---|---|---|---|---|
| AGRICULTURE, FORESTRY AND FISHING | A | na | na | na | na | A |
| Crop and animal production, hunting and related service activities | A | 01 | na | na | na | 01 |
| Growing of non-perennial crops | A | 01 | 011 | na | na | 011 |
| Growing of cereals, other than rice, leguminous crops and oil seeds | A | 01 | 011 | 0111 | na | 0111 |
| Growing of cereals (full subclass) | A | 01 | 011 | 0111 | 01110 | 01110 |

- `na` = that level doesn't apply to the row
- The deepest populated level is the code for that row

**Levels:**

| Level | Digits | Example |
|---|---|---|
| Section | 1 letter | A |
| Division | 2 digits | 01 |
| Group | 3 digits | 011 |
| Class | 4 digits | 0111 |
| Subclass | 5 digits | 01110 |

### 4.2 Index

**Sheet:** `SIC 2026 Index`

**Columns:**

| Column | Meaning |
|---|---|
| UK SIC 2026 | Current code (blank for legacy rows) |
| UK SIC 2007 | Legacy code (blank for current rows) |
| Activity | Description or keyword |

**Example rows:**

| UK SIC 2026 | UK SIC 2007 | Activity |
|---|---|---|
| 01110 | | Growing of cereals, other than rice, leguminous crops and oil seeds |
| | 01110 | Barley growing |
| 01110 | | Wheat growing |

- Multiple rows per code (keyword synonyms)
- Column 3 is the searchable text

---

## 5. Process we took

1. **Identified the need** — frictions need SIC codes for firm counts and market size
2. **Found the source** — ONS UK SIC 2026 page (current standard, not 2007)
3. **Noted the download links** — Summary and Index xlsx
4. **Created the local folder** — `data/ons/sic/`
5. **Downloaded both files** — kept original ONS filenames for traceability
6. **Peeked at the structure** — using `openpyxl` to read sheet names, columns, sample rows
7. **Documented the structure** — in this file and in `BUILD.md` section 6

---

## 6. How the code uses these files

`sic_mapper.py` will:

1. Parse the Summary file → build a `code → {description, level, parent}` lookup
2. Parse the Index file → build a `keyword → [codes]` search
3. Save both to `data/ons/sic_codes.json`
4. Expose:
   - `lookup(code)` — returns description and level
   - `search(keyword)` — returns matching codes
   - `list_all()` — returns every code
   - `parent(code)` — returns the parent code, if any

All levels (Section, Division, Group, Class, Subclass) are kept.

---

## 7. Verification

Before using a SIC code for a friction:

1. Look it up in `sic_mapper.py`
2. Confirm the description matches the friction
3. Cross-check with Companies House (count firms for the code)
4. If the count is implausible, verify the code is correct

**Known check:**

| Code | Expected description | Verified? |
|---|---|---|
| 49410 | Freight transport by road | Pending |

Verification status logged in `BUILD.md` section 6.

---

## 8. Re-downloading (when ONS updates)

ONS publishes SIC updates periodically. To refresh:

1. Go to https://www.ons.gov.uk/methodology/classificationsandstandards/ukstandardindustrialclassificationofeconomicactivities/uksic2026
2. Find the latest "Summary of structure" and "Indexes" xlsx
3. Download both to `data/ons/sic/`
4. Keep the ONS filename (includes date) for traceability
5. Update this file with the new filenames and dates
6. Update `BUILD.md` section 6
7. Re-run `sic_mapper.py` to regenerate `sic_codes.json`

---

## 9. Gotchas

- **SIC 2026 ≠ SIC 2007.** Use 2026. Check the year on the file.
- **5-digit Subclass is what Companies House uses.** Not 4-digit Class.
- **`na` means "not applicable at this level."** Not an error.
- **The Index has duplicate codes.** Multiple rows per code (synonyms). Deduplicate when building the keyword map.
- **Some rows in the Index are 2007-only.** Legacy entries. Filter by `UK SIC 2026` column being non-blank.
- **File names change each release.** Keep them, don't rename. Traceability.
- **LibreOffice vs openpyxl.** We use openpyxl. LibreOffice was considered but rejected (only reads first sheet).

---

## 10. Decision log

| Date | Decision |
|---|---|
| 2026-09-16 | Use SIC 2026, not SIC 2007 |
| 2026-09-16 | Use Summary + Index files, not the integral notes version |
| 2026-09-16 | Keep original ONS filenames for traceability |
| 2026-09-16 | Use openpyxl for parsing |
| 2026-09-16 | Keep all SIC levels (Section through Subclass) |
| 2026-09-16 | `sic_mapper.py` will output `data/ons/sic_codes.json` |