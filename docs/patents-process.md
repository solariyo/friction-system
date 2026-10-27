# Patents Process

How the system sources, stores, and uses patent data.

**Last updated:** 2026-09-17
**Related:** `BUILD.md` section 6, `README.md` section 15

---

## 1. Why patents matter

Patents serve two purposes in the system:

**1. Node 10 — patent landscape**
- Search existing patents on the friction
- Identify white space (what's not patented)
- Draft a claim outline for the patent attorney
- Flags only — never drops a friction

**2. Business plan evidence**
- Proves innovation — "no prior patents on this method"
- Or warns — "5 patents already cover this"
- Endorsers respect patent diligence

---

## 2. Sources evaluated

| Source | Status | Reason |
|---|---|---|
| **EPO OPS** | ✅ Primary | Official API, covers UK + global, free tier |
| **Google Patents** | ✅ Secondary | Aggregator, covers global, XHR JSON endpoint |
| **UK IPO** | ❌ Skipped | Cloudflare blocks scraping |
| **USPTO** | ❌ Skipped | Requires US address for API key |
| **WIPO PATENTSCOPE** | ❌ Skipped | JSF app, session-based URLs, not automatable |
| **Google Scholar** | ⚠️ Optional | Non-patent literature, deferred |

---

## 3. EPO OPS (primary)

**What it is:** European Patent Office's Open Patent Services.

**URL:** https://developers.epo.org/

**Coverage:** EU + global, including UK patents.

**Auth:** Consumer key + secret. Free tier: 4 GB/month.

**Verification:**
- Test patent: EP3873788 (Flexible combined transport of people and freight)
- Found in both UK IPO search and Espacenet (EPO's public search)
- Confirmed EPO covers UK-related patents

**Status:** Registered. Awaiting key from EPO (validation takes 1–2 business days).

**When built:**
- `src/friction_system/sources/epo.py`
- Interface: `search(query, limit)`, `get(patent_id)`
- Cache 30 days
- Used by Node 10

---

## 4. Google Patents (secondary)

**What it is:** Google's patent search aggregator.

**URL:** https://patents.google.com/

**Coverage:** Global — PCT, UK, US, EU, JP, CN, and others.

**Auth:** None.

**Access method:** XHR endpoint.

**Endpoint:**
```
GET https://patents.google.com/xhr/query
  ?url=q=(freight+transport)&page=0
  &exp=
```

**Response:** JSON with `results.cluster[].result[]`.

**Per-patent fields:**
- `id` — like `patent/CN121209484A/en`
- `patent.title`
- `patent.snippet`
- `rank`

**Pagination:** `page` parameter. 10 results per page.

**Status:** Working. Rate-limited by Google on rapid requests (503). Retry with backoff handles it.

**Verified:** Searched "freight transport" — returned 122,931 results. Pagination confirmed across pages 0, 1, 2.

**When built:**
- `src/friction_system/sources/google_patents.py` (built)
- Interface: `search(query, pages)`, `count_results(query)`
- Cache 30 days
- Used by Node 10

---

## 5. UK IPO (skipped)

**What it is:** UK Intellectual Property Office patent search.

**URL:** https://www.search-for-intellectual-property.service.gov.uk/

**Why skipped:**

- Behind Cloudflare
- httpx returns 403
- Playwright headless gets Cloudflare challenge ("Just a moment...")
- CSV export endpoint returns 403
- Verified with markers: `cloudflare`, `cf-`, `challenges.cloudflare.com`, `__cf_chl` all True in HTML

**Verdict:** Not scrapable reliably. Same class as Indeed, LinkedIn, Google search.

**Fallback:** Manual search in a browser when a specific UK-only patent needs checking.

---

## 6. USPTO (skipped)

**What it is:** US Patent and Trademark Office.

**Why skipped:** PatentsView API requires a US address and state during signup. UK-based applicant cannot complete honestly.

**Fallback:** Google Patents indexes US patents.

---

## 7. WIPO PATENTSCOPE (skipped)

**What it is:** WIPO's international patent search.

**URL:** https://patentscope.wipo.int/search/en/search.jsf

**Why skipped:**

- Built on JavaServer Faces (JSF)
- Search URLs contain session IDs (`_vid=...`)
- URLs expire and aren't reusable
- Hostile to automation
- EPO already indexes PCT applications

**Fallback:** Google Patents covers PCT.

---

## 8. Google Scholar (optional, deferred)

**What it is:** Academic paper search.

**Why deferred:**

- Non-patent literature matters for prior art
- But it's a secondary concern
- Can be added if an endorser asks

**Not built.** Add later if needed.

---

## 9. What the system does with patents

**Node 10 (patent landscape) will:**

1. Take a friction
2. Query EPO OPS and Google Patents with friction keywords
3. Count and classify results (direct hit / adjacent / unrelated)
4. Identify white space
5. Draft a rough claim outline
6. Write to `frictions.patent_status`, `patent_brief`, `prior_art`

**Flags only.** Never drops a friction.

---

## 10. Output format

For each friction:

| Field | Source |
|---|---|
| `patent_status` | clear / crowded / blocked |
| `patent_brief` | rough claim outline |
| `prior_art` | list of relevant patents |

**Uses:**
- Node 14 scoring (small weight)
- Business plan narrative
- Attorney consultation

---

## 11. Verification log

| Item | Claim | Source | Verified? |
|---|---|---|---|
| EPO covers UK patents | EP3873788 found in both UK IPO and Espacenet | EPO + UK IPO | ✅ 2026-09-17 |
| Google Patents XHR works | 200, JSON, 122,931 results for "freight transport" | Live test | ✅ 2026-09-17 |
| UK IPO blocked | Cloudflare markers confirmed | HTML analysis | ✅ 2026-09-17 |

---

## 12. Decisions log

| Date | Decision |
|---|---|
| 2026-09-17 | EPO OPS as primary patent source |
| 2026-09-17 | Google Patents as secondary |
| 2026-09-17 | Skip UK IPO (Cloudflare) |
| 2026-09-17 | Skip USPTO (US address requirement) |
| 2026-09-17 | Skip WIPO (JSF, session-based) |
| 2026-09-17 | Defer Google Scholar (optional) |

---

## 13. Links

- EPO OPS portal: https://developers.epo.org/
- EPO Espacenet: https://worldwide.espacenet.com/
- Google Patents: https://patents.google.com/
- UK IPO search: https://www.gov.uk/search-for-patent
- UK IPO IPSUM: https://www.ipo.gov.uk/p-ipsum.htm
- WIPO PATENTSCOPE: https://patentscope.wipo.int/
- Google Scholar: https://scholar.google.com/

---

## 14. Gotchas

- **Google Patents throttles by IP.** Add retries with backoff. Wait 5–15 minutes if blocked.
- **EPO keys take 1–2 days to validate** after registration.
- **Cloudflare-protected sites should be avoided entirely.** Not worth the fight.
- **Patent URLs matter.** Google Patents uses `patent/{id}/en` format. EPO uses different IDs.
- **A patent in one system may lag in another.** Recent filings appear faster in their filing office.
- **Non-patent literature is a separate domain.** Google Scholar is not a patent source.