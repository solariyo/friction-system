
---

# Updated `README.md` — full

Save over `README.md`.

```markdown
# Friction System

A research system that finds UK B2B frictions from live signals — job ads, tenders, patents, forum posts, and **documents** (annual reports, council minutes, trade bodies, consulting firms) — and produces a shortlist of buildable business ideas for the Innovator Founder application.

The system is friction-first. It does not start from a list of industries. It finds frictions from live signals, then checks whether the market behind each friction is big enough. Industry is a label, not a filter.

---

## Table of contents

1. [What it does](#1-what-it-does)
2. [How it works](#2-how-it-works)
3. [Requirements](#3-requirements)
4. [Setup](#4-setup)
5. [Configuration](#5-configuration)
6. [Running](#6-running)
7. [Project layout](#7-project-layout)
8. [Data and storage](#8-data-and-storage)
9. [API keys](#9-api-keys)
10. [Links](#10-links)
11. [Models](#11-models)
12. [Cost model](#12-cost-model)
13. [Naming conventions](#13-naming-conventions)
14. [Glossary](#14-glossary)
15. [Verification](#15-verification)
16. [Backup and restore](#16-backup-and-restore)
17. [MLflow](#17-mlflow)
18. [Troubleshooting](#18-troubleshooting)
19. [Testing philosophy](#19-testing-philosophy)
20. [Security notes](#20-security-notes)
21. [What's out of scope](#21-whats-out-of-scope)
22. [Future extensions](#22-future-extensions)
23. [Changelog](#23-changelog)
24. [Rules](#24-rules)
25. [Build tracker](#25-build-tracker)

---

## 1. What it does

The system ingests raw, live signals from the UK web, extracts structure from them, and surfaces recurring problems — frictions — that businesses are paying humans to solve manually. It then filters those frictions down to the ones worth building a business around.

The pipeline:

1. **User chooses mode** — ads only, docs only, or both
2. **Docs mode:** searches for PDFs (annual reports, council minutes, trade bodies, consulting, regulators), HITL reviews, downloads approved, extracts friction hints
3. **Ads mode:** scrapes job ads, tenders, patents, forum posts
4. **Merges signals** from all sources
5. **Extracts structure** — roles, salaries, tasks, patterns
6. **Clusters into frictions**
7. **Counts firms** showing each friction (500+ employer threshold)
8. **Checks regulation, IP, patents, competitors**
9. **Computes cost and derived price**
10. **Scores and ranks**
11. **You review and approve**
12. **Outputs a shortlist** and narrative for the Innovator Founder application
13. **Optionally generates business plans** (Word + Excel) and lead lists (CSV with contacts)

---

## 2. How it works

The system has **two signal discovery modes** (ads, docs) and **three tracking tools**.

**The main graph** is 16 nodes + auxiliary nodes (Node 0.5 mode gate, Node 4.5 employers, Node 20a/b docs, Node 19 contact labeling). Node 7 filters hard — only frictions with 500+ named employers survive. Node 15 pauses the whole graph and waits for you to approve or reject each candidate.

**The context layer** supports the graph with data that doesn't come from job ads. Companies House gives SIC-level firm counts + officers + filing history. ONS gives employment trends. Patents give IP context. Documents give real-world friction evidence.

**Three tracking tools** keep everything traceable. Git tracks files. MLflow tracks runs — which weights were used, what the outcome was. SQLite tracks results — every friction, every decision, every lifecycle event.

### The nodes

| Node | Purpose |
|---|---|
| 0.5 | Mode gate (ads/docs/both) |
| 1 | Scrape job ads |
| 2 | Scrape reviews |
| 3 | Scrape forums |
| 4 | Extract and tag (Qwen) |
| 4.5 | Extract employers → companies table |
| 5 | Cluster into frictions (accepts ads + forum posts + documents) |
| 6 | Deduplicate |
| 7 | Count signal firms (500+) |
| 8 | Check regulation |
| 9 | Assess IP potential |
| 10 | Patent landscape |
| 11 | Check competitors |
| 12 | Attach proofs |
| 13 | Compute cost |
| 14 | Score |
| 15 | Human review |
| 16 | Terminal output |
| 19 | Label contacts per friction |
| 20a | Search documents (PDF candidates) |
| 20b | Download + extract approved docs |
| 16.5 | Business plan generation |
| 16.6 | Lead list generation |

### 2.1 Signal sources — why each source exists

Every source has one job: confirm or discover a friction. None is industry-specific. All take a keyword and return data about that keyword. Industry is a label applied later, never a filter.

#### Discovery sources

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Adzuna** | UK job ads. Primary discovery. | API key (free, 1,000/mo) | Built |
| **Reed** | UK job ads. Second discovery. | API key (free) | Built |
| **Documents (SearXNG)** | Annual reports, trade bodies, consulting, regulators, think tanks, industry bodies — auto-detected per query | None | Built (C1–C3) |
| **GOV.UK Find a Job** | Public-sector job ads. | API key (free) | Not built |

#### Confirmation sources

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Companies House** | SIC firm counts, officers, filing history | API key (free, 600/5min) | Built |
| **Contracts Finder** | UK public-sector tenders | None | Built |
| **Find a Tender** | UK/EU above-threshold tenders | None | Not built |
| **ONS** | Employment trends, industry stats (context only) | Optional key | Partial |
| **NOMIS** | Granular employment by SIC/region | None | Built, endpoint broken |
| **EPO OPS** | Patents | OAuth2 (free) | Built |
| **Hacker News** | B2B / startup discussions | None | Built |
| **Stack Exchange** | Practitioner forums | Optional key | Built |
| **OpenAlex** | Academic research | None | Not built |
| **TPOS** | Property Ombudsman (UK estate agents) | None | Built |

#### Context sources

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Trade body PDFs** | Feed a GraphRAG store for narrative | None | Partial (via documents) |

#### Dropped sources — not usable

Capterra, Trustpilot, G2, UKBF (Playwright); Google Patents (no official API); UK IPO (bulk download only); Reddit (suspended); Jooble (weak UK); WebS.io (deferred).

### 2.2 Why sources are API-only

Playwright sources were dropped. Reason: scraping is fragile, slow, and legally grey. Every active source is an official public API — documented, stable, legal, and free. Document search uses SearXNG (local meta-search) — not a browser.

---

## 3. Requirements

The system runs on a standard Windows laptop. No GPU required — Modal handles all GPU work.

| Item | Value |
|---|---|
| OS | Windows 10/11 (macOS/Linux work too) |
| Python | 3.11+ |
| Conda | Anaconda or Miniconda |
| Disk | ~7 GB (+ PDF cache) |
| RAM | 8 GB minimum, 16 GB recommended |
| GPU | Not required |
| Docker | Required for SearXNG (search) |
| Internet | Required |

---

## 4. Setup

Every command below is copy-pasteable in PowerShell.

### 4.1 Create conda environment

```powershell
conda create -n friction python=3.11
conda activate friction
```

### 4.2 Install dependencies

```powershell
cd C:\Users\T\friction-system
pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install -e .
```

### 4.3 Install pypdf (document extraction)

```powershell
pip install pypdf
```

### 4.4 Install Playwright Chromium

Playwright is still installed for future use, though no active source uses it.

```powershell
playwright install chromium
```

### 4.5 Authenticate Modal

```powershell
modal token new
modal profile current
```

### 4.6 Start SearXNG (Docker)

```powershell
docker run -d --name searxng -p 8080:8080 -v C:\Users\T\friction-system\searxng\settings.yml:/etc/searxng/settings.yml searxng/searxng
```

Verify:

```powershell
curl http://localhost:8080/search?q=test&format=json
```

### 4.7 Create data folders

```powershell
mkdir data
mkdir data\ons
mkdir data\ons\cache
mkdir data\docs
```

### 4.8 Copy `.env.example` to `.env`

```powershell
Copy-Item .env.example .env
```

Fill in keys (section 9).

---

## 5. Configuration

Four config files drive the system. Three live in `config/`. One is `.env`.

### 5.1 `config/settings.yaml`

```yaml
run:
  mode: on_demand
  max_age_days: 90
  min_salary: 28000
  signal_firm_threshold: 500

