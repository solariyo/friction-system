# Friction System — Build Tracker

Single source of truth for build progress, decisions, and open items.
Updated after every approved decision.

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
| 11 — Refine | In Progress |

---

## 2. Phase Details

### Phase 0 — Setup

| Step | Status |
|---|---|
| 1 — Python 3.11 install | Done |
| 2 — Folder structure (src/ layout) | Done |
| 3 — pyproject.toml | Done |
| 4 — .gitignore | Done |
| 5 — .env / .env.example | Done |
| 6 — config/sources.yaml + settings.yaml | Done |
| 7 — README.md | Done |
| 8 — Install dependencies | Done |
| 9 — Playwright Chromium | Done |
| 10 — Modal auth (heartofchr) | Done |
| 11 — Verification | Done |

### Phase 1 — Storage

| Step | Status |
|---|---|
| 1 — schema.sql | Done |
| 2 — db.py | Done |
| 3 — models.py | Done |
| 4 — queries.py | Done |
| 5 — test_storage.py | Done (16 tests) |

### Phase 2 — Adzuna

| Step | Status |
|---|---|
| 1 — Adzuna signup + keys | Done |
| 2 — base.py | Done |
| 3 — adzuna.py | Done |
| 4 — test_adzuna.py | Done (13 tests) |
| 5 — Live fetch to SQLite | Done (50 ads) |

### Phase 3 — More scrapers

| Step | Status |
|---|---|
| 1 — Reed key + test | Done |
| 2 — reed.py | Done |
| 3 — test_reed.py | Done |
| 4 — Jooble | Skipped |
| 5 — Reddit | Skipped |
| 6 — ONS | Done (context only) |
| 7 — Companies House | Done |
| 8 — EPO | Done |
| 9 — Contracts Finder | Done |
| 10 — Hacker News | Done |
| 11 — Trustpilot | Dropped (Playwright) |
| 12 — Capterra | Dropped (Playwright) |
| 13 — G2 | Dropped (Playwright) |
| 14 — UKBF | Dropped (Playwright) |
| 15 — Find a Tender | Not started |
| 16 — NOMIS | Built. Tests pass. Live endpoint returns 406 with filter params — deferred to Phase 11. |
| 17 — Stack Exchange | Done |
| 18 — OpenAlex | Not started |

### Phase 4 — Graph skeleton

| Step | Status |
|---|---|
| 1 — state.py | Done |
| 2 — main_graph.py | Done |
| 3 — 16 stub nodes | Done |
| 4 — main.py entry point | Done |
| 5 — test_graph.py (4 tests) | Done |
| 6 — HITL pause at Node 15 | Done |

### Phase 11 — Refine and enhance

**Status:** In progress

#### Prerequisites

| Step | Status |
|---|---|
| 1 — Install Crawl4AI | Pending |
| 2 — Install python-docx + openpyxl | Pending |
| 3 — Copy Excel templates to `templates/` | Pending |
| 4 — Verify installs | Pending |

#### Phase 11A — Competitor search + business plan (highest value)

| Step | Status |
|---|---|
| 5 — Add generate_business_section() to groq.py | Done |
| 6 — Add to claude.py | Done |
| 7 — Add to gemini.py | Done |
| 8 — frontier.list_available_providers() with cost/status | Done |
| 9 — business_plan_generator.py (narrative) | Done |
| 10 — excel_generator.py (template fill) | Done |
| 11 — word_generator.py (Markdown → Word) | Done |
| 12 — Node 11 rewrite: Serper + LLM keyword extraction | Done |
| 13 — Node 15.5 HITL gate (model + type + scope) | Done |
| 14 — Node 16.5 orchestrator | Done |
| 15 — Graph wiring for Nodes 15.5, 16.5 | Done (two-stage resume) |
| 16 — resume.py update | Done |
| 17 — config/settings.yaml business_plan block | Done |
| 18 — tests/test_business_plan.py | Done |


#### Day 2 — Node 11 + Business Plan (complete)

