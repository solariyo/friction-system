# Friction System

A research system that finds UK B2B frictions from live signals — job ads, tenders, patents, and forum posts — and produces a shortlist of buildable business ideas.

The system is friction-first. It does not start from a list of industries. It finds frictions from live signals, then checks whether the market behind each friction is big enough. Industry is a label, not a filter.

----

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

1. Scrapes live signals — job ads, tenders, patents, forum posts
2. Extracts structured data — roles, salaries, tasks, patterns
3. Clusters signals into frictions
4. Counts firms showing each friction (500+ employer threshold)
5. Checks regulation, IP potential, patents, competitors
6. Computes cost and derived price
7. Scores and ranks
8. You review and approve
9. Outputs a shortlist and narrative for the Innovator Founder application

---

## 2. How it works

The system has two layers and three tracking tools.

**The main graph** is 16 nodes that run on live signals. Each node does one job and passes state to the next. Node 7 filters hard — only frictions with 500+ named employers survive. Node 15 pauses the whole graph and waits for you to approve or reject each candidate.

**The context layer** supports the graph with data that doesn't come from job ads. Companies House gives SIC-level firm counts. ONS gives employment trends. Patents give IP context. Trade body PDFs feed a GraphRAG store used for the final narrative.

**Three tracking tools** keep everything traceable. Git tracks files. MLflow tracks runs — which weights were used, what the outcome was. SQLite tracks results — every friction, every decision, every lifecycle event.

### The 16 nodes

| Node | Purpose |
|---|---|
| 1 | Scrape job ads |
| 2 | Scrape reviews |
| 3 | Scrape forums |
| 4 | Extract and tag |
| 5 | Cluster into frictions |
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

### 2.1 Signal sources — why each source exists

Every source has one job: confirm or discover a friction. None is industry-specific. All take a keyword and return data about that keyword. Industry is a label applied later, never a filter.

#### Discovery sources — where frictions are found

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Adzuna** | UK job ads. The primary discovery source. Roles, salaries, tasks — the raw material for friction extraction. | API key (free, 1,000/mo) | Built, tested |
| **Reed** | UK job ads. Second discovery source. Broader coverage, especially SME roles. | API key (free) | Built, tested |
| **GOV.UK Find a Job** | UK public-sector job ads. Confirms frictions that appear in public-sector roles. | API key (free) | Not built |

#### Confirmation sources — proof that a friction is real

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Companies House** | SIC-level firm counts. Confirms 500+ named employers exist in the friction's market. | API key (free, 600 req/5min) | Built, tested |
| **Contracts Finder** | UK public-sector tenders. Confirms someone is already paying real money to solve the friction. | None | Built, tested |
| **Find a Tender** | UK/EU above-threshold tenders. Bigger money than Contracts Finder. | None | Not built (endpoint confirmed reachable) |
| **ONS** | UK employment trends and industry stats. Confirms market size and direction. Context only, not signal. | Optional key | Partial (catalogue + dataset metadata) |
| **NOMIS** | Granular UK employment by SIC, region, occupation. Confirms a role exists in enough firms at finer grain than ONS. | None | Not built |
| **EPO OPS** | European Patent Office. Confirms a friction has not already been patented. Industry-agnostic. | OAuth2 (free) | Built, tested |
| **Hacker News** | B2B / startup discussions. Confirms frictions are discussed by practitioners, not just employers. | None | Built, tested |
| **Stack Exchange** | Workplace, freelancing, project management Q&A. Confirms frictions in practitioner language. | Optional key | Built, tested |
| **OpenAlex** | Academic research. Confirms a problem is studied — signals it's recognised, not invented. | None | Not built |

#### Context sources — narrative support

| Source | Purpose | Auth | Status |
|---|---|---|---|
| **Trade body PDFs** | Feed a GraphRAG store used for the visa narrative. | None | Not built |

#### Dropped sources — not usable

