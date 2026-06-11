[README.md](https://github.com/user-attachments/files/28852922/README.md)

# APEX360 — AI-Powered Commercial Intelligence Platform

> **A fully deployable enterprise sales command center for medical device and healthcare technology companies — built on AWS, powered by Claude AI, and designed for Area VPs managing distributed sales teams.**

![AWS](https://img.shields.io/badge/AWS-Serverless-orange) ![Bedrock](https://img.shields.io/badge/Amazon-Bedrock-purple) ![Claude](https://img.shields.io/badge/Anthropic-Claude-blue) ![Python](https://img.shields.io/badge/Python-3.11-green) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## What Is APEX360?

APEX360 is a commercial intelligence platform prototype built as a single deployable artifact. It demonstrates what a modern GTM engineering stack looks like when you combine:

- A serverless AWS backend (S3, Lambda, API Gateway, Cognito)
- A structured sales data lake (13 CSVs → production-ready Aurora schema)
- Three AI agents powered by Amazon Bedrock Claude
- A signal-based SQL scoring engine that surfaces opportunities from existing data
- A single-file HTML dashboard with 20+ pages covering every dimension of commercial performance

The platform is built around a fictional medical device company called **VitalEdge** — a surgical robotics and AI imaging company competing against Intuitive Surgical, Stryker, and Medtronic. The architecture, data model, and GTM logic are directly portable to any enterprise medtech, SaaS, or capital equipment commercial organization.

---

## What It Does

### Command Center
The landing page. Shows real-time pipeline health, at-risk deal alerts, Q3 forecast vs. target, activity recommendations, and recent win/loss results — all in one view.

### Pipeline & Forecast
Full deal-level pipeline visibility with stage, probability, weighted amount, LAER stage, GPO compliance flag, and risk level. AVP view spans all 15 territories. Territory filter scopes to individual reps. Forecast page shows commit vs. best case vs. pipeline with a waterfall chart.

### Territory Planning
Interactive US map — click any state to select a territory, see the rep, pipeline vs. OP target, deal count, and attainment. Territory list view shows all 15 reps ranked by attainment with sustainability scores.

### Rep Funnel & Coaching
Per-rep pipeline funnel showing stage distribution, average days per stage, close rate, and YTD win/loss. Rep Coaching page has a 9-category 1-5 rubric (Prospecting Cadence, Discovery Questioning, Solution Positioning, Competitive Handling, Pipeline Accuracy, CRM Discipline, Executive Engagement, Clinical Acumen, Coachability) with unique scores, manager op notes, and field observation notes for all 15 reps. Select any rep from the dropdown or click any row in the Team Performance table — both update the rubric instantly.

### GTM Intelligence (The GTM Engineer Layer)
Three pages that demonstrate a signal-based SQL generation engine:

**Opportunity Engine** — Scores every account in the installed base against five GTM signals and surfaces ranked SQLs:
- EOL Displacement (+40 pts) — competitor system hitting end-of-life, customer must decide
- Whitespace Gap (+25 pts per $1M) — existing account with unexpanded departments
- Champion Signal (+30 pts) — warm contact with no active opportunity
- Budget-Loss Reopen (+35 pts) — lost to budget freeze, new FY window open
- High Definitive Score (+20 pts) — high-value unworked prospect
- GPO Membership (+10 pts) — Vizient/Premier compliance removes pricing friction

**SQL Generator** — Visual funnel showing how raw accounts flow through signal scoring into qualified pipeline. Shows the CRM write-back architecture (EventBridge → Lambda → Salesforce Connected App → SNS rep notification).

**MQL Pipeline** — Shows how intent signals from Bombora, HubSpot, LinkedIn Lead Gen, and G2 Buyer Intent layer on top of the existing data to create a full MQL→SQL handoff workflow.

### Customer Intelligence
Account profiles with installed base summary — total systems, competitive installs, EOL systems, annual service value. Device-level inventory joined with model data showing displacement priority and contract expiry.

### Deal Review AI / Sales Plays
Structured sales play library with RACI matrices, competitive counter-messaging playbooks, and a bundled pricing configurator that calculates TCV at different service tier and term combinations.

### Win/Loss Program
85 win/loss records with customer verbatims, competitor analysis, exec sponsor flag, GPO pricing flag, and incumbent loss flag. Filter by outcome, competitor, or territory.

### AI Agents (Three Specialized Claude Instances)
- **Product Expert** — Surgical robotics, clinical evidence, competitive positioning vs. Intuitive/Stryker/Medtronic, APEX AI guidance module differentiators
- **Commercial Agent** — GPO pricing, contract structures, discount approval levels, 5-year TCO models, RaaS vs. Capital+Sub financial modeling
- **Travel Agent** — Territory-optimized itinerary generation, account prioritization, deal-stage-aware scheduling

All three agents support multi-turn conversation within a session and are backed by Amazon Bedrock with configurable knowledge bases.

### Knowledge Base & Document Repository
File browser for the S3 KB documents bucket. Upload a PDF and the auto-sync Lambda triggers a Bedrock ingestion job within 30 seconds — agents answer from the document within 5 minutes.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (S3 Static)                      │
│         Single HTML file — 20+ pages, zero build step       │
│         D3 US map · Chart.js · Responsive · iOS-compatible  │
└────────────────────────┬────────────────────────────────────┘
                         │ fetch() REST calls
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  API GATEWAY (REST)                          │
│   GET  /territories    GET  /pipeline                        │
│   GET  /accounts       GET  /installed-base                  │
│   GET  /win-loss       POST /agent/{type}                    │
└────────────────────────┬────────────────────────────────────┘
                         │ Lambda proxy integration
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  LAMBDA FUNCTIONS (6)                        │
│   apex360-territories    apex360-pipeline                    │
│   apex360-accounts       apex360-installed-base              │
│   apex360-win-loss       apex360-agent                       │
│   apex360-kb-sync  (S3 trigger → Bedrock ingestion)         │
└──────────┬──────────────────────────────┬───────────────────┘
           │ read CSV                     │ invoke_model
           ▼                              ▼
┌──────────────────────┐    ┌────────────────────────────────┐
│   S3 DATA LAKE       │    │   AMAZON BEDROCK               │
│   13 CSVs organized  │    │   Claude (3 agent types)       │
│   by domain folder   │    │   Knowledge Bases (optional)   │
│                      │    │   OpenSearch vector store      │
└──────────────────────┘    └────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                  COGNITO (Optional)                          │
│   User pool · JWT tokens · Territory-scoped claims          │
│   15 rep accounts + 1 KAM + 1 AVP                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Model

13 CSV files matching a production Aurora PostgreSQL schema:

| File | Rows | Description |
|---|---|---|
| `territories.csv` | 15 | Territory definitions, rep assignments, pipeline vs. OP target |
| `accounts.csv` | 55 | Health system accounts with IDN type, EHR, beds, GPO, whitespace |
| `contacts.csv` | ~290 | Stakeholders with buying role, sentiment, relationship score |
| `opportunities.csv` | 97 | Deals with stage, probability, LAER, GPO flag, risk flags (JSON) |
| `installed_base.csv` | ~360 | Device inventory with age, status, displacement priority |
| `device_models.csv` | 18 | Device catalog with compatibility flag and EOL dates |
| `contracts.csv` | 55 | Service contracts with tier, SLA, auto-renew, discount level |
| `win_loss.csv` | 85 | Win/loss records with customer verbatims and competitive flags |
| `activities.csv` | 220 | Calls, demos, site visits linked to accounts and opportunities |
| `def_market_intelligence.csv` | 55 | Procedure volumes, financials, EHR vendor per account |
| `gpo_contracts.csv` | 7 | GPO pricing ceilings (Vizient, Premier, HealthTrust, etc.) |
| `product_catalog.csv` | 14 | Commercial models with list price, subscription, per-use pricing |
| `discount_matrix.csv` | 5 | Discount approval levels by role |

---

## Deployment

### Prerequisites
- AWS account
- Python 3.8+
- `pip install boto3 jupyter`
- AWS credentials configured (`aws configure`) or SageMaker notebook instance

### Recommended: SageMaker (No credential setup needed)

```
AWS Console → SageMaker → Notebook Instances → Create
  Name:     apex360-deploy
  Instance: ml.t3.medium
  IAM Role: Create new → AdministratorAccess
→ Open JupyterLab → upload files → select conda_python3 kernel
```

### File Layout

```
your-deploy-folder/
  APEX360_VitalEdge_Deploy.ipynb    ← deployment notebook
  apex360_vitaledge.html            ← dashboard prototype
  csvs/
    territories.csv
    accounts.csv
    contacts.csv
    opportunities.csv
    installed_base.csv
    device_models.csv
    contracts.csv
    win_loss.csv
    activities.csv
    def_market_intelligence.csv
    gpo_contracts.csv
    product_catalog.csv
    discount_matrix.csv
```

### Run Order

Edit **Cell 0** — set your S3 bucket suffix and email. Then run cells in order:

| Cell | Action | Time |
|---|---|---|
| 0 | Configuration | instant |
| 1 | IAM role | 15 sec |
| 2 | S3 buckets | 10 sec |
| 3 | Upload CSVs | 30 sec |
| 4 | Deploy Lambda functions | 1 min |
| 5 | Create API Gateway | 1 min |
| 6 | Cognito users (optional) | 1 min |
| 7 | Patch HTML with live API URL | instant |
| 8 | Upload dashboard to S3 | instant — **prints your live URL** |
| 9 | Bedrock Knowledge Bases | **skip** — $90/month, add later |
| 10 | Smoke test all endpoints | 30 sec |
| 11 | Summary | instant |
| 12 | Teardown | all commented out |

**Total time: ~12 minutes.** Stop the SageMaker instance immediately after Cell 11.

### Bedrock Model Access

In `us-east-2` and most regions, use cross-region inference prefix:
```
us.anthropic.claude-sonnet-4-5-20250929-v1:0
us.anthropic.claude-haiku-4-5-20251001-v1:0
```

First invocation may require submitting use case details in the Bedrock console. Approval is typically instant for Haiku and Sonnet tiers.

### Estimated Monthly Cost

| Service | Cost |
|---|---|
| S3 (3 buckets, ~2MB data) | ~$0.01 |
| Lambda (light traffic) | ~$0.00 |
| API Gateway | ~$1.00 |
| Bedrock Claude (agent calls) | ~$15.00 |
| Cognito (under 50K MAU) | ~$0.00 |
| **Total without KBs** | **~$16/month** |
| OpenSearch Serverless (KBs) | +~$90/month |

---

## Connecting Real Data — Production Roadmap

This section outlines the path from CSV prototype to a production system with live data flows. It is intentionally directional — the right implementation details depend on your CRM configuration, data governance requirements, and engineering team.

### Phase 1 — Salesforce Bi-Directional Sync

The CSV files are a stand-in for what should be a live Salesforce connection. In production:

**Reading from Salesforce:**
Replace the S3 CSV reads in each Lambda with Salesforce REST API calls using a Connected App with OAuth 2.0. Each Lambda would query the relevant Salesforce object (Opportunity, Account, Contact, Task) filtered by the rep's territory. An EventBridge scheduled rule runs a sync Lambda every 4 hours that reads from Salesforce and writes updated CSVs (or directly into Aurora) so the dashboard always reflects current CRM data.

The key Salesforce objects to map to the existing CSV schemas:
- `Opportunity` → `opportunities.csv` columns
- `Account` → `accounts.csv` columns
- `Contact` → `contacts.csv` columns
- `Task/Event` → `activities.csv` columns
- Custom objects for installed base, contracts, win/loss debrief

**Writing back to Salesforce:**
The GTM Opportunity Engine's "Create SQL" button is designed to POST a new Opportunity record to Salesforce via `/services/data/v58.0/sobjects/Opportunity`. The rep notification flow (Lambda → SNS → email/Slack) should trigger on that write-back. Custom Salesforce fields for GTM score, signal type, and source system allow you to track which opportunities the engine generated.

**Authentication pattern:**
Store the Connected App consumer key, consumer secret, and refresh token in AWS Secrets Manager. The sync Lambda retrieves credentials at runtime — never hardcoded, never in environment variables, fully rotatable.

### Phase 2 — Aurora PostgreSQL (Replace CSV Data Lake)

The CSV structure maps directly to a relational schema. The migration path is:

1. Provision an Aurora PostgreSQL Serverless v2 cluster
2. Run the DDL to create tables matching the CSV column names exactly — the Lambda code doesn't need to change, just swap `read_csv()` for a `psycopg2` query
3. Add an RDS Proxy between Lambda and Aurora to handle connection pooling at scale
4. Load the CSV data as seed data via `COPY` command
5. Point the Salesforce sync Lambda to upsert into Aurora instead of writing CSVs

Aurora Serverless v2 scales to zero when idle (important for a prototype) and handles millions of rows with sub-100ms queries. At enterprise scale with thousands of opportunities and daily syncs, Aurora is the right data layer. The CSV approach in this prototype is intentional — it keeps the deployment self-contained and lets you run the smoke test without a database.

### Phase 3 — React Frontend (Replace Single-File HTML)

The single HTML file was chosen for portability — it runs locally, deploys to S3 in one upload, and needs no build toolchain. For a production system with a real engineering team, the right move is a React application.

**What to think about when migrating:**

The existing HTML dashboard is organized around a `showPage(id)` function that swaps visible sections. This maps naturally to React Router — each "page" becomes a route, each major card becomes a component, and the global data arrays (`pipelineDeals`, `TERRITORY_DATA`, etc.) become a Redux store or React Context.

The API calls are already clean `fetch()` calls returning JSON — they drop directly into `useEffect` hooks with no modification. The Lambda functions don't need to change at all.

The D3 US map, Chart.js charts, and the rubric scoring components are the most complex pieces to convert — plan those last. Everything else (tables, cards, KPI tiles, nav) is straightforward JSX.

**Deployment pattern:**
Build the React app with `npm run build`, upload the `build/` folder to the S3 app bucket, put CloudFront in front for HTTPS and global CDN, and wire a custom domain via Route 53. This gives you `https://apex360.yourcompany.com` with sub-second load times globally.

**Authentication:**
Replace the raw Cognito user pool with AWS Amplify Auth — it handles the JWT token lifecycle, refresh, and storage automatically. The territory claim in the JWT flows through to the API Gateway authorizer and Lambda unchanged.

### Phase 4 — Knowledge Bases with Real Documents

Once the agents are working from system prompts, add knowledge bases to ground them in company-specific documents:

- Product spec sheets and clinical evidence → `surgical/` KB prefix
- GPO contract PDFs and pricing guides → `pricing/` KB prefix
- Territory coverage guidelines → `travel/` KB prefix

Upload any PDF to the S3 KB bucket and the auto-sync Lambda (Cell 10) triggers a Bedrock ingestion job automatically. The agent then answers with citations from the actual document.

---

## Repository Structure

```
apex360-vitaledge/
  README.md                         ← this file
  APEX360_VitalEdge_Deploy.ipynb    ← self-deploying AWS notebook
  apex360_vitaledge.html            ← dashboard prototype (single file)
  csvs/
    territories.csv
    accounts.csv
    contacts.csv
    opportunities.csv
    installed_base.csv
    device_models.csv
    contracts.csv
    win_loss.csv
    activities.csv
    def_market_intelligence.csv
    gpo_contracts.csv
    product_catalog.csv
    discount_matrix.csv
```

---

## What This Demonstrates

For anyone evaluating this repository as a portfolio piece or starting point:

**GTM Engineering** — The Opportunity Engine is a working signal-based SQL scoring system. It reads from the data layer, applies weighted scoring across five signal types, filters against active pipeline, and outputs a ranked, territory-assigned list of SQLs with recommended plays. The CRM write-back architecture is documented and the Lambda hook is built.

**Commercial Operations** — The data model covers every dimension of enterprise commercial performance: pipeline health, forecast accuracy, territory sustainability, rep development, installed base displacement, contract lifecycle, win/loss analysis, and GTM signal processing. This is what a VP of Sales Operations builds — not a reporting dashboard, but an operating system for a commercial team.

**AWS Architecture** — The deployment notebook provisions a complete serverless stack in ~12 minutes from a single file. Every AWS service used (S3, Lambda, API Gateway, Cognito, Bedrock, OpenSearch, EventBridge, SNS) is production-grade, not prototype-grade. The teardown cell cleans up everything with no orphaned resources.

**AI Integration** — Three specialized Claude agents with domain-specific system prompts, multi-turn conversation history, and a knowledge base architecture that scales from zero documents to hundreds via S3 trigger automation.

---

## License

MIT License — use freely, attribution appreciated.

---

## Contributing

Issues and PRs welcome. Key areas where contributions add value:

- Salesforce Connected App sync Lambda implementation
- Aurora PostgreSQL DDL and migration scripts
- React component library starter
- Additional GTM signal types for the Opportunity Engine
- Industry-specific variants (SaaS, diagnostics, life sciences, industrial)