| Component | Status |
|---|---|
| keyword_extraction.py (LLM + rule fallback) | Done |
| serper_filter.py (product filtering) | Done |
| Node 11 rewrite (real Serper search) | Done |
| business_plan_generator.py (10 sections) | Done |
| word_generator.py (md → docx) | Done |
| excel_generator.py (template fill, full + summary) | Done |
| Node 15.5 business plan gate | Done |
| Node 16.5 orchestrator (multi-plan) | Done |
| Two-stage resume (friction → business plan) | Done |
| Streamlit Business Plan tab | Done |
| Live test: 3 plans generated | Done |


#### Phase 11B — Lead list with contacts

| Step | Status |
|---|---|
| 19 — Node 15.6 HITL gate | Pending |
| 20 — Node 16.6 lead list orchestrator | Pending |
| 21 — sources/crawl4ai.py | Pending |
| 22 — sources/lead_extractor.py | Pending |
| 23 — Companies House search_by_name + officers | Pending |
| 24 — Lead list v1 (basic) | Pending |
| 25 — Lead list v2 (full contacts) | Pending |
| 26 — tests/test_lead_list.py | Pending |

#### Phase 11C — Quality additions

| Step | Status |
|---|---|
| 27 — Sample company quotes per friction | Pending |
| 28 — Confidence flag per friction | Pending |
| 29 — Friction history across runs | Pending |
| 30 — Founder Innovation -aligned template enforcement | Pending |
| 31 — Comparison view (multi-query) | Pending |
| 32 — Serper redirect URL resolution | Pending |

#### Phase 11D — Reliability

| Step | Status |
|---|---|
| 33 — Retry logic for Crawl4AI | Pending |
| 34 — Cache Crawl4AI results | Pending |
| 35 — Missing field handling | Pending |
| 36 — Email validation | Pending |
| 37 — Lead scoring | Pending |
| 38 — CRM export format | Pending |

#### Phase 11E — Optional polish

| Step | Status |
|---|---|
| 39 — Tavily second search source | Pending |
| 40 — Real-time job ad monitoring | Pending |
| 41 — Email enrichment (Hunter.io) | Pending |
| 42 — Auto-generate pitch deck | Pending |
| 43 — Friction scoring v2 (buildability) | Pending |
| 44 — Delete gemini.py or migrate to google.genai | Pending |

Not started. Details filled in when reached.

### Phase 5 — Local models

| Step | Status |
|---|---|
| 1 — MiniLM embedding module | Done |
| 2 — scikit-learn clustering module | Done |
| 3 — Node 1 real (Adzuna + Reed) | Done |
| 4 — Node 3 real (HN + Stack Exchange) | Done |
| 5 — Node 5 real (k-means) | Done |
| 6 — Node 6 real (dedupe) | Done |
| 7 — Node 7 (firm count, deferred to Phase 6) | Done |
| 8 — Test file (13 tests) | Done |


### Phase 6 — Fine-tune SLMs

| Step | Status |
|---|---|
| 1 — Modal app (qwen_app.py) | Done |
| 2 — Qwen client (qwen.py) | Done |
| 3 — Node 4 real extraction | Done |
| 4 — Test file (14 tests) | Done |
| 5 — Deployed to Modal | Done |
| 6 — Live test 57/60 extracted | Done |

### Phase 7 — Frontier API

| Step | Status |
|---|---|
| 1 — Gemini client | Done (fallback) |
| 2 — Claude client | Done (fallback) |
| 3 — Groq client | Done (active) |
| 4 — Frontier router | Done |
| 5 — Nodes 5, 9, 10, 11, 12 updated | Done |
| 6 — Test file (16 tests) | Done |
| 7 — Live run verified | Done |


### Phase 8 — Scoring + output

| Step | Status |
|---|---|
| 1 — Node 13 real cost | Done |
| 2 — Node 14 weighted scoring | Done |
| 3 — Node 16 output (JSON + MD) | Done |
| 4 — shortlists DB table | Done |
| 5 — Clustering config (k=10, dedupe=0.80) | Done |
| 6 — Test file (17 tests) | Done |

### Phase 9 — Human review UI

| Step | Status |
|---|---|
| 1 — Node 15 file-based pause | Done |
| 2 — Streamlit review_app.py | Done |
| 3 — decisions.json per run | Done |
| 4 — resume.py CLI | Done |
| 5 — Node 16 early exit if paused | Done |
| 6 — Live test: 2 runs paused, reviewed, resumed | Done |

### Phase 10 — First full run