| Source | Reason |
|---|---|
| Capterra | Requires Playwright. Review scraping out of scope. |
| Trustpilot | Requires Playwright. |
| G2 | Requires Playwright. |
| UKBF | Requires Playwright. |
| Google Patents | No official API. Scraping blocked. |
| UK IPO | No API. Bulk download only. |
| Reddit | Account suspended. Appeal pending. |
| Jooble | Weak UK coverage. |
| WebS.io | Deferred. Not needed while free official APIs cover the same signal types. |

### 2.2 Why sources are API-only

Playwright sources were dropped entirely. Reason: scraping is fragile (HTML changes), slow (headless browsers), and legally grey (ToS). Every active source is an official public API — documented, stable, legal, and free. Industry-agnostic by design.

---

## 3. Requirements

The system runs on a standard Windows laptop. No GPU required — Modal handles all GPU work. You need conda for environment isolation, and about 7 GB of disk for dependencies, Playwright, and models.

| Item | Value |
|---|---|
| OS | Windows 10/11 (macOS/Linux work too) |
| Python | 3.11+ |
| Conda | Anaconda or Miniconda |
| Disk | ~7 GB |
| RAM | 8 GB minimum, 16 GB recommended |
| GPU | Not required |
| Internet | Required |

---

## 4. Setup

Every command below is copy-pasteable in PowerShell. Follow in order.

### 4.1 Create conda environment

The system uses its own conda environment called `friction`. Never share environments between projects.

```powershell
conda create -n friction python=3.11
conda activate friction
```

### 4.2 Install dependencies

Install CPU-only torch first — it's 10× smaller than the CUDA version and the E7270 has no GPU. Then install the project.

```powershell
cd C:\Users\T\friction-system
pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install -e .
```

### 4.3 Install Playwright Chromium

Playwright is still installed for future use, though no active source uses it.

```powershell
playwright install chromium
```

### 4.4 Verify

Two checks: the package imports, and Chromium launches.

```powershell
python -c "import friction_system; print('ok')"
python -c "from playwright.sync_api import sync_playwright; p = sync_playwright().start(); b = p.chromium.launch(); b.close(); p.stop(); print('playwright ok')"
```

### 4.5 Authenticate Modal

Modal runs the Qwen models on serverless GPU. Authenticate once.

```powershell
modal token new
modal profile current
```

### 4.6 Create data folders

```powershell
mkdir data
mkdir data\ons
mkdir data\ons\cache
```

### 4.7 Copy `.env.example` to `.env`

The `.env` file holds your secrets. Copy the template, then fill in keys (section 9).

```powershell
Copy-Item .env.example .env
```

---

## 5. Configuration

Four config files drive the system. Three live in `config/`. One is `.env`.

### 5.1 `config/settings.yaml`

System-wide thresholds and model choices. Edit when you want to change behaviour — no code changes needed.

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
  frontier: claude-sonnet-5

storage:
  db_path: storage/friction.db

scalability:
  growth_lookback_days: 365
  geo_scope_default: UK

mlflow:
  tracking_uri: http://127.0.0.1:5000
  experiment_name: friction-system

ons:
  api_key: ${ONS_API_KEY}
  cache_expiry_days: 30
  catalogue_refresh_days: 30
```

The most important keys:

| Key | Meaning |
|---|---|
| `max_age_days` | Only fetch job ads from last N days |
| `min_salary` | Ignore roles below this |
| `signal_firm_threshold` | Node 7 minimum employers |
| `db_path` | Where SQLite lives |

### 5.2 `config/sources.yaml`

Enable or disable any source. Flip `enabled` to `false` to skip.

```yaml
sources:
  adzuna:
    enabled: true
    type: api
  reed:
    enabled: true
    type: api
  govuk_jobs:
    enabled: false
    type: api
  jooble:
    enabled: false
    type: api
  reddit:
    enabled: false
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
    enabled: false
    type: api
  stack_exchange:
    enabled: false
    type: api
  openalex:
    enabled: false
    type: api
  nomis:
    enabled: false
    type: api