scoring:
  weights_version: 1

models:
  embedding: all-MiniLM-L6-v2
  slm_extract: Qwen/Qwen2.5-7B-Instruct
  slm_classify: Qwen/Qwen2.5-3B-Instruct
  frontier_provider: groq
  frontier: claude-sonnet-4-5
  frontier_free: openai/gpt-oss-120b

storage:
  db_path: storage/friction.db

clustering:
  n_clusters_max: 10
  n_clusters_min: 2
  signals_per_cluster: 8
  dedupe_threshold: 0.80

mlflow:
  tracking_uri: http://127.0.0.1:5000
  experiment_name: friction-system

ons:
  api_key: ${ONS_API_KEY}
  cache_expiry_days: 30
  catalogue_refresh_days: 30
```

### 5.2 `config/sources.yaml`

```yaml
sources:
  adzuna:
    enabled: true
    type: api
  reed:
    enabled: true
    type: api
  companies_house:
    enabled: true
    type: api
  contracts_finder:
    enabled: true
    type: api
  epo:
    enabled: true
    type: api
  ons:
    enabled: true
    type: api
  hacker_news:
    enabled: true
    type: api
  stack_exchange:
    enabled: true
    type: api
```

### 5.3 `config/scoring_weights.yaml`

```yaml
version: 1
weights:
  signal_firm: 0.20
  regulation: 0.10
  price: 0.15
  demand: 0.15
  trigger: 0.10
  proof: 0.10
  ip: 0.10
  patent: 0.05
  competitors: 0.05
