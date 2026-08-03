<div align="center">

# AI Systems Portfolio

**Nine production AI systems across six industries — what they do, how they are
built, and the decisions that turned out to matter.**

[![Systems](https://img.shields.io/badge/systems-9-1f6feb?style=flat-square)](#the-systems)
[![Live](https://img.shields.io/badge/live%20in%20production-5-2ea043?style=flat-square)](#the-systems)
[![Patents](https://img.shields.io/badge/provisional%20patents-5-8957e5?style=flat-square)](#premiumradar)
[![Website](https://img.shields.io/badge/sivakumar.ai-Visit-0A66C2?style=flat-square&logo=googlechrome&logoColor=white)](https://sivakumar.ai)

</div>

---

> 🔒 **No source code here.** These are commercial, client-facing products, so
> the repositories are private and product links are omitted. What is public is
> the architecture and the reasoning — which is the part worth reading anyway.
> Case studies and demos on request, or at [sivakumar.ai](https://sivakumar.ai).

Three related repositories, doing different jobs:

| | |
|---|---|
| **This repo** | *What I built* — system case studies |
| [**ai-architecture-notes**](https://github.com/uaesivakumar/ai-architecture-notes) | *How I think* — design decisions with the trade-offs left in |
| [**llm-gateway**](https://github.com/uaesivakumar/llm-gateway) | *Code you can run* — the LLM routing layer, open sourced |

---

## The systems

| System | Domain | What it does | Status |
|---|---|---|---|
| [**PremiumRadar**](#premiumradar) | B2B sales intelligence | Governed intelligence platform for regulated industries | 🤝 Design-partner stage |
| [**Inteller**](#inteller) | Talent intelligence | Ghost-job detection and resume tailoring across 30+ job platforms | 🟢 Live |
| [**SKC Digital**](#skc-digital) | Consulting / decision support | Executive simulation built on 30+ documented AI programme failures | 🟢 Live |
| [**RM Assistant**](#rm-assistant) | Financial services | Relationship-management copilot with structured memory | 🟢 Live |
| [**AI Leads Portal**](#ai-leads-portal) | Sales intelligence | Conversational intent capture with self-learning scoring | 🟢 Live |
| [**Coach**](#coach) | HR / performance | Structured performance evaluation with constrained LLM output | 🛠️ Architecture |
| [**Arsha LMS**](#smaller-systems) | EdTech | Multi-tenant learning platform | 🛠️ In design |
| [**Chunav**](#smaller-systems) | Civic analytics | Election analytics and visualisation | 🛠️ In design |
| [**Payroll Enrichment**](#smaller-systems) | Data engineering | ADGM registry extraction and enrichment pipeline | 🛠️ In build |

---

## PremiumRadar

**Governed B2B sales intelligence for regulated industries.**

The largest of the nine and the one the other architectures borrowed from.
Aggregates company data, enriches it with LLM reasoning, and surfaces
opportunities to sales teams — with every inference traceable back to its
evidence, because the buyers are institutions that have to explain their
decisions.

**Scale:** 2,600+ source files across 4 repositories · 38+ services · 80+
database tables over 87 migrations.

**Architecture worth noting**

- **An 11-layer discovery pipeline** for entity resolution and enrichment.
  Identity is resolved *before* retrieval, so downstream search is scoped by
  canonical ID rather than by a name string — the difference between precision
  and confident nonsense. → [Hybrid retrieval: what vectors miss](https://github.com/uaesivakumar/ai-architecture-notes/blob/main/notes/03-hybrid-retrieval-what-vectors-miss.md)
- **A three-tier tool architecture**, separating what the model may propose from
  what the system may execute. The reasoning layer holds no credential capable
  of changing state. → [The reasoning layer holds no credential](https://github.com/uaesivakumar/ai-architecture-notes/blob/main/notes/01-credentials-and-the-reasoning-layer.md)
- **Deterministic state machines with six decision gates.** The model produces
  evidence; the gates produce the verdict, in code, reproducibly. → [Deterministic gates around a probabilistic core](https://github.com/uaesivakumar/ai-architecture-notes/blob/main/notes/02-deterministic-gates-around-a-probabilistic-core.md)
- **Multi-provider LLM abstraction** with failover and per-request cost
  attribution — the pattern later extracted and open sourced as
  [llm-gateway](https://github.com/uaesivakumar/llm-gateway).

**Five provisional patents** filed on the novel parts.

*Stack:* Python · pgvector · Neo4j · multi-provider LLM · Cloud Run · GCP

---

## Inteller

**Ghost-job detection and resume tailoring.**

Scores job listings across 30+ platforms for whether the role is genuinely open —
a signal candidates currently have no way to see — and tailors applications to
the ones that are. Consumer-facing, subscription-billed, which makes unit
economics a design constraint rather than a finance detail: every scoring pass
has to be worth its cost per user.

*Stack:* Next.js · LLM scoring · Stripe · scheduled ingestion

---

## SKC Digital

**Executive simulation for AI decision-making.**

Built on a corpus of 30+ documented AI programme failures. Executives work
through scenarios and see where the decision actually went wrong — which is
almost never the model. → [Why AI programmes fail](https://github.com/uaesivakumar/ai-architecture-notes/blob/main/notes/04-why-ai-programmes-fail.md)

*Stack:* Next.js · Vertex AI (Gemini) · PostgreSQL + pgvector · Cloud Run

---

## RM Assistant

**Relationship-management copilot for bankers.**

Structured memory across conversations, so context persists between sessions
without stuffing an ever-growing transcript into every prompt. The interesting
constraint is deciding what to persist and what to deliberately forget — a
summary that keeps everything is just a slower transcript.

*Stack:* FastAPI · Claude · Firestore

---

## AI Leads Portal

**Conversational intent capture and pipeline scoring.**

Built for on-site sales events. Captures visitor intent through conversation
rather than a form, scores the lead, and routes it. Time-to-recommendation went
from roughly two minutes to under 45 seconds.

The scoring engine improves in three stages: an initial LLM assessment, feedback
collected from what actually happened commercially, and prompt refinement driven
by those historical patterns. No training loop, no labelled dataset — the
business outcome *is* the label.

*Stack:* Node.js · OpenAI · Firestore · service workers for offline capture

---

## Coach

**Structured performance evaluation.**

Generates written performance assessments from structured inputs. The lesson
here was narrow and useful: LLM output quality was governed almost entirely by
how tightly the prompt constrained the shape of the answer. Loosening the
constraints did not produce more insight, only more words.

*Stack:* Python · Flask microservices · Firestore · OpenAI · Cloud Run

---

## Smaller systems

**Arsha LMS** — multi-tenant learning platform with per-tenant isolation.
*FastAPI · Firestore · Gemini*

**Chunav** — election analytics and visualisation.
*Next.js · D3 · GCP*

**Payroll Enrichment** — extraction and enrichment of company records from the
ADGM registry. The recurring lesson of every data pipeline: cleaning is most of
the work, and the interesting engineering is in reconciliation, not extraction.
*Python · BeautifulSoup · Pandas*

---

## What carries across all nine

Patterns that turned out to be portable — none of them are about model choice:

- **Governance designed in, not retrofitted.** Auditability is an architectural
  constraint. Added at the end, it means a rebuild.
- **The model produces evidence; code produces decisions.** Reproducibility is
  not optional in regulated domains, and models are not reproducible.
- **Identity before retrieval.** Resolve the entity, then search.
- **Cost as a design input.** Unit economics decide what architecture is even
  available to you.
- **Failure modes over happy paths.** The demo is the easy 20%.

Written up properly in
[**ai-architecture-notes**](https://github.com/uaesivakumar/ai-architecture-notes).

---

## Stack across the portfolio

**AI/LLM** Claude · GPT · Gemini · Vertex AI · pgvector · Neo4j
**Backend** Python · FastAPI · Flask · Node.js
**Frontend** Next.js · React · TypeScript · D3
**Data** PostgreSQL · Firestore · Redis · Pandas
**Infra** GCP · Cloud Run · Docker · CI/CD

---

## Background

20 years: 8 in technology delivery across 100+ client projects, then 12 in
banking, where I was recognised as a top 1% performer. The AI work sits at the
intersection — I have been the builder, the end user, and the person accountable
for the number.

**Open to** AI solutions architecture, AI/ML engineering, LLM platform
engineering and technical leadership — enterprise SaaS, fintech, government,
consulting or product. Abu Dhabi / UAE, and remote.

📬 [siva@sivakumar.ai](mailto:siva@sivakumar.ai) · [sivakumar.ai](https://sivakumar.ai) · [LinkedIn](https://linkedin.com/in/uaesivakumar)