```

### 5.3 `config/scoring_weights.yaml`

Weights for Node 14 scoring. Editable without touching code. Git tracks every change.

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

---

## 6. Running

Every session starts the same way.

```powershell
conda activate friction
cd C:\Users\T\friction-system
```

### Start MLflow before any run

MLflow logs every run. Start it in a separate terminal and leave it running.

```powershell
conda activate friction
mlflow server --host 127.0.0.1 --port 5000
```

UI at http://127.0.0.1:5000

### Run individual sources (smoke tests)

Each source runs standalone. Useful for testing and for confirming the API is live.

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
pytest tests/test_storage.py -v
pytest tests/test_adzuna.py -v
pytest tests/test_reed.py -v
pytest tests/test_companies_house.py -v
pytest tests/test_epo.py -v
pytest tests/test_contracts_finder.py -v
```

### Full system run (once built)

```powershell
python -m friction_system.main
```

### Streamlit UIs (once built)

```powershell
streamlit run src/friction_system/ui/review_app.py
streamlit run src/friction_system/ui/ons_review.py
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
├── src/friction_system/
│   ├── storage/                    # SQLite schema, models, queries
│   │   ├── schema.sql
│   │   ├── db.py
│   │   ├── models.py
│   │   └── queries.py
│   ├── sources/                    # API clients (API-only, no scraping)
│   │   ├── base.py
│   │   ├── adzuna.py
│   │   ├── reed.py
│   │   ├── companies_house.py
│   │   ├── contracts_finder.py
│   │   ├── epo.py
│   │   ├── ons.py
│   │   ├── ons_catalogue.py
│   │   └── sic_mapper.py
│   ├── graph/                      # LangGraph setup [not built]
│   ├── nodes/                      # 16 nodes [not built]
│   ├── models/                     # MiniLM, clustering, SLM, frontier [not built]
│   ├── modal/                      # serverless GPU serving [not built]
│   ├── ui/                         # Streamlit UIs
│   │   └── ons_review.py
│   ├── output/                     # JSON + Markdown exports [not built]
│   └── graphrag/                   # PDF layer [not built]
│
├── data/
│   └── ons/
│       ├── catalogue.json
│       ├── approved_datasets.json
│       ├── sic_codes.json
│       ├── sic/                    # source XLSX files
│       └── cache/                  # gitignored
│           ├── epo/
│           ├── contracts_finder/
│           ├── companies_house/
│           └── ...
│
├── storage/friction.db             # gitignored
├── runs/                           # gitignored
├── mlruns/                         # gitignored
├── mlflow.db                       # gitignored
│
├── tests/
│   ├── test_storage.py
│   ├── test_adzuna.py
│   ├── test_reed.py
│   ├── test_companies_house.py
│   ├── test_contracts_finder.py
│   ├── test_epo.py
│   └── test_setup.py
│
├── docker/
├── prompts/
└── docs/
│   ├── sic-process.md
│   ├── ons-process.md
│   └── patents-process.md
│   ├── graph/
│   │   ├── state.py
│   │   └── main_graph.py
│   ├── nodes/
│   │   ├── node_01_scrape_job_ads.py
│   │   ├── ... (16 total)
│   │   └── node_16_terminal_output.py
│   ├── main.py
```

---

## 8. Data and storage

The system stores data in three places. SQLite holds results. The `data/` folder holds reference data and caches. The `runs/` folder holds per-run output.

### SQLite tables

| Table | Purpose | Written by |
|---|---|---|
| `job_ads` | Raw job ads | Nodes 1–2 |
| `reviews` | Raw reviews | Node 2 (not active — Playwright dropped) |
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

### Files and caches

| Location | Purpose | Gitignored |
|---|---|---|
| `storage/friction.db` | SQLite database | Yes |
| `data/ons/cache/` | All API caches (30-day TTL) | Yes |
| `data/ons/catalogue.json` | ONS dataset catalogue | No |
| `data/ons/approved_datasets.json` | Your approvals | No |
| `data/ons/sic_codes.json` | SIC lookup | No |
| `runs/` | Per-run output | Yes |
| `mlflow.db` | MLflow backend | Yes |
| `mlruns/` | MLflow artifacts | Yes |