| Step | Status |
|---|---|
| 1 — Real query "care home", 3 pages | Done |
| 2 — 420 job ads + 181 forum posts | Done |
| 3 — 420/420 extracted (0 failed) | Done |
| 4 — 10 clusters → 9 frictions | Done |
| 5 — Streamlit review + resume | Done |
| 6 — Shortlist output | Done |



### Phase 8 — Scoring + output

| Step | Status |
|---|---|
| 1 — Node 13 real cost | Done |
| 2 — Node 14 real scoring | Done |
| 3 — Node 16 output (JSON + MD) | Done |
| 4 — DB save to shortlists table | Done (after fix) |
| 5 — Clustering config (k=10, dedupe=0.8) | Done |
| 6 — Test file (17 tests) | Done |
| 7 — Live run: 4 distinct frictions | Done |

---

## 3. Decisions Log

| Date | Decision |
|---|---|
| 2026-09-14 | Friction-first design. No industries upfront. Industry is a label, not a filter. |
| 2026-09-14 | Node 7 counts unique employers from job ads. 500+ threshold. |
| 2026-09-14 | Derived pricing from friction cost, not fixed £1k. |
| 2026-09-14 | Target £3M ARR, not 1,000 customers. |
| 2026-09-14 | MLflow for run and weights tracking. |
| 2026-09-14 | Append-only ledger for frictions. Nothing deleted. |
| 2026-09-14 | 16 nodes total in main graph. |
| 2026-09-14 | Qwen 2.5 7B Instruct (extraction), Qwen 2.5 3B Instruct (classification), Claude Sonnet 5 (frontier). MiniLM for embeddings. |
| 2026-09-14 | Modal for serverless GPU. |
| 2026-09-15 | Jooble skipped (weak UK coverage). |
| 2026-09-15 | Reddit skipped (account suspended). Appeal sent. |
| 2026-09-16 | ONS demoted to context only (broad industry groups, not SIC-level). |
| 2026-09-16 | Companies House added back for SIC-level firm counts. |
| 2026-09-16 | Verification rule: every SIC, dataset, and number must trace to a verified source. |
| 2026-09-16 | BUILD.md created as master tracker. |
| 2026-10-21 | EPO confirmed industry-agnostic. Smoke test spans 8 industries. Node 10 will pass any friction keyword without sector assumptions. |
| 2026-10-21 | ONS query_ons() removed. ONS stays context-only. Catalogue + dataset metadata only, per 2026-09-16 decision. |
| 2026-10-21 | sic_mapper DIVISIONS_BY_SECTION defined. section_of() now works for all SIC codes. |
| 2026-10-21 | .env.example created. All active + placeholder keys documented. No secrets committed. |
| 2026-10-21 | tenders table added to schema. test_schema_creates_all_tables updated to expect 12 tables. |
| 2026-10-22 | Contracts Finder built. API uses POST + JSON body, not GET. Endpoint: /api/rest/2/search_notices/json. Response wraps each result in {"item": {...}}. No auth. Industry-agnostic — proven with 8 industries in smoke test. |
| 2026-10-22 | Hacker News built. Uses Algolia search endpoint (hn.algolia.com/api/v1/search). No auth. Industry-agnostic — smoke test spans 8 industries. |
| 2026-10-22 | Stack Exchange built. Searches 5 sites: workplace, freelancing, pm, serverfault, softwareengineering. Optional API key. Industry-agnostic. |
| 2026-10-26 | NOMIS built, tests pass. Live endpoint rejects filter params (406). Deferred. Companies House covers firm counts. |
| 2026-10-27 | Phase 4 complete. 16-node linear graph runs end-to-end. Node 15 pauses via terminal input(). All nodes are stubs — real logic in Phases 5–9. 125 tests pass. |
| 2026-10-28 | Phase 5 complete. MiniLM + k-means. Node 1 fetches live Adzuna + Reed. Node 3 fetches live HN + Stack Exchange. Node 5 clusters via k-means. Node 6 dedupes with cosine similarity. Node 7 pending SIC extraction. 138 tests pass. Live run: 60 ads + 29 posts → 17 clusters → 16 frictions. |
| 2026-10-28 | Phase 6 complete. Qwen 2.5 7B on Modal via HTTP endpoint. Node 4 extracts structured fields (role_title, tasks, salary, sic_code, is_manual, friction_hint). Live run: 57/60 extracted (95%). 3 transient timeouts, acceptable. Frictions: 13 from payroll query. |
| 2026-10-29 | Phase 7 complete. Frontier router supports Groq (free, active), Gemini (free, fallback), Claude (paid, upgrade). Active model: openai/gpt-oss-120b. Run produces 14 real frictions with IP, patent, competitor, and proof analysis. Groq reasoning model needs generous max_tokens (350-550) to allow 30-50 reasoning tokens. |
| 2026-10-30 | Phase 8 complete. Scoring from weights, cost from median salary £34k. Output: shortlist.json + shortlist.md + DB. Clustering tuned: k=10, dedupe=0.8. Friction repetition fixed (14 → 4). Top friction scored 4.05/10. |
| 2026-10-30 | Phase 8 complete. Cost formula B: labour_cost = median_salary × 0.20 × people_per_firm. Price 15–25% capture. Clustering tuned: k=10, dedupe=0.80. Output: shortlist.json + shortlist.md + DB. |
| 2026-10-30 | Phase 9 complete. File-based HITL. Node 15 pauses, Streamlit reviews, resume.py continues. |
| 2026-10-30 | Phase 10 complete. First real run "care home", 420 job ads, 420/420 extracted, 9 frictions, top score 4.91. Median care salary £31,200. |
| 2026-10-30 | Gemini SDK deprecated warning noted. Deferred to Phase 11. Likely action: delete gemini.py and rely on Groq + Claude only. |
| 2026-10-31 | Business plan output format: Word (.docx) + Excel (.xlsx) + Markdown. Markdown for source, Word for investors/Innovator Founder submission, Excel for financials. |
| 2026-10-31 | Business plan template: ask user at generation time (service vs goods) with idea summary shown. Both full and summary Excel outputs. |
| 2026-10-31 | Business plan model selection: HITL at Node 15.5. User chooses provider per section. Groq (free) default, Claude (paid) available if key present. |
| 2026-10-31 | Node 11 rewrite: LLM keyword extraction → Serper search → LLM judgment. Real competitors required for Innovator Founder (4–5 minimum). |
| 2026-10-31 | Lead list v1 + v2: per approved friction, per company. Uses Companies House + Serper + Crawl4AI. Full contacts. |
| 2026-10-31 | Phase 11 Day 2 complete. Node 11 rewritten with real Serper search + LLM keyword extraction + product filter. Business plan generation (10 sections) + Word + Excel per plan. Multi-plan support (per-friction folders). Two-stage HITL resume: stage 1 (friction review → shortlist + business plan gate), stage 2 (business plan selection → generate). 208 tests passing. Live: care home query generated 3 plans (ChefCare Connect 1, CareEase Solutions 2, CareSync UK 6). |