```

### 5.4 `.env`

Secrets only. Never committed. See section 9 for keys.

### 5.5 `searxng/settings.yml`

SearXNG Docker config. See `searxng/settings.yml`.

---

## 6. Running

### Start MLflow (separate terminal)

```powershell
conda activate friction
mlflow server --host 127.0.0.1 --port 5000
```

UI at http://127.0.0.1:5000

### Full pipeline — ads mode (default)

```powershell
python -m friction_system.main "estate agency" -v
```

### Full pipeline — docs mode

```powershell
python -m friction_system.main "estate agency" -v --mode docs
```

### Full pipeline — both modes

```powershell
python -m friction_system.main "estate agency" -v --mode both
```

### Dry-run (cache only, no API calls)

```powershell
python -m friction_system.main "estate agency" -v --dry-run
```

### Resume after HITL pause

```powershell
python -m friction_system.resume <run_id>
```

### Streamlit UIs

```powershell
streamlit run src/friction_system/ui/review_app.py
streamlit run src/friction_system/ui/ons_review.py
```

### Run individual sources (smoke tests)

```powershell
python -m friction_system.sources.adzuna
python -m friction_system.sources.reed
python -m friction_system.sources.companies_house
python -m friction_system.sources.epo
python -m friction_system.sources.contracts_finder
python -m friction_system.sources.ons_catalogue
```

### Run tests

```powershell
pytest tests/ -v
```

---

## 7. Project layout

```
friction-system/
├── BUILD.md                        # master build tracker
├── README.md                       # this file
├── pyproject.toml
├── .env / .env.example
├── .gitignore
│
├── config/
│   ├── sources.yaml
│   ├── settings.yaml
│   └── scoring_weights.yaml
│
├── searxng/
│   └── settings.yml                # SearXNG Docker config
│
├── src/friction_system/
│   ├── storage/                    # SQLite schema, models, queries
│   │   ├── schema.sql
│   │   ├── db.py
│   │   ├── models.py
│   │   └── queries.py
│   ├── sources/                    # API clients
│   │   ├── base.py
│   │   ├── adzuna.py
│   │   ├── reed.py
│   │   ├── companies_house.py
│   │   ├── contracts_finder.py
│   │   ├── epo.py
│   │   ├── ons.py
│   │   ├── ons_catalogue.py
│   │   ├── sic_mapper.py
│   │   ├── hacker_news.py
│   │   ├── stack_exchange.py
│   │   ├── tpos_scraper.py
│   │   ├── searxng.py
│   │   ├── search_router.py
│   │   ├── serper.py
│   │   ├── company_dedupe.py
│   │   ├── enrichment_cache.py
│   │   ├── facebook_lookup.py
│   │   ├── yell_scraper.py
│   │   ├── linkedin_reverse.py
│   │   ├── documents_searcher.py
│   │   ├── industry_bodies.py
│   │   ├── pdf_downloader.py
│   │   ├── pdf_extractor.py
│   │   ├── crawl4ai_client.py
│   │   └── lead_extractor.py
│   ├── graph/
│   │   ├── state.py
│   │   └── main_graph.py
│   ├── nodes/                      # All nodes
│   │   ├── node_00_5_mode_gate.py
│   │   ├── node_01_scrape_job_ads.py
│   │   ├── ... (16+ total)
│   │   ├── node_04_5_extract_companies.py
│   │   ├── node_19_label_contacts.py
│   │   ├── node_20a_doc_searcher.py
│   │   ├── node_20b_doc_fetcher.py
│   │   ├── node_16_5_business_plan.py
│   │   └── node_16_6_lead_list.py
│   ├── models/                     # MiniLM, clustering, SLM, frontier
│   ├── modal/                      # serverless GPU serving
│   ├── ui/                         # Streamlit UIs
│   │   ├── review_app.py
│   │   └── ons_review.py
│   ├── output/                     # JSON + Markdown exports
│   └── graphrag/                   # PDF layer
│
├── data/
│   ├── ons/                        # reference + caches
│   ├── docs/                       # PDFs per run (gitignored)
│   ├── signal_cache/               # raw + extracted ads (gitignored)
│   └── cache/                      # SearXNG, Serper, etc. (gitignored)
│
├── storage/friction.db             # gitignored
├── runs/                           # gitignored
├── mlruns/                         # gitignored
├── mlflow.db                       # gitignored
│
├── scripts/
│   ├── smoke_test.py
│   ├── check_contacts.py
│   ├── check_companies.py
│   ├── check_contacts_table.py
│   ├── check_cfl_table.py
│   ├── check_cfr_table.py
│   ├── check_job_ads_company.py
│   ├── check_documents.py
│   └── test_node19.py
│
└── tests/
    ├── test_storage.py
    ├── test_adzuna.py
    └── ...