---

## 9. API keys

Every key goes in `.env`. Values never committed. Fill only what you use.

### Active keys (required for active sources)

| Key | Source | Notes |
|---|---|---|
| `ADZUNA_APP_ID` | https://developer.adzuna.com/signup | 1,000 calls/mo |
| `ADZUNA_APP_KEY` | Same | Same |
| `REED_API_KEY` | https://www.reed.co.uk/developers | Basic auth, no format |
| `COMPANIES_HOUSE_API_KEY` | https://developer.company-information.service.gov.uk/ | 600 req/5min. Raw key, Basic auth |
| `EPO_CONSUMER_KEY` | https://developers.epo.org/ | OAuth2. Free tier. |
| `EPO_CONSUMER_SECRET` | Same | Same |

### Optional keys

| Key | Source | Notes |
|---|---|---|
| `ONS_API_KEY` | https://developer.ons.gov.uk/ | Not required for public data |
| `ANTHROPIC_API_KEY` | https://console.anthropic.com/ | Paid. Few cents per run. |
| `OPENAI_API_KEY` | https://platform.openai.com/ | Paid (not used yet) |
| `MODAL_TOKEN_ID` | https://modal.com/settings/tokens | Free $30/mo |
| `MODAL_TOKEN_SECRET` | Same | Same |

### Placeholder keys (no value needed yet)

| Key | Source | Notes |
|---|---|---|
| `CONTRACTS_FINDER_API_KEY` | — | No auth required. Left blank. |
| `FIND_A_TENDER_API_KEY` | — | No auth required. |
| `NOMIS_API_KEY` | — | No auth required. |
| `STACK_EXCHANGE_API_KEY` | https://api.stackexchange.com/ | Optional, higher rate limit |

### Skipped / suspended

| Key | Reason |
|---|---|
| `GOVUK_JOBS_API_KEY` | Not built |
| `JOOBLE_API_KEY` | Weak UK coverage |
| `REDDIT_CLIENT_ID` / `SECRET` | Account suspended |

---

## 10. Links

### APIs (data sources — active)

- Adzuna: https://developer.adzuna.com/
- Adzuna signup: https://developer.adzuna.com/signup
- Reed: https://www.reed.co.uk/developers
- Companies House: https://developer.company-information.service.gov.uk/
- EPO OPS: https://developers.epo.org/
- EPO OPS documentation: https://developers.epo.org/ops/documentation
- Contracts Finder: https://www.contractsfinder.service.gov.uk/
- Contracts Finder API docs: https://www.contractsfinder.service.gov.uk/apidocumentation
- ONS Developer Hub: https://developer.ons.gov.uk/
- ONS Beta API: https://api.beta.ons.gov.uk/v1

### APIs (future sources)

- Hacker News (Algolia): https://hn.algolia.com/api
- Hacker News (Firebase): https://github.com/HackerNews/API
- Stack Exchange API: https://api.stackexchange.com/docs
- OpenAlex: https://docs.openalex.org/
- NOMIS: https://www.nomisweb.co.uk/api/v01/help
- Find a Tender: https://www.find-tender.service.gov.uk/API/Documentation

### Data sources (reference)

- Companies House SIC list: https://resources.companieshouse.gov.uk/sic/
- ONS UK SIC 2007: https://www.ons.gov.uk/methodology/classificationsandstandards/ukstandardindustrialclassificationofeconomicactivities/uksic2007
- ONS Business Register: https://www.ons.gov.uk/businessindustryandtrade/business/activitysizeandlocation
- UK public sector APIs: https://www.api.gov.uk/

### Tooling

- LangGraph: https://langchain-ai.github.io/langgraph/
- Playwright Python: https://playwright.dev/python/docs/intro (installed but not active)
- Pydantic: https://docs.pydantic.dev/latest/
- httpx: https://www.python-httpx.org/
- SQLite Python: https://docs.python.org/3/library/sqlite3.html
- MLflow: https://mlflow.org/docs/latest/
- Streamlit: https://docs.streamlit.io/
- Modal: https://modal.com/docs
- python-dotenv: https://saurabh-kumar.com/python-dotenv/
- pytest: https://docs.pytest.org/