## 4. Sources

| Source | Type | Cost | Status | Notes |
|---|---|---|---|---|
| Adzuna | API | Free (1,000/mo) | Built | Job ads |
| Reed | API | Free | Built | Job ads |
| Jooble | API | Free (500 lifetime) | Skipped | Weak UK coverage |
| Reddit | API | Free | Skipped | Suspended, appeal pending |
| ONS | API | Free | In progress | Context only |
| Companies House | API | Free (600/5min) | In progress | SIC-level firm counts |
| Capterra | Playwright | Free | Pending | Reviews |
| G2 | Playwright | Free | Pending | Reviews |
| Trustpilot | Playwright | Free | Pending | Reviews |
| UKBF | Playwright | Free | Pending | Forums |
| Google Patents | API | Free | Pending | Patents |
| UK IPO | API | Free | Pending | Patents |
| EPO | API | Free | Built | Patents (OAuth2). Industry-agostic. |
| Trade body sites | Playwright | Free | Pending | Distribution + GraphRAG PDFs |
| Contracts Finder | API | Free | Built | UK public tenders. POST+JSON. No auth. Industry-agnostic. |
| Hacker News | API | Free | Built | B2B discussions via Algolia search. No auth. |
| Stack Exchange | API | Free | Built | Practitioner forums (Workplace, PM, Freelancing). Optional key. |
| NOMIS | API | Free | Built (live deferred) | UK labour stats. 406 on filtered queries. |

----

## 5. Models

| Model | Where used | Notes |
|---|---|---|
| Qwen 2.5 7B Instruct | Nodes 4, 6, 11 extraction | Modal GPU |
| Qwen 2.5 3B Instruct | Node 8 classification | Modal GPU |
| Claude Sonnet 5 | Nodes 5, 9, 10, 11, 12 | Frontier API |
| all-MiniLM-L6-v2 | Nodes 5, 6 embeddings | Local E7270 |
| scikit-learn | Node 5 clustering | Local E7270 |

---

## 6. Verification Log

See `docs/sic-process.md` for the full process, file structure, and re-download instructions.

| Item | Claim | Source | Verified? |
|---|---|---|---|
| SIC 49410 | Freight transport by road | ONS SIC 2026 (Summary file) | **Verified 2026-09-16** |
| Firm count 76,479 (SIC 49410) | Active UK firms | Companies House API | **Verified 2026-09-16** (SIC confirmed; count from API) |
| EPO OPS reachable | OAuth2 auth + search + detail work | Live query 2026-10-21 | **Verified 2026-10-21** |
| EPO OPS landscape | 8 industries tested (logistics, finance, HR, healthcare, legal, sales, property, manufacturing) — all return results, no bias | Live query 2026-10-21 | **Verified 2026-10-21** |
| sic_mapper section_of() | 49410→H, 41201→F, 81100→N | Live test 2026-10-21 | **Verified 2026-10-21** |
| Contracts Finder reachable | POST JSON returns real UK tenders, 5 per keyword across 8 industries | Live query 2026-10-22 | **Verified 2026-10-22** |
| Hacker News reachable | Algolia search returns hits for 8 industries | Live query 2026-10-22 | **Verified 2026-10-22** |
| Stack Exchange reachable | API v2.3 returns questions across 5 sites | Live query 2026-10-22 | **Verified 2026-10-22** |
| Graph runs end-to-end | 16 nodes execute, HITL pause works, decision recorded | pytest 2026-10-27 | **Verified 2026-10-27** |
| Phase 6 live run | 57/60 job ads extracted with Qwen 7B on Modal | Query "payroll" 2026-10-28 | **Verified 2026-10-28** |
| Phase 7 live run | 14 frictions with real sentences, IP, patents, proofs | Query "payroll" 2026-10-29 | **Verified 2026-10-29** |
| Phase 8 live run | 4 distinct frictions, scored, written to runs/ | Query "payroll" 2026-10-30 | **Verified 2026-10-30** |
| Phase 8 live run | 5 frictions, scored, output to runs/ | Query "payroll" 2026-10-30 | **Verified 2026-10-30** |
| Phase 9 HITL flow | Graph pauses, Streamlit review, resume completes | 2 runs tested 2026-10-30 | **Verified 2026-10-30** |
| Phase 10 care home run | 420 job ads → 9 frictions, top score 4.91 | Query "care home" 2026-10-30 | **Verified 2026-10-30** |
| Phase 11 Node 11 | Real Serper search returns product competitors | 10 frictions, live query 2026-10-31 | **Verified 2026-10-31** |
| Phase 11 business plan | 3 full plans generated (narrative + Word + Excel) | care home run 2026-10-31 | **Verified 2026-10-31** |



See `docs/ons-process.md` for the ONS purpose, dataset types, priority order, and approval flow.

See `docs/patents-process.md` for the patent sources, decisions, and access methods.

---

## 7. Open Questions

- Which ONS datasets to approve? (in progress)
- Which SIC codes to target as seeds?
- When to write `companies_house.py`?
- Is ONS worth keeping at all, given broad groups?
ONS purpose clarified: metadata only, evidence for business plan

GVA/ASHE moved to "business plan stage"

Manual CSV layer deferred to business plan phase

---

## 8. What's Locked

- Friction-first, no industries upfront
- Node 7 counts unique employers, 500+ threshold
- Derived pricing from friction cost
- £3M ARR target
- 16 nodes in main graph
- Qwen + Claude + MiniLM models
- Modal for GPU
- MLflow for tracking
- Append-only ledger
- No auto-learning
- No paid tools without approval
- Verification rule: every number traces to source
