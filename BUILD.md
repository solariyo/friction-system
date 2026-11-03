# Updated `BUILD.md` — full

Save over `BUILD.md`.

```markdown
# Friction System — Build Tracker

Single source of truth for build progress, decisions, and open items.
Updated after every approved decision.

**Last updated:** 2026-11-03
**Current phase:** Batch C refactor (Node 20a → standalone)
**Progress:** 40/81 items done (~49%)
---

## 1. Phase Overview

| Phase | Status |
|---|---|
| 0 — Setup | Done |
| 1 — Storage | Done |
| 2 — Adzuna | Done |
| 3 — Sources (API-only) | Done |
| 4 — Graph skeleton | Done |
| 5 — Local models | Done |
| 6 — Fine-tune SLMs | Done |
| 7 — Frontier API | Done |
| 8 — Scoring + output | Done |
| 9 — Human review UI | Done |
| 10 — First full run | Done |
| 11A — Competitor + business plan | Done |
| 11B — Pipeline bug fixes | Done |
| 11C — Lead enrichment | Done |
| 11D — Gemini fallback | Done |
| 11E — Verbose + timing | Done |
| 11F — Batch A + B | Done |
| 11G — Batch C (documents) | In progress — refactor needed |
| 81-item plan (Phases 1–12) | In progress — 40/81 done |

---

## 2. Phase Details (0–10, historical)

### Phase 0 — Setup
Python 3.11 · folder structure · pyproject.toml · .gitignore · .env/.env.example · configs · README · install deps · Playwright Chromium · Modal auth · verification — **all Done**.

### Phase 1 — Storage
schema.sql · db.py · models.py · queries.py · test_storage.py (16 tests) — **all Done**.

### Phase 2 — Adzuna
signup + keys · base.py · adzuna.py · test_adzuna.py (13 tests) · live fetch — **all Done**.

### Phase 3 — Sources
Reed · Companies House · EPO · Contracts Finder · Hacker News · Stack Exchange · ONS (context only) — **all Done**. Dropped: Trustpilot/Capterra/G2/UKBF (Playwright), Jooble (weak UK), Reddit (suspended). Not started: Find a Tender, OpenAlex. Built but broken: NOMIS.

### Phase 4 — Graph skeleton
state.py · main_graph.py · 16 stub nodes · main.py · test_graph.py · HITL pause at Node 15 — **all Done**.

### Phase 5 — Local models
MiniLM · scikit-learn clustering · Nodes 1, 3, 5, 6, 7 real · test (13 tests) — **all Done**.

### Phase 6 — Fine-tune SLMs
qwen_app.py · Qwen client · Node 4 real · test (14 tests) · Modal deployed · live 57/60 — **all Done**.

### Phase 7 — Frontier API
Gemini/Claude/Groq clients · frontier router · Nodes 5, 9, 10, 11, 12 updated · test (16 tests) — **all Done**.

### Phase 8 — Scoring + output
Node 13 cost · Node 14 scoring · Node 16 output · shortlists table · clustering config · test (17 tests) — **all Done**.

### Phase 9 — Human review UI
Node 15 file pause · Streamlit review_app · decisions.json · resume.py CLI · live 2 runs — **all Done**.

### Phase 10 — First full run
Care home · 420 ads + 181 posts · 420/420 extracted · 9 frictions · Streamlit review + resume — **all Done**.

---

## 3. Phase 11 — Refine (Completed Sub-Phases)

### Phase 11A — Competitor + business plan (Done 2026-10-31)
Full section generation, Excel/Word output, Node 15.5/16.5, two-stage resume, live 3 plans.

### Phase 11B — Pipeline bug fixes (Done 2026-11-02)
`queries.py` dict+source inference · Node 4 Reed keys + retry · `main.py` init_db · Node 16.6 extracted ads · source count fix · `median_salary` in gate · review_app salaries + defaults.

### Phase 11C — Lead enrichment (Done 2026-11-02)
`sources/searxng.py` · `sources/search_router.py` · `sources/company_dedupe.py` · `sources/enrichment_cache.py` · `sources/tpos_scraper.py` · `sources/facebook_lookup.py` · `sources/yell_scraper.py` · `models/gemini.py` (3.5-flash-lite) · `models/groq.py` (Gemini fallback) · `company_enrichment` table · Node 16.6 full rewrite.

### Phase 11D — Gemini fallback (Done 2026-11-02)
Groq 429 → auto Gemini · `_get_gemini()` lazy init · verified across sections.

### Phase 11E — Verbose + timing (Done 2026-11-03)
`main.py` `--verbose` · `graph/state.py` verbose/timings/api_counts · `graph/main_graph.py` `_wrap_with_timing` · `utils/api_counter.py` · Node 4 Qwen counter · verbose summary prints runtime + per-node timings + API counts.

### Phase 11F — Batch A + B (Done 2026-11-03)
- **Batch A:** extraction cache (Node 4 = 0s on rerun) · `--verbose` · `--dry-run` · `raw_ads.json` + `raw_posts.json` per run · CH filing history · reverse LinkedIn lookup · `.env.example` updated
- **Batch B:** `queries.py` upsert_company/insert_contact/link_company_friction/link_contact_friction/get_friction_id_by_line · Node 4.5 extracts employers → companies table · Node 19 labels contacts per friction · Node 16.6 persists to companies+contacts

### Phase 11G — Batch C: documents (IN PROGRESS 2026-11-03)

**Design:** hybrid auto-search + HITL review + manual upload. Per-query industry body mapping. User chooses ads/docs/both at Node 0.5.

**Config:**
- PDF folder: `data/docs/<run_id>/`
- Retention: keep permanently (gitignored)
- Dedupe: skip if URL already in `documents` table
- Abort: if user unchecks all docs, continue with ads only

**Sub-batches:**

**C1 — Infrastructure (Done 2026-11-03)**
- `documents` table (schema)
- `sources/documents_searcher.py`
- `sources/industry_bodies.py` (estate agency → RICS/Propertymark/ARLA; care → CQC/Care England/King's Fund; etc.)
- Node 0.5 mode gate
- `state.py` new fields
- `main.py` `--mode` flag
- SearXNG empty-cache fix

**C2 — PDF fetch + extract (Done 2026-11-03)**
- `sources/pdf_downloader.py` (50MB cap, dedupe)
- `sources/pdf_extractor.py` (pypdf, 30k cap)
- Node 20a — doc searcher (writes `pending_docs.json`)
- Node 20b — doc fetcher (downloads approved, extracts, Qwen hints)
- Live test: 20MB TPO Annual Review downloaded + 30k chars extracted

**C3 — Integration (Done, refactor pending)**
- Node 5 accepts `friction_docs`
- Node 20a/20b wired into graph
- Streamlit Doc Review tab
- `resume.py` Stage 0
- **Known bug:** Node 20a pauses in-graph, needs refactor to standalone script

---

## 4. The 81-Item Tracker

**Legend:** ✅ done · 🟡 partial · ⬜ pending · ❌ N/A

### Phase 1 — Fix broken pipeline

| # | Item | Status |
|---|---|---|
| 1 | `queries.py` — model_dump fix | ✅ |
| 2 | `node_04` — Reed keys + retry | ✅ |
| 3 | `main.py` — init_db | ✅ |
| 4 | `node_16_6` — extracted ads | ✅ |
| 5 | `node_01` — count fix | ✅ |
| 6 | `node_03` — count fix | ✅ |
| 7 | Sanity check job ads | ✅ |
| 8 | `--verbose` flag | ✅ |

### Phase 1.5 — SearXNG with Serper fallback

| # | Item | Status |
|---|---|---|
| 9 | Verify Docker | ✅ |
| 10 | Start SearXNG container | ✅ |
| 11 | Test SearXNG JSON | ✅ |
| 12 | Build `searxng.py` | ✅ |
| 13 | Build `search_router.py` | ✅ |
| 14 | Node 11 uses SearchRouter | ✅ |
| 15 | Node 16.6 uses SearchRouter | ✅ |
| 16 | Result filter | ✅ |
| 17 | HITL when SearXNG down | ✅ |
| 18 | Update `.env.example` | ✅ |

### Phase 2 — Verify base

| # | Item | Status |
|---|---|---|
| 19 | Clear cache, rerun | ✅ |
| 20 | 420/420 extracted, DB = 420 | ✅ |
| 21 | Medians distinct, smoke passes | ✅ |
| 22 | Save raw ads per run | ✅ |

### Phase 2.5 — Documents as discovery source

| # | Item | Status | Notes |
|---|---|---|---|
| 23 | `documents` table | ✅ | C1 |
| 24 | `scripts/ingest_docs.py` | 🟡 | Replaced by Node 20a/b |
| 25 | Node 20 — extract friction text | ✅ | Node 20b |
| 26 | Wire Node 20 into graph | ✅ | Needs refactor |
| 27 | Link docs to companies | ⬜ | Not yet |
| 28 | Streamlit upload tab | ✅ | Doc Review tab |
| 29 | Support annual/NHS/council docs | ✅ | Via industry_bodies + templates |

### Phase 3 — Contact database

| # | Item | Status |
|---|---|---|
| 30 | `companies` table | ✅ |
| 31 | `contacts` table | ✅ |
| 32 | `company_friction_links` table | ✅ |
| 33 | `contact_friction_roles` table | ✅ |
| 34 | Add `company_id` to `job_ads` | ✅ |

### Phase 4 — Enrichment

| # | Item | Status |
|---|---|---|
| 35 | Node 4.5 — employers → `companies` | ✅ |
| 36 | Node 18 — enricher | ✅ |
| 37 | CH officers → directors | ✅ |
| 38 | Website deep-crawl | ✅ |
| 39 | LinkedIn via SearXNG | ✅ |
| 40 | Email pattern detection | ❌ |
| 41 | Filing-history freshness | ✅ |
| 42 | Reverse lookup + LinkedIn | ✅ |
| 43 | Email verification | ⬜ |

### Phase 5 — Labeling

| # | Item | Status |
|---|---|---|
| 44 | Node 19 — label per friction | ✅ |
| 45 | Evidence-anchored labels | ⬜ |
| 46 | Contact scoring/ranking | 🟡 |
| 47 | Label basis + confidence | ⬜ |
| 48 | Person vs role vs email | ⬜ |
| 49 | Branch vs company | ⬜ |

### Phase 6 — Lead export

| # | Item | Status |
|---|---|---|
| 50 | Node 16.6 writes to DB | ✅ |
| 51 | `leads/` folder per friction | ✅ |
| 52 | CSV + MD output | ✅ |

### Phase 7 — Friction quality

| # | Item | Status |
|---|---|---|
| 53 | Confidence flag | ⬜ |
| 54 | Gaps report | ⬜ |
| 55 | Sample quotes | ⬜ |
| 56 | Friction history table | ⬜ |
| 57 | Fix `[Name 1]` fallback | ⬜ |

### Phase 8 — Visa output

| # | Item | Status |
|---|---|---|
| 58 | 19-section template | ⬜ |
| 59 | "Evidence of demand" Section 5 | ⬜ |
| 60 | SearXNG redirect fix | ❌ |
| 61 | Call script + tracker | ⬜ |

### Phase 9 — Multi-run

| # | Item | Status |
|---|---|---|
| 62 | Comparison view | ⬜ |
| 63 | Cross-friction warmest | ⬜ |
| 64 | Buildability score | ⬜ |
| 65 | Friction scoring v2 | ⬜ |

### Phase 10 — Infrastructure

| # | Item | Status |
|---|---|---|
| 66 | Update BUILD.md | 🟡 |
| 67 | Update README | 🟡 |
| 68 | Cost tracking | ✅ |
| 69 | `--dry-run` | ✅ |
| 70 | Prompt regression tests | ⬜ |

### Phase 11 — Cleanup

| # | Item | Status |
|---|---|---|
| 71 | Delete `gemini.py` | ⬜ |
| 72 | Delete `.bak` files | 🟡 |
| 73 | Delete unused sources | ⬜ |

### Phase 12 — Deferred

| # | Item | Status |
|---|---|---|
| 74 | Tavily | ⬜ |
| 75 | Real-time monitoring | ⬜ |
| 76 | Hunter.io | ⬜ |
| 77 | Pitch deck | ⬜ |
| 78 | Slide deck | ⬜ |
| 79 | OCR | ⬜ |
| 80 | Semantic search | ⬜ |
| 81 | Doc topic clustering | ⬜ |

**Progress:** ✅ 40 · 🟡 5 · ❌ 2 · ⬜ 34

---

## 5. Batch Method — Reduce Modal Costs

**Rule:** never run after every item. Batch fixes, then run once.

### Batch A — Fixes + cost reduction ✅ COMPLETE (2026-11-03)
Items 4.5, 8, 18, 22, 68, 69, 41, 42. Dry-run cost £0.00.

### Batch B — Phase 3 + Phase 5 implementation ✅ COMPLETE (2026-11-03)
Items 35, 44, 50. Node 4.5 + Node 19 built. DB: 118 companies, 64 contacts, 90 links, 64 labels.

### Batch C — Documents as discovery source (IN PROGRESS)

**Design:** hybrid auto-search + HITL review + manual upload. Per-query industry body mapping. User chooses ads/docs/both at Node 0.5.

**Config:**
- PDF folder: `data/docs/<run_id>/`
- Retention: keep permanently (gitignored)
- Dedupe: skip if URL already in `documents` table
- Abort: if user unchecks all docs, continue with ads only

**C1 — Infrastructure (Done)**
- `documents` table
- `sources/documents_searcher.py`
- `sources/industry_bodies.py` (keyword → UK industry body sites)
- Node 0.5 mode gate
- `state.py` new fields
- `main.py` `--mode` flag
- SearXNG empty-cache fix

**C2 — PDF fetch + extract (Done)**
- `sources/pdf_downloader.py` (50MB cap, dedupe)
- `sources/pdf_extractor.py` (pypdf)
- Node 20a — doc searcher (writes `pending_docs.json`)
- Node 20b — doc fetcher (downloads approved, extracts, Qwen hints)

**C3 — Integration (Done, refactor pending)**
- Node 5 accepts `friction_docs`
- Node 20a/20b wired into graph
- Streamlit Doc Review tab
- `resume.py` Stage 0
- **Refactor needed:** Node 20a → standalone `scripts/search_docs.py` (currently pauses in-graph)

### Batch D — Friction quality + visa output

| # | Item |
|---|---|
| 53 | Confidence flag |
| 54 | Gaps report |
| 55 | Sample quotes per friction |
| 56 | Friction history table |
| 57 | Fix `[Name 1]` fallback |
| 58 | 19-section template enforcement |
| 59 | "Evidence of demand" Section 5 |
| 61 | Call script + tracker |

**Known bug fix in D:** Node 19 inserts `frictions` rows without dedup. Fix: Node 16 persists frictions first, or Node 19 skips if `friction_line + run_id` exists. Add `run_id` column to `frictions` table.

### Batch E — Multi-run + infrastructure

| # | Item |
|---|---|
| 62 | Comparison view |
| 63 | Cross-friction warmest companies |
| 64 | Buildability score |
| 65 | Friction scoring v2 |
| 67 | Update README |
| 70 | Prompt regression tests |

---

## 6. Decisions Log

| Date | Decision |
|---|---|
| 2026-09-14 | Friction-first design. No industries upfront. |
| 2026-09-14 | Node 7 counts unique employers, 500+ threshold. |
| 2026-09-14 | Derived pricing from friction cost. |
| 2026-09-14 | £3M ARR target. |
| 2026-09-14 | MLflow tracking. |
| 2026-09-14 | Append-only ledger. |
| 2026-09-14 | 16 nodes in main graph. |
| 2026-09-14 | Qwen 7B/3B + Claude Sonnet 5 + MiniLM. |
| 2026-09-14 | Modal GPU. |
| 2026-09-15 | Jooble skipped. |
| 2026-09-15 | Reddit skipped. |
| 2026-09-16 | ONS context only. Companies House for SIC. |
| 2026-09-16 | Verification rule. |
| 2026-09-16 | BUILD.md created. |
| 2026-10-21 | EPO industry-agnostic. |
| 2026-10-21 | sic_mapper DIVISIONS_BY_SECTION. |
| 2026-10-21 | .env.example created. |
| 2026-10-21 | tenders table added. |
| 2026-10-22 | Contracts Finder built (POST+JSON). |
| 2026-10-22 | Hacker News built (Algolia). |
| 2026-10-22 | Stack Exchange built (5 sites). |
| 2026-10-26 | NOMIS built, endpoint broken. Deferred. |
| 2026-10-27 | Phase 4 complete. 125 tests. |
| 2026-10-28 | Phase 5 complete. 138 tests. |
| 2026-10-28 | Phase 6 complete. 57/60 extracted. |
| 2026-10-29 | Phase 7 complete. 14 frictions. |
| 2026-10-30 | Phase 8 complete. Cost formula B. |
| 2026-10-30 | Phase 9 complete. File HITL. |
| 2026-10-30 | Phase 10 complete. First run 9 frictions. |
| 2026-10-31 | Business plan: Word + Excel + Markdown. |
| 2026-10-31 | Business plan: service vs goods template. |
| 2026-10-31 | HITL at Node 15.5. |
| 2026-10-31 | Node 11 real Serper search. |
| 2026-10-31 | Lead list v1 + v2. |
| 2026-10-31 | Phase 11 Day 2 complete. 208 tests. |
| 2026-11-02 | SearXNG replaces Serper (free local). |
| 2026-11-02 | TPOS email/phone source (legal, free). |
| 2026-11-02 | No email guessing. Real only. |
| 2026-11-02 | Gemini 3.5-flash-lite auto-fallback. |
| 2026-11-02 | Enrichment cache (30-day TTL). |
| 2026-11-02 | Documents as Phase 2.5 foundational. |
| 2026-11-02 | Cloudflare-blocked: accepted loss, TPOS fills. |
| 2026-11-03 | Verbose flag works (15 nodes timed, 903 API calls). |
| 2026-11-03 | Batch method locked: one run per batch. |
| 2026-11-03 | Extraction cache: Node 4 = 0s on rerun, dry-run = £0.00. |
| 2026-11-03 | Node 4.5 extracts employers → companies table (118 companies). |
| 2026-11-03 | Node 19 labels contacts per friction (64 labels from CH directors). |
| 2026-11-03 | Documents = hybrid auto-search + HITL review + manual upload. |
| 2026-11-03 | User chooses ads/docs/both at Node 0.5 gate before any run. |
| 2026-11-03 | PDFs: `data/docs/<run_id>/`, keep permanently, dedupe by URL. |
| 2026-11-03 | Industry body mapping per query keyword (estate agency → RICS/Propertymark/ARLA; care → CQC/Care England/King's Fund; etc.) |
| 2026-11-03 | **SearXNG empty-cache fix** — don't cache empty results (prevents poisoning when upstream engines rate-limit) |
| 2026-11-03 | **SearXNG rate-limited** from testing — DuckDuckGo CAPTCHA, Google CSE 403, Brave 429 — cooling off before live test |
| 2026-11-03 | **Node 20a refactor queued:** move out of graph, standalone `scripts/search_docs.py` — cleaner than in-graph pause |

---

## 7. Sources

| Source | Type | Cost | Status | Notes |
|---|---|---|---|---|
| Adzuna | API | Free (1,000/mo) | Built | Job ads |
| Reed | API | Free | Built | Job ads |
| Companies House | API | Free (600/5min) | Built | SIC + officers + filings |
| Contracts Finder | API | Free | Built | UK tenders |
| EPO OPS | API | Free | Built | Patents |
| Hacker News | API | Free | Built | Algolia |
| Stack Exchange | API | Free | Built | 5 sites |
| ONS | API | Free | Partial | Metadata |
| **TPOS** | HTTP | Free | Built | Property Ombudsman |
| **SearXNG** | HTTP | Free | Built | Local Docker |
| Serper | API | Free (2,500/mo) | Fallback | Only if SearXNG down |
| Crawl4AI | Local | Free | Built | Scrape |
| curl_cffi | Local | Free | Built | CF bypass |
| Yell | Scrape | Free | Partial | CF-blocked |
| **Documents** | HTTP | Free | **Built (C1–C3)** | Annual reports, trade bodies, consulting, regulators, think tanks, industry bodies |
| NOMIS | API | Free | Deferred | Broken |
| Jooble | API | Free | Skipped | Weak UK |
| Reddit | API | Free | Skipped | Suspended |
| Find a Tender | API | Free | Not started | — |
| OpenAlex | API | Free | Not started | Academic |

Dropped (Playwright): Trustpilot, Capterra, G2, UKBF, Google Patents, UK IPO.

---

## 8. Models

| Model | Where used | Notes |
|---|---|---|
| Qwen 2.5 7B Instruct | Nodes 4, 6, 11, 20b extraction | Modal GPU |
| Qwen 2.5 3B Instruct | Node 8 classification | Not built |
| Groq `openai/gpt-oss-120b` | Nodes 5, 9, 10, 11, 12, plans | Free, active |
| Gemini `gemini-3.5-flash-lite` | Fallback on Groq 429 | Free, 6 req/min |
| Claude Sonnet 5 | Optional upgrade | Paid |
| all-MiniLM-L6-v2 | Nodes 5, 6 embeddings | Local CPU |
| scikit-learn | Node 5 clustering | Local CPU |

---

## 9. Verification Log

| Item | Claim | Source | Verified? |
|---|---|---|---|
| SIC 49410 | Freight transport by road | ONS SIC 2026 | 2026-09-16 |
| EPO OPS | 8 industries | Live query | 2026-10-21 |
| Contracts Finder | POST JSON tenders | Live query | 2026-10-22 |
| Hacker News | Algolia hits | Live query | 2026-10-22 |
| Stack Exchange | v2.3, 5 sites | Live query | 2026-10-22 |
| Graph | 16 nodes execute | pytest | 2026-10-27 |
| Phase 6 | 57/60 extracted | "payroll" | 2026-10-28 |
| Phase 7 | 14 frictions | "payroll" | 2026-10-29 |
| Phase 8 | 4 distinct frictions | "payroll" | 2026-10-30 |
| Phase 9 | Pause + review + resume | 2 runs | 2026-10-30 |
| Phase 10 | 420 → 9 frictions | "care home" | 2026-10-30 |
| Phase 11A | 3 full plans | "care home" | 2026-10-31 |
| Phase 11B | 420/420 extracted | "estate agency" | 2026-11-02 |
| Phase 11C | 36 leads, 27 emails, 23 phones, 36 directors | "estate agency" | 2026-11-02 |
| TPOS Hamptons | towerbridge@hamptons-int.com, 02074073173 | TPOS | 2026-11-02 |
| TPOS Barnard Marcus | Tottenhamhighroad@barnardmarcus.co.uk, 02033691246 | TPOS | 2026-11-02 |
| TPOS White & Sons | oxted@whiteandsons.co.uk, 01883712315 | TPOS | 2026-11-02 |
| Gemini fallback | Groq 429 → Gemini OK | Live run | 2026-11-02 |
| SearXNG | 0 Serper calls across 3 frictions | Live run | 2026-11-02 |
| Verbose flag | 15 nodes timed, 903 API calls | "estate agency" | 2026-11-03 |
| Extraction cache | Node 4 = 0s on rerun, cost £0.00 | "estate agency" | 2026-11-03 |
| Node 4.5 | 118 employers → companies table | "estate agency" | 2026-11-03 |
| Node 19 | 90 links, 64 labels from CH directors | Test script | 2026-11-03 |
| CH filing | 10 filings, active: True (Northern Estate) | CH API | 2026-11-03 |
| **PDF download** | **20MB TPO Annual Review downloaded** | **Live test** | **2026-11-03** |
| **PDF extract** | **30,000 chars extracted** | **Live test** | **2026-11-03** |
| **Doc search** | **39 PDF candidates across 8 types (estate agency)** | **Live test** | **2026-11-03** |
| **Industry bodies** | **estate agency → RICS/Propertymark/ARLA; care → CQC/King's Fund** | **Live test** | **2026-11-03** |

---

## 10. Open Questions

- Which ONS datasets to approve?
- Which SIC codes to target as seeds?
- Document auto-search: how to fix SearXNG rate limit — add Mojeek/Startpage? Or use Serper for docs?
- Company dedupe: how aggressive?
- Contact labeling: rule-based vs LLM-assisted?
- Branch vs company labels?

---

## 11. What's Locked

- Friction-first, no industries upfront
- Node 7 counts unique employers, 500+ threshold
- Derived pricing from friction cost
- £3M ARR target
- 16 nodes in main graph
- Qwen + Claude/Groq + Gemini + MiniLM models
- Modal for GPU
- MLflow tracking
- Append-only ledger
- No auto-learning
- No paid tools without approval
- Verification rule: every number traces to source
- SearXNG primary, Serper fallback with HITL
- No email guessing — real only
- TPOS for UK estate agency contacts
- Enrichment cache persists across runs
- Documents as foundational source (Phase 2.5)
- Batch method: one run per batch, 5 total
- Documents: hybrid auto-search + HITL review + manual upload
- User chooses ads/docs/both at Node 0.5 before any run
- PDFs: `data/docs/<run_id>/`, keep permanently, dedupe by URL

---

## 12. Current State

**Latest commits:**
```
c44e0ac Batch C2: PDF downloader + extractor + Node 20a searcher + Node 20b fetcher. Live test: 20MB TPO Annual Review downloaded and text extracted.
1b517fa Batch C1: documents table + documents_searcher + Node 0.5 mode gate + industry_bodies map + extra_sites prompt + cache empty fix. SearXNG rate-limited (all engines suspended) - cooling off before C3 test.
0d37459 Batch B: companies/contacts/company_friction_links/contact_friction_roles populated. Node 4.5 extracts employers from ads. Node 19 labels contacts by friction. Node 16.6 persists to companies+contacts.
```

**Progress:** 40/81 items done.

**Next:** C3 refactor (Node 20a → standalone) then live doc test when SearXNG recovers.

**DB state (last check):**
- companies: 118
- contacts: 64
- company_friction_links: 90
- contact_friction_roles: 64
- frictions: 3
- documents: 0 (pending live test)

---

## 13. Quick Reference

**Commands:**
```powershell
python -m friction_system.main "estate agency" -v
python -m friction_system.main "estate agency" -v --dry-run
python -m friction_system.main "estate agency" -v --mode docs
python -m friction_system.main "estate agency" -v --mode both
python -m friction_system.resume <run_id>
streamlit run src/friction_system/ui/review_app.py
python scripts/smoke_test.py runs/<run_id>
python scripts/check_contacts.py
```

**Environment:**
- Conda env: `friction`
- Python: 3.11.16
- Docker: SearXNG on `localhost:8080`
- DB: `storage/friction.db`
- Runs: `runs/<run_id>/`
- Docs: `data/docs/<run_id>/`

**Key files:**
- `nodes/node_16_6_lead_list.py` — lead builder
- `nodes/node_04_extract_and_tag.py` — Qwen + cache
- `nodes/node_04_5_extract_companies.py` — employers → companies
- `nodes/node_19_label_contacts.py` — contact labeling
- `nodes/node_20a_doc_searcher.py` — doc search (needs refactor)
- `nodes/node_20b_doc_fetcher.py` — doc download + extract
- `nodes/node_00_5_mode_gate.py` — mode selection
- `sources/tpos_scraper.py` — TPOS
- `sources/searxng.py` — SearXNG client
- `sources/search_router.py` — SearXNG → Serper fallback
- `sources/documents_searcher.py` — PDF candidates
- `sources/industry_bodies.py` — keyword → body sites
- `sources/pdf_downloader.py` — download + dedupe
- `sources/pdf_extractor.py` — pypdf
- `sources/enrichment_cache.py` — persistent cache
- `sources/companies_house.py` — CH + officers + filings
- `sources/linkedin_reverse.py` — reverse LinkedIn
- `models/groq.py` — Groq + Gemini fallback
- `models/gemini.py` — `gemini-3.5-flash-lite`
- `graph/main_graph.py` — verbose timing wrapper
- `utils/api_counter.py` — API counter helper

**Environment variables (.env):**
- `ADZUNA_APP_ID`, `ADZUNA_APP_KEY`
- `REED_API_KEY`
- `COMPANIES_HOUSE_API_KEY`
- `EPO_CONSUMER_KEY`, `EPO_CONSUMER_SECRET`
- `GROQ_API_KEY` (primary)
- `GEMINI_API_KEY` (fallback)
- `SERPER_API_KEY` (fallback search only)
- `MODAL_QWEN_URL`
- `ONS_API_KEY` (optional)
- `ANTHROPIC_API_KEY` (optional)

---

## 14. New Thread Handoff

Provide: this BUILD.md + `git log --oneline -3` + message:

> "Continue with C3 refactor: Node 20a → standalone `scripts/search_docs.py`. 40/81 items done. Batches A + B complete. Batch C1–C3 infrastructure built. PDF download + extract works (20MB TPO test). Doc search works but SearXNG rate-limited — cooling off. Known issue: Node 20a pauses in-graph, needs refactor. Read BUILD.md, confirm status, refactor Node 20a."
```