### Models

- Qwen 2.5 7B Instruct: https://huggingface.co/Qwen/Qwen2.5-7B-Instruct
- Qwen 2.5 3B Instruct: https://huggingface.co/Qwen/Qwen2.5-3B-Instruct
- all-MiniLM-L6-v2: https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2
- Claude docs: https://docs.anthropic.com/
- OpenAI docs: https://platform.openai.com/docs/

### Learning

- sentence-transformers: https://www.sbert.net/
- scikit-learn: https://scikit-learn.org/stable/
- pandas: https://pandas.pydata.org/docs/
- Hugging Face: https://huggingface.co/docs
- PEFT/LoRA: https://huggingface.co/docs/peft/index
- vLLM: https://docs.vllm.ai/

---

## 11. Models

Three model families. Each has a specific role.

**Qwen 2.5** runs on Modal for structured extraction and classification. It's fast, cheap, and strong at JSON output. Two sizes: 7B for extraction, 3B for binary classification.

**Claude Sonnet 5** runs via API for reasoning tasks. Synthesis, IP assessment, patent landscape, competitor judging, proof assembly. Few calls per run — cost is negligible.

**MiniLM** runs locally on the E7270 for embeddings. Free, fast on CPU, good enough for clustering and dedup.

### Model-to-node mapping

The 16-node graph from section 2, mapped to the model that powers each node:

| Node | Task | Model |
|---|---|---|
| 1 | Scrape job ads | — (API only) |
| 2 | Scrape reviews | — (not active) |
| 3 | Scrape forums | — (API only) |
| 4 | Extract fields | Qwen 2.5 7B Instruct |
| 5 | Cluster | MiniLM + scikit-learn |
| 5 | Synthesize friction | Claude Sonnet 5 |
| 6 | Deduplicate | MiniLM |
| 7 | Count firms | — (Companies House) |
| 8 | Regulation classify | Qwen 2.5 3B Instruct |
| 9 | IP assess | Claude Sonnet 5 |
| 10 | Patent landscape | Claude Sonnet 5 (reads EPO results) |
| 11 | Competitor judge | Qwen 2.5 7B + Claude Sonnet 5 |
| 12 | Attach proofs | Claude Sonnet 5 |
| 13 | Compute cost | — (arithmetic) |
| 14 | Score | — (weighted formula) |
| 15 | Human review | — (human) |
| 16 | Terminal output | Claude Sonnet 5 (for narrative) |

---

## 12. Cost model

The system is designed to run on free tiers. Adzuna, Reed, Companies House, Contracts Finder, EPO, ONS, Hacker News, Stack Exchange, OpenAlex, and NOMIS are all free. Modal gives $30/month free GPU. The only genuinely paid item is the frontier API (Claude), which is a few cents per run.

| Source | Free tier | Over-limit |
|---|---|---|
| Adzuna | 1,000 calls/mo | Paid |
| Reed | Free | — |
| Companies House | 600 req/5min | — |
| Contracts Finder | Free, no auth | — |
| EPO OPS | ~4 GB/mo | Paid |
| ONS | Free, no key | — |
| Hacker News | Free, no auth | — |
| Stack Exchange | 300/day no key, 10k/day with key | — |
| OpenAlex | Free, no auth | — |
| NOMIS | Free, no auth | — |
| Modal | $30/mo free | Per-second |
| Claude API | — | $2 / $10 per MTok |
| MLflow | Free (local) | — |
| Streamlit | Free (local) | — |

**Per run:** ~$5–$15.
**Total build:** ~150–400 Adzuna calls. Free tier covers it.

---

## 13. Naming conventions

Keep names consistent to avoid confusion across files and services.

| Thing | Name |
|---|---|
| Conda env | `friction` |
| Project folder | `friction-system` |
| Scratch folder | `friction-system-sub` |
| Companies House app | `friction` |
| MLflow experiment | `friction-system` |
| SQLite DB | `friction.db` |
| Python package | `friction_system` |

---

## 14. Glossary

| Term | Meaning |
|---|---|
| Friction | A recurring workplace problem found across signals |
| Signal | One piece of raw evidence (job ad, tender, patent, forum post) |
| Discovery source | A source that finds new frictions (Adzuna, Reed) |
| Confirmation source | A source that proves a friction is real (Companies House, Contracts Finder, EPO) |
| Context source | A source that supports the narrative, not the signal (ONS, trade body PDFs) |
| Pattern label | Which of 5 shapes the friction fits |
| SIC code | UK industry classification code |
| SLM | Small language model |
| Frontier model | Large API model (Claude, GPT) |
| HITL | Human-in-the-loop |
| Derived price | Price computed from friction cost |
| Signal firm | A firm showing the friction via a live signal |
| Tender | A public-sector contract opportunity (Contracts Finder, Find a Tender) |
| Patent landscape | Aggregate view of patents for a keyword (counts, applicants, IPC, years) |
| Prior art | Existing patents or knowledge that could block a new patent |
| Novelty signal | Whether a friction is already patented (from EPO + Claude) |
| GraphRAG | Graph-based retrieval over trade body PDFs |
| Node | One step in the main graph |
| Lifecycle | Append-only record of a friction's status changes |

---

## 15. Verification

Every number that goes into a friction or the business plan must trace to a verified source. No guessing.

**SIC codes** come from the official Companies House list. Never invent them. Log code and description in `data/ons/sic_codes.json`.

**Firm counts** come from Companies House API. Where possible, cross-check against a second source. Log every verification in `BUILD.md`.

**ONS datasets** get approved through the review UI. Verify numbers against known sources before approving. Log the verification date and source.

**Patent landscapes** come from EPO OPS. Log the keyword, jurisdiction split, and count in `BUILD.md`.

**Tenders** come from Contracts Finder. Log the keyword and count in `BUILD.md`.

See `docs/sic-process.md` for the SIC code process.

---

## 16. Backup and restore

The system has three things worth backing up: the database, the config, and the run output.

**Database:**

```powershell
Copy-Item storage\friction.db storage\friction.db.bak
```

**Config:**

```powershell
git add config\
git commit -m "config snapshot"
```

**Full snapshot:** copy the entire `C:\Users\T\friction-system` folder.

**Restore:** copy the folder back, then `pip install -e .` to re-link the package.

---

## 17. MLflow

MLflow tracks every run. Which weights were used, what the metrics were, what artifacts were produced. Git tracks files; MLflow tracks runs.

**Start the server:**

```powershell
mlflow server --host 127.0.0.1 --port 5000
```

**View the UI:** http://127.0.0.1:5000

**Storage:** `mlflow.db` (backend), `mlruns/` (artifacts). Both gitignored.

---

## 18. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `ModuleNotFoundError: friction_system` | Package not installed | `pip install -e .` |
| Playwright browser missing | Chromium not installed | `playwright install chromium` |
| 401 from any API | Wrong key | Check `.env`, no quotes/spaces |
| EPO 400/403 | Token expired or wrong creds | Check `EPO_CONSUMER_KEY` and `EPO_CONSUMER_SECRET` |
| EPO 429 | Rate limited | Retries are automatic; reduce `sample` size |
| Contracts Finder 400 | GET used instead of POST | Source uses POST; if calling manually, POST + JSON body |
| Contracts Finder timeout | Server slow | Retries are automatic; 60s timeout |
| YAML error | Indentation | 2 spaces per level |
| Modal auth fails | Token expired | `modal token new` |
| Test file-lock on Windows | DB Browser open | Close it, rerun |
| `settings.yaml must contain storage.db_path` | Bad indentation | Check YAML nesting |

---

## 19. Testing philosophy

Every source gets a test file. Tests use mocked HTTP — no live API calls. Storage tests use a temporary database, never the real one. One test file per module. The aim is full coverage of core logic.

Tests are also the reference for the response shape. When an API changes, the test fails first, and the failure tells you which field moved.

---

## 20. Security notes

- `.env` never committed
- `.env.example` holds no values
- Rotate API keys every 90 days
- No PII stored — all signals are public
- No external database — everything local
- Cache is local only

---

## 21. What's out of scope

- Customer-facing product (this is a research tool)
- Multi-user access
- Paid services without approval
- Playwright scraping (all dropped — API-only now)
- Blocked scraping (Indeed, LinkedIn, Google, Facebook)
- Auto-learning

---

## 22. Future extensions

Not built, but possible:

- Hacker News (Algolia search) — B2B confirmation source
- Stack Exchange — Workplace / Freelancing confirmation
- OpenAlex — academic confirmation
- NOMIS — granular UK labour market
- Find a Tender — larger tenders than Contracts Finder
- GOV.UK Find a Job — public-sector job ads
- GraphRAG layer for trade body PDFs
- Fine-tuned SLMs on Modal
- MLflow model registry
- WebS.io — if UKBF or other no-API source is needed

---

## 23. Changelog

| Date | Change |
|---|---|
| 2026-09-14 | Initial design |
| 2026-09-14 | MLflow added |
| 2026-09-14 | 16-node graph finalised |
| 2026-09-15 | Jooble skipped |
| 2026-09-15 | Reddit suspended, appeal sent |
| 2026-09-16 | ONS added (context only) |
| 2026-09-16 | Companies House added back |
| 2026-09-16 | BUILD.md created |
| 2026-09-16 | Verification rule locked |
| 2026-10-21 | EPO OPS built and verified (industry-agnostic, 8 industries tested) |
| 2026-10-21 | `sic_mapper.py` fixed (`DIVISIONS_BY_SECTION` defined) |
| 2026-10-21 | `ons.py` `query_ons()` removed (ONS stays context-only) |
| 2026-10-21 | `.env.example` created |
| 2026-10-22 | Contracts Finder built and verified (POST+JSON, no auth) |
| 2026-10-22 | Playwright sources dropped — API-only policy |
| 2026-10-22 | README.md updated (source purposes, current state) |
| 2026-10-22 | Stack Exchange built and verified (5 sites, optional key) |
| 2026-10-27 | Phase 4 complete. 16-node graph skeleton runs end-to-end. HITL pause at Node 15. 125 tests passing. |



See `BUILD.md` for full details.

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

## 25. Build tracker

Full details — every step, decision, source, model, and verification — live in `BUILD.md`.

### Phase status

| Phase | Name | Status |
|---|---|---|
| 0 | Setup | Done |
| 1 | Storage | Done |
| 2 | Adzuna | Done |
| 3 | Sources (API-only) | Done |
| 4 | Graph skeleton | Done |
| 5 | Local models | In Progress |
| 6 | Fine-tune SLMs | Pending |
| 7 | Frontier API | Pending |
| 8 | Scoring + output | Pending |
| 9 | Human review UI | Pending |
| 10 | First full run | Pending |
| 11 | Refine | Pending |

### Phase 3 — Sources

| Source | Role | Status |
|---|---|---|
| Adzuna | Job ads (discovery) | Done |
| Reed | Job ads (discovery) | Done |
| Companies House | Firm counts (confirmation) | Done |
| Contracts Finder | Tenders (confirmation) | Done |
| EPO OPS | Patents (confirmation) | Done |
| Hacker News | Forums (confirmation) | Done |
| ONS | Industry context | Partial (metadata only) |
| Find a Tender | Larger tenders | Not built |
| NOMIS | UK labour market | Not built |
| Stack Exchange | Practitioners | Done |
| OpenAlex | Academic | Not built |
| GOV.UK Find a Job | Public-sector job ads | Not built |

---

## Version

| Field | Value |
|---|---|
| Project version | 0.1.0 |
| Build phase | 3 |
| Last updated | 2026-10-22 |
| Modal workspace | heartofchr |
| Project root | C:\Users\T\friction-system |
| Scratch | C:\Users\T\friction-system-sub |

---

## License

Personal project. Not for redistribution.

