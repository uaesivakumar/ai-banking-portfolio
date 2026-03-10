# AI + Banking Portfolio — Sivakumar Chandrasekaran

**20 years of experience: 8 years in technology delivery, 12 years in banking sales. Now building AI systems for financial services.**

> These projects are in private repositories. This document provides architecture overviews and technical decisions without exposing proprietary code.

LinkedIn: [linkedin.com/in/uaesivakumar](https://www.linkedin.com/in/uaesivakumar/) | Web: [sivakumar.ai](https://sivakumar.ai) | Email: siva@sivakumar.ai

---

## Table of Contents

1. [Premier Radar — Enterprise B2B Sales Intelligence](#1-premier-radar--enterprise-b2b-sales-intelligence-platform)
2. [AI Leads Portal — Event Lead Capture](#2-ai-leads-portal--event-lead-capture--scoring)
3. [SKC Digital — AI Consulting Platform](#3-skc-digital--ai-consulting-platform)
4. [Coach — Performance Evaluation System](#4-coach--performance-evaluation-system)
5. [PAYROLL — Data Enrichment Pipeline](#5-payroll--data-enrichment-pipeline)
6. [Background](#background)

---

## 1. Premier Radar — Enterprise B2B Sales Intelligence Platform

The largest project in this portfolio. A multi-service platform that aggregates public business data, enriches it with LLM analysis, and surfaces actionable sales intelligence for banking relationship managers.

**Scale:** 2,676+ source files | 4 repositories | 38+ specialized services | 87 database migrations | 80+ tables | Months of development alongside a full-time job

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENTS                                  │
│              Next.js 14 / React 18 / TypeScript                 │
│                    Tailwind CSS                                  │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                     API GATEWAY                                  │
│                Node.js / Express.js                               │
│         Authentication · Rate Limiting · Routing                 │
└────┬─────────────────┬──────────────────────┬────────────────────┘
     │                 │                      │
     ▼                 ▼                      ▼
┌──────────┐   ┌──────────────┐   ┌─────────────────────┐
│ Service 1│   │  Service 2   │   │     Service 3       │
│ Data     │   │  Intelligence│   │     Enrichment      │
│ Ingestion│   │  & Analysis  │   │     & Scoring       │
└────┬─────┘   └──────┬───────┘   └──────────┬──────────┘
     │                │                      │
     │                ▼                      │
     │   ┌─────────────────────┐             │
     │   │   LLM Gateway       │             │
     │   │                     │             │
     │   │  ┌───────────────┐  │             │
     │   │  │ OpenAI GPT    │  │             │
     │   │  ├───────────────┤  │             │
     │   │  │ Anthropic     │  │             │
     │   │  │ Claude        │  │             │
     │   │  ├───────────────┤  │             │
     │   │  │ Google Vertex │  │             │
     │   │  │ AI (Gemini)   │  │             │
     │   │  └───────────────┘  │             │
     │   └─────────────────────┘             │
     │                                       │
     ▼                                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                      DATA LAYER                                  │
│                                                                  │
│   ┌────────────────────┐         ┌──────────────┐               │
│   │   PostgreSQL       │         │    Redis      │               │
│   │   + pgvector       │         │    Cache      │               │
│   │   (Cloud SQL)      │         │               │               │
│   │                    │         │               │               │
│   │   92 migrations    │         │               │               │
│   └────────────────────┘         └──────────────┘               │
└──────────────────────────────────────────────────────────────────┘

Deployment: GCP Cloud Run · Docker · Pub/Sub · Secret Manager
```

### System Scope (Full Platform)

The simplified diagram above shows the core flow. The full platform comprises:

- **SaaS Layer** (Next.js 14): 1,690 files, 392 app routes, Prisma ORM, Stripe billing, multi-tenant auth
- **Intelligence Engine** (Node.js/Express): 986 files, 38+ services, 51 OS-layer modules, multi-agent orchestration
- **Worker Service**: Async background processing on GCP Cloud Run
- **Infrastructure**: Terraform on GCP (Cloud Run, Cloud SQL, Pub/Sub, Secret Manager, BigQuery)

Key subsystems: 11-layer discovery pipeline with multi-agent retrieval (Brave + Firecrawl + Jina), SIVA AI brain with 3-tier tool architecture (Foundation/Strict/Delegated), persona-as-policy governance, QTLE composite scoring, deterministic outreach state machine, event-sourced audit trails, and 6 hard gates enforcing invariants at every decision point.

5 provisional patent applications filed for novel architectures.

### Key Technical Decisions

**pgvector over Pinecone for vector search**

Pinecone is purpose-built for vector similarity search, but using it would have meant managing a separate service, separate billing, and network latency between Cloud SQL and an external vector DB. pgvector keeps embeddings in the same PostgreSQL instance as the relational data. Queries that combine vector similarity with structured filters (e.g., "find companies similar to X in this industry with revenue above Y") run as a single SQL query instead of requiring application-level joins across two data stores. For this scale, pgvector's performance is sufficient and the operational simplicity is worth it.

**Multi-provider LLM gateway instead of single provider**

Different tasks have different cost/quality tradeoffs. Summarization can use a cheaper model. Complex reasoning needs Claude or GPT-4. Having an abstraction layer means swapping providers requires changing a config, not rewriting calling code. It also provides resilience — if one provider has an outage, traffic reroutes.

**Cloud Run over GKE (Kubernetes)**

GKE is the right choice when you need persistent workloads, complex networking, or fine-grained resource control. This platform has bursty traffic patterns and the services are stateless. Cloud Run scales to zero when idle (critical for a side project budget), handles container orchestration automatically, and deploys in seconds. The tradeoff is less control over networking and no persistent connections, but for this use case that is acceptable.

**92 database migrations instead of wiping and recreating**

Every schema change is a numbered, reversible migration. This is slower during development but means the database schema has a complete, auditable history. Any environment can be rebuilt from scratch by running migrations in order. This discipline paid off multiple times when debugging data issues — being able to trace exactly when a column was added or a constraint changed.

### Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | Next.js 14, React 18, TypeScript | SSR for performance, type safety across the stack |
| Styling | Tailwind CSS | Utility-first, fast iteration, no CSS naming debates |
| API | Node.js, Express.js | Same language as frontend, good ecosystem for REST APIs |
| Database | PostgreSQL + pgvector | Relational + vector search in one engine |
| Cache | Redis | Session management, rate limiting, hot data caching |
| LLMs | OpenAI GPT, Anthropic Claude, Google Vertex AI | Multi-provider for cost optimization and resilience |
| Infra | GCP Cloud Run, Cloud SQL, Docker | Serverless containers, managed database, zero-scale |
| Messaging | GCP Pub/Sub | Async job processing, service decoupling |
| Secrets | GCP Secret Manager | Centralized credential management |

### What I Learned

- **Schema migrations are an investment that compounds.** At 92 migrations, the discipline of never manually altering a database pays for itself in debugging time saved and deployment confidence.
- **Multi-provider LLM abstraction is not premature.** I initially thought one provider would be enough. Within weeks I needed to switch between models for cost reasons. The gateway abstraction, built early, saved significant refactoring.
- **Cloud Run's cold start penalty is real but manageable.** Keeping minimum instances at 1 for the primary service and letting secondary services scale to zero was the right balance between cost and latency.
- **Building a large system with AI-assisted development (Claude Code) requires its own skill set.** Context window management, session planning, knowing when to let the AI scaffold vs. when to write manually, preventing state drift across long sessions — these are real engineering skills, not "just prompting."

---

## 2. AI Leads Portal — Event Lead Capture & Scoring

A lead capture system built for bank expo events. Relationship managers scan or enter lead details on-site, and the system scores and enriches each lead using LLM analysis.

### Architecture

```
┌──────────────────┐     ┌──────────────────────────────────┐
│  Mobile/Tablet   │     │         Admin Dashboard          │
│  Lead Capture    │────▶│     Lead Review & Analytics      │
│  (Event Floor)   │     │                                  │
└────────┬─────────┘     └──────────────┬───────────────────┘
         │                              │
         ▼                              ▼
┌──────────────────────────────────────────────────────────────┐
│                    Node.js / Express.js                       │
│                                                              │
│   ┌─────────────┐   ┌──────────────────┐   ┌─────────────┐ │
│   │ Lead        │   │ Scoring Engine   │   │ Enrichment  │ │
│   │ Ingestion   │   │ (Self-Learning)  │   │ Service     │ │
│   └─────────────┘   └────────┬─────────┘   └─────────────┘ │
│                              │                               │
│                              ▼                               │
│                     ┌─────────────────┐                      │
│                     │   OpenAI GPT    │                      │
│                     └─────────────────┘                      │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │    Firestore    │
                     └─────────────────┘
```

### How the Scoring Engine Works (Conceptual)

The scoring engine is not a static rules system. It works in three stages:

1. **Initial scoring** — LLM analyzes the raw lead data (company name, stated interest, interaction notes from the RM) and produces a structured score across dimensions like urgency, product fit, and estimated wallet size.

2. **Feedback loop** — When an RM marks a lead as converted or lost, that outcome is stored alongside the original lead data and score. Over time, the system accumulates a history of what "good" and "bad" leads looked like.

3. **Prompt refinement** — The scoring prompt incorporates patterns from historical outcomes. It is not fine-tuning the model — it is adjusting the context and examples provided to the LLM so that scoring improves as more data flows through the system.

This approach avoids the cost and complexity of model fine-tuning while still improving over time.

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Node.js, Express.js |
| LLM | OpenAI GPT |
| Database | Firestore |
| Deployment | GCP Cloud Run, Docker |

### What I Learned

- **Firestore is the right choice when the schema is evolving rapidly.** During expo events, the fields captured changed from event to event. A document database made this painless compared to running migrations for every field change.
- **LLM-based scoring outperforms rule-based scoring for unstructured input.** Relationship managers enter free-text notes. Rules break on messy text. LLMs handle it naturally.
- **The self-learning loop does not require fine-tuning.** Few-shot examples from historical outcomes, injected into the prompt, produced measurable scoring improvement without any model training infrastructure.

---

## 3. SKC Digital — AI Consulting Platform

**Live at [skc.digital](https://skc.digital)**

A platform for delivering AI consulting services — strategic briefs, judgment frameworks, and learning resources for banking and fintech professionals.

### What It Does

- AI-assisted strategic analysis and brief generation
- Consulting service showcase and client intake
- Content delivery for banking + AI intersection topics

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js, TypeScript, Tailwind CSS |
| AI | Google Vertex AI (Gemini) |
| Database | PostgreSQL |
| Deployment | GCP Cloud Run, Docker |

### What I Learned

- **Vertex AI's Gemini models are competitive for structured output tasks.** For generating formatted briefs and analyses, Gemini performs well and the GCP-native integration reduces auth complexity when already on GCP.
- **A consulting platform needs to demonstrate judgment, not just technology.** The hardest part was not the code — it was designing outputs that show genuine strategic thinking rather than generic LLM summaries.

---

## 4. Coach — Performance Evaluation System

A microservices-based system for performance evaluation. Managers input evaluation criteria and employee data; the system generates structured assessments with actionable feedback.

### Architecture

```
┌──────────────────┐
│   Web Frontend   │
└────────┬─────────┘
         │
         ▼
┌──────────────────────────────────────────────────┐
│              API Layer (Flask)                    │
│                                                  │
│   ┌──────────────┐    ┌───────────────────────┐  │
│   │ Evaluation   │    │ Feedback Generation   │  │
│   │ Service      │    │ Service (LLM)         │  │
│   └──────┬───────┘    └───────────┬───────────┘  │
│          │                        │               │
│          ▼                        ▼               │
│   ┌──────────────┐    ┌───────────────────────┐  │
│   │  Firestore   │    │   OpenAI GPT          │  │
│   └──────────────┘    └───────────────────────┘  │
└──────────────────────────────────────────────────┘

Deployment: GCP Cloud Run · Docker
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python, Flask |
| Database | Firestore |
| LLM | OpenAI GPT |
| Deployment | GCP Cloud Run, Docker |

### What I Learned

- **Flask is still a good choice for small, focused services.** Not everything needs a full framework. Flask's simplicity made it easy to build, test, and deploy individual microservices quickly.
- **LLM-generated evaluations require careful prompt design to avoid generic output.** The difference between useful and useless feedback comes down to how specifically you constrain the LLM's output format and require it to reference concrete data points.

---

## 5. PAYROLL — Data Enrichment Pipeline

A data pipeline that scrapes, cleans, and enriches company data from the Abu Dhabi Global Market (ADGM) registry. Used for prospecting and market analysis.

### Architecture

```
┌───────────────────┐     ┌──────────────────┐     ┌──────────────┐
│  ADGM Public      │     │   Scraping &     │     │  Enriched    │
│  Registry         │────▶│   Cleaning       │────▶│  Dataset     │
│  (Web Pages)      │     │   Pipeline       │     │  (CSV/Excel) │
└───────────────────┘     └──────────────────┘     └──────────────┘
                                  │
                          ┌───────┴────────┐
                          │                │
                    ┌─────▼─────┐   ┌──────▼───────┐
                    │ Beautiful │   │   Pandas     │
                    │ Soup      │   │   Transform  │
                    │ (Extract) │   │   & Clean    │
                    └───────────┘   └──────────────┘
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python |
| Scraping | BeautifulSoup |
| Data Processing | Pandas |
| Output | Structured CSV/Excel |

### What I Learned

- **Data cleaning is 80% of the work.** The scraping code was straightforward. Handling inconsistent company names, missing fields, duplicate entries, and encoding issues took far longer than writing the scraper itself.
- **Pandas is good enough for mid-scale data pipelines.** For thousands of records with complex transformations, Pandas is fast to write and fast to run. No need for Spark or distributed computing at this scale.

---

## Background

I am a Senior Retail Banking Officer at Emirates NBD with 12 years in banking sales and 8 years prior experience in technology delivery (B.Tech in IT, built 100+ web projects for clients, handled full lifecycle from acquisition to support).

These projects represent the intersection of those two careers — using modern AI and software engineering to solve real problems in banking and financial services.

All recent projects were built using AI-assisted development (Claude Code), which is a distinct engineering discipline. It requires architecture planning, context management, session orchestration, state management across long builds, and the judgment to know when AI-generated code is correct vs. when it needs human intervention. The tool is AI. The engineering decisions are human.

---

*Last updated: March 2026*