```

---

## 8. Data and storage

### SQLite tables

| Table | Purpose | Written by |
|---|---|---|
| `job_ads` | Raw job ads | Nodes 1–2 |
| `reviews` | Raw reviews | Node 2 (not active) |
| `forum_posts` | Raw forum posts | Node 3 |
| `tenders` | Raw public-sector tenders | Confirmation layer |
| `frictions` | Discovered frictions | Node 5 onwards |
| `friction_lifecycle` | Append-only status changes | Every status transition |
| `friction_sources` | Links friction → raw signal | Node 12 |
| `human_decisions` | Approve / reject decisions | Node 15 |
| `competitors` | Competitor findings | Node 11 |
| `model_pricing` | Model price reference | Setup |
| `run_costs` | Cost per run | Every run |
| `pdfs` | Trade body PDFs | GraphRAG layer |
| `shortlists` | Approved frictions per run | Node 16 |
| `company_enrichment` | Company cache (30-day TTL) | Node 16.6 |
| `companies` | Canonical UK businesses | Node 4.5 + 16.6 + 19 |
| `contacts` | Individuals at companies | Node 16.6 + 19 |
| `company_friction_links` | Company × friction links | Node 19 |
| `contact_friction_roles` | Contact × friction labels | Node 19 |
| `documents` | Downloaded PDFs + friction hints | Node 20b |

### Files and caches

| Location | Purpose | Gitignored |
|---|---|---|
| `storage/friction.db` | SQLite database | Yes |
| `data/ons/cache/` | ONS API caches (30-day TTL) | Yes |
| `data/signal_cache/` | Raw ads/posts + extractions | Yes |
| `data/cache/searxng/` | SearXNG results | Yes |
| `data/cache/serper/` | Serper fallback results | Yes |
| `data/docs/<run_id>/` | PDFs per run | Yes |
| `runs/` | Per-run output | Yes |
| `mlflow.db` | MLflow backend | Yes |
| `mlruns/` | MLflow artifacts | Yes |

---

## 9. API keys

### Active keys

| Key | Source | Notes |
|---|---|---|
| `ADZUNA_APP_ID` | https://developer.adzuna.com/signup | 1,000 calls/mo |
| `ADZUNA_APP_KEY` | Same | Same |
| `REED_API_KEY` | https://www.reed.co.uk/developers | Basic auth |
| `COMPANIES_HOUSE_API_KEY` | https://developer.company-information.service.gov.uk/ | 600 req/5min |
| `EPO_CONSUMER_KEY` | https://developers.epo.org/ | OAuth2 |
| `EPO_CONSUMER_SECRET` | Same | Same |
| `GROQ_API_KEY` | https://console.groq.com/ | Free |
| `GEMINI_API_KEY` | https://aistudio.google.com/app/apikey | Free |

### Optional keys

| Key | Source | Notes |
|---|---|---|
| `ONS_API_KEY` | https://developer.ons.gov.uk/ | Not required |
| `SERPER_API_KEY` | https://serper.dev/ | Fallback search only |
| `MODAL_TOKEN_ID` | https://modal.com/settings/tokens | Free $30/mo |
| `MODAL_TOKEN_SECRET` | Same | Same |
| `MODAL_QWEN_URL` | From `modal deploy` | Qwen endpoint |
| `ANTHROPIC_API_KEY` | https://console.anthropic.com/ | Paid |
| `STACK_EXCHANGE_API_KEY` | https://api.stackexchange.com/ | Optional |

---

## 10. Links

See `BUILD.md` for full list of API endpoints, docs, and model links.

---

## 11. Models

| Model | Where used | Notes |
|---|---|---|
| Qwen 2.5 7B Instruct | Nodes 4, 6, 11, 20b extraction | Modal GPU |
| Qwen 2.5 3B Instruct | Node 8 classification | Not built |
| Groq `openai/gpt-oss-120b` | Nodes 5, 9, 10, 11, 12, business plans | Free, active |
| Gemini `gemini-3.5-flash-lite` | Fallback on Groq 429 | Free |
| Claude Sonnet 5 | Optional upgrade | Paid |
| all-MiniLM-L6-v2 | Nodes 5, 6 embeddings | Local CPU |
| scikit-learn | Node 5 clustering | Local CPU |

---

## 12. Cost model

| Source | Free tier | Over-limit |
|---|---|---|
| Adzuna | 1,000 calls/mo | Paid |
| Reed | Free | — |
| Companies House | 600 req/5min | — |
| EPO OPS | ~4 GB/mo | Paid |
| SearXNG | Free (local) | — |
| Serper | 2,500 queries/mo | Paid |
| Groq | Free | Rate-limited |
| Gemini | Free | 6 req/min |
| Modal | $30/mo free | Per-second |
| Claude API | — | Paid |

**Per ads run:** ~£0.30 Modal (first run) → £0.00 (cached).
**Per docs run:** ~£0.01 (Qwen on PDFs).
**Per full run:** ~£0.30 max first time.

---

## 13. Naming conventions

| Thing | Name |
|---|---|
| Conda env | `friction` |
| Project folder | `friction-system` |
| SQLite DB | `friction.db` |
| Python package | `friction_system` |

---

## 14. Glossary

| Term | Meaning |
|---|---|
| Friction | A recurring workplace problem found across signals |
| Signal | One piece of raw evidence (job ad, tender, patent, forum post, document) |
| Discovery source | A source that finds new frictions |
| Confirmation source | A source that proves a friction is real |
| Context source | A source that supports the narrative |
| Document source | PDFs from annual reports, trade bodies, consulting, regulators |
| SLM | Small language model |
| Frontier model | Large API model (Groq, Claude, Gemini) |
| HITL | Human-in-the-loop |
| Derived price | Price computed from friction cost |
| Signal firm | A firm showing the friction via a live signal |

---

## 15. Verification

Every number that goes into a friction or business plan must trace to a verified source. See `BUILD.md` §9 for full verification log.

---

## 16. Backup and restore

**Database:**
```powershell
Copy-Item storage\friction.db storage\friction.db.bak
```

**Config:**
```powershell
git add config\
git commit -m "config snapshot"
```

**Full snapshot:** copy entire `C:\Users\T\friction-system`.

**Restore:** copy folder back, then `pip install -e .`.

---

## 17. MLflow

```powershell
mlflow server --host 127.0.0.1 --port 5000
```

UI at http://127.0.0.1:5000

---

## 18. Troubleshooting

| Symptom | Fix |
|---|---|
| `ModuleNotFoundError: friction_system` | `pip install -e .` |
| 401 from any API | Check `.env` |
| EPO 400/403 | Check creds |
| SearXNG unreachable | `docker start searxng` |
| SearXNG rate-limited | `docker restart searxng`; wait; or add Mojeek/Startpage to `settings.yml` |
| PDF download fails | Check URL ends in `.pdf`; check size <50MB |
| Node 20a pauses in-graph | Known issue — refactor queued (C3) |
| Modal auth fails | `modal token new` |
| Test file-lock | Close DB Browser |
| YAML error | 2 spaces indentation |

---

## 19. Testing philosophy

Every source gets a test file. Tests use mocked HTTP — no live API calls. Storage tests use temp DB.

---

## 20. Security notes

- `.env` never committed
- No PII stored
- Cache is local only
- Rotate API keys every 90 days

---

## 21. What's out of scope

- Customer-facing product
- Multi-user access
- Playwright scraping
- Auto-learning

---

## 22. Future extensions

- OpenAlex (academic)
- Find a Tender
- GOV.UK Find a Job
- GraphRAG for trade body PDFs (partial via documents)
- MLflow model registry
- Better SearXNG engine fallback (Mojeek, Startpage)

---

## 23. Changelog

| Date | Change |
|---|---|
| 2026-09-14 | Initial design |
| 2026-09-16 | BUILD.md created |
| 2026-10-21 | EPO OPS built |
| 2026-10-22 | Contracts Finder + HN + SE built |
| 2026-10-27 | Phase 4 complete |
| 2026-10-28 | Phase 5 + 6 complete |
| 2026-10-29 | Phase 7 complete |
| 2026-10-30 | Phase 8–10 complete |
| 2026-10-31 | Phase 11 Day 2 (business plan) |
| 2026-11-02 | SearXNG + TPOS + Gemini fallback + enrichment cache |
| 2026-11-03 | Batch A + B + C1–C3 (documents infrastructure) |

---

## 24. Rules

1. Friction-first. No industries upfront.
2. Nothing deleted. Append-only ledger.
3. No auto-learning. Human is the loop.
4. No paid tools without approval.
5. Every number traces to a verified source.
6. Cache what doesn't change often.
7. Config in files, results in DB.
8. HITL where it matters.
9. API-only sources. No scraping.

---

## 25. Build tracker

Full details — every step, decision, source, model, and verification — live in `BUILD.md`.

### Phase status

| Phase | Name | Status |
|---|---|---|
| 0–10 | Core pipeline | Done |
| 11A–11F | Business plan + leads + verbose + Batch A/B | Done |
| 11G | Batch C (documents) | In progress — refactor needed |

### Next up

- C3 refactor: Node 20a → standalone `scripts/search_docs.py`
- Live docs test when SearXNG recovers
- Batch D (friction quality + visa output)
- Batch E (multi-run + infrastructure)

---

## Version

| Field | Value |
|---|---|
| Project version | 0.1.0 |
| Build phase | 11G (documents) |
| Last updated | 2026-11-03 |
| Modal workspace | heartofchr |
| Project root | C:\Users\T\friction-system |

---

## License

Personal project. Not for redistribution.
```

---

## Save + commit

```powershell
git add BUILD.md README.md
git commit -m "BUILD.md + README.md: full sync through Batch C3 (documents infrastructure). 40/81 items done."
```