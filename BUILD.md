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
| 6 — Fine-tune SLMs | In progress |
| 7 — Frontier API | Pending |
| 8 — Scoring + output | Pending |
| 9 — Human review UI | Pending |
| 10 — First full run | Pending |
| 11 — Refine | Pending |

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

### Phases 4–11

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
