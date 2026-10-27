# ONS Process

How the system sources, approves, and uses ONS datasets.

**Last updated:** 2026-09-17
**Related:** `BUILD.md` section 6, `README.md` section 15

---

## 1. Why ONS matters

ONS (Office for National Statistics) is used to **prove scalability and viability** for the Innovator Founder application.

Every shortlisted friction needs evidence that the market behind it is:

- **Viable** — the sector produces real economic value
- **Scalable** — the sector is big enough and growing

ONS is a government source. Endorsers accept it without question. It gives:

- Sector employment size
- Employment growth trends
- Wage trends (labour cost)
- Economic output (GVA)
- Firm births and deaths (business demography)

This is not a filter. It does not decide which frictions survive. It is **evidence** for the business plan and the visa application.

---

## 2. What ONS is NOT

- **Not a firm count source.** Firm counts come from Companies House (SIC-level, live).
- **Not a filter.** ONS does not remove or rank frictions.
- **Not live data.** ONS updates monthly or annually. Not daily.
- **Not SIC-precise.** ONS often uses broad industry groups, not 5-digit SIC codes.

---

## 3. Relationship to Companies House

| Question | Source |
|---|---|
| How many firms in SIC 49410? | Companies House → 76,192 |
| How many people work in transport? | ONS labour market |
| Is the sector growing? | ONS business demography |
| What's the economic output? | ONS GVA by industry |
| Are wages rising? | ONS ASHE / earnings |

Both are evidence of scalability. Different units. Different sources. Both go in the business plan.

---

## 4. Dataset types to hunt

| Type | What it proves | Search terms |
|---|---|---|
| Labour market | Employment size, growth | `labour market` |
| Business demography | Firm births, deaths, survival | `business demography` |
| GVA by industry | Economic output, value | `gva industry` |
| Employment by industry | Sector workforce | `employment industry` |
| Regional employment | Geographic spread | `regional employment` |
| Earnings / wages | Labour cost trends | `earnings` `ashe` |
| Productivity | Output per worker | `productivity` |

---

## 5. Priority order

**Must-have (for visa evidence):**

1. **Labour market** — employment size, growth ✅ approved
2. **Business demography** — firm births/deaths, survival
3. **GVA by industry** — economic value
4. **Earnings / ASHE** — wage trends

**Nice-to-have:**

5. Regional employment
6. Productivity
7. Turnover / output

**Minimum viable set:** labour market + business demography + GVA + earnings.

---

## 6. What each proves

**Labour market:**
- How many people work in the sector
- Is employment growing or shrinking
- → Scalability

**Business demography:**
- Are firms being created or dying?
- Do firms survive long enough to buy software?
- → Scalability

**GVA by industry:**
- Sector produces real economic value
- Not a dying industry
- → Viability

**Earnings / ASHE:**
- Wages rising = hard to hire = friction is real
- The role you're automating is getting expensive
- → Friction justification

**Regional employment:**
- Sector is UK-wide (not just London)
- Bigger addressable market
- → Scalability

**Productivity:**
- Low productivity = manual work = opportunity
- → Friction justification

---

## 7. What to skip

- **Census datasets (`RM*`)** — old (2021), partial UK (England & Wales only), static snapshots
- **Broad business register** (`uk-business-by-enterprises-and-local-units`) — Companies House covers firm counts better
- **Non-business datasets** — deaths, wellbeing, housing, crime, education
- **Anything pre-2020**

Rule: if it doesn't prove scalability or viability for a business sector, skip it.

---

## 8. How to search the catalogue

In the ONS review UI:

```powershell
streamlit run src\friction_system\ui\ons_review.py
```

Search terms:

- `labour market`
- `business demography`
- `gva`
- `earnings` or `ashe`
- `regional employment`
- `productivity`

For each candidate:

1. Read the title and description
2. Check the unit (people? firms? £?)
3. Check coverage (UK-wide? England only?)
4. Check date (recent? current?)
5. Approve if it proves scalability or viability

---

## 9. Approval flow

1. Open the review UI
2. Search for a dataset
3. Expand to read details
4. Approve / Reject / Defer
5. Add a note explaining why
6. Decisions save to `data/ons/approved_datasets.json`

Approved datasets are queried by `ons.py`. Rejected ones are logged and skipped.

---

## 10. Verification

Before using a dataset in the business plan:

1. Confirm it's approved in `approved_datasets.json`
2. Confirm the unit (people, firms, £)
3. Confirm the coverage (UK, England, region)
4. Confirm the date (recent)
5. Log verification in `BUILD.md` section 6

**Known checks:**

| Dataset | What it proves | Verified? |
|---|---|---|
| `labour-market` | Employment size, growth | Pending |
| `RM063` | Census — unapproved | Removed |
| `uk-business-by-enterprises-and-local-units` | Broad groups only — skipped | — |

---

## 11. Decisions log

| Date | Decision |
|---|---|
| 2026-09-16 | ONS demoted to context only (broad groups, not SIC-level) |
| 2026-09-17 | Reframed: ONS proves scalability and viability, not context |
| 2026-09-17 | `labour-market` approved |
| 2026-09-17 | `RM063` unapproved (Census, England & Wales only) |
| 2026-09-17 | Broad business register skipped (Companies House covers it) |
| 2026-09-17 | Priority set: labour market + business demography + GVA + earnings |

---

## 12. Gotchas

- **ONS uses broad groups and SOC codes.** Not SIC. Direct mapping to a friction's SIC code isn't always possible.
- **Census datasets are snapshots.** Old, partial UK, not trends.
- **Data updates monthly or annually.** Not daily. Cache responses.
- **Coverage varies.** Some datasets are England only, some UK-wide. Check.
- **Different units across datasets.** People, firms, £, hours. Don't mix.

---

## 13. Re-downloading / refreshing

ONS datasets update on their own schedule. When you need fresh data:

1. Open the review UI
2. Find the dataset
3. Click "Refresh catalogue from ONS" (sidebar)
4. Re-approve if the ID changed
5. Old cache expires after 30 days automatically

---

## 14. How the code uses ONS

`ons.py` (to be built) will:

1. Read `data/ons/approved_datasets.json`
2. Query each approved dataset via the ONS Beta API
3. Cache responses (30-day expiry)
4. Expose helpers:
   - `get_employment_by_sector(sector)`
   - `get_growth_trend(sector)`
   - `get_gva(sector)`
5. Used by Nodes 12 (scalability block), 14 (score), 16 (business plan output)

Update to reflect: ONS API = metadata; values = manual CSV
CSV sizes (GVA 268 MB, ASHE 3.9 GB)
GVA and ASHE for business plan only. Manual CSV parse at plan stage. Not in the pipeline.