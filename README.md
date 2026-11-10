<div align="center">

# DocMaps

**A live directory helping patients in Ontario find family doctors who are accepting new patients, kept accurate by AI voice agents.**

[![Live](https://img.shields.io/badge/Live-docmaps.ca-0d1117?style=for-the-badge&logo=vercel&logoColor=white)](https://docmaps.ca)
![Client project](https://img.shields.io/badge/Client%20project-lead%20engineer-0d1117?style=for-the-badge)

![DocMaps on desktop](screenshots/cover.png)

</div>

> This is a showcase repo. DocMaps is a client product and its source code is private, so this page covers what it does, how it's built and what it looks like. Happy to walk through the architecture on a call.

## The problem

Patients in Ontario struggle to find a family doctor who is actually accepting new patients. Public directories go stale the moment a clinic's intake status changes, so people waste days calling clinics that are full.

## What I built

I'm the lead engineer and built the platform end to end: architecture, backend, frontend, database, the AI calling pipeline, deployment and production support.

- **Outbound AI voice campaigns.** Voice agents call clinics on a schedule and ask whether they're accepting patients, which doctors, languages and more
- **LLM transcript review.** Every call is reviewed by an LLM that extracts structured data and returns a verdict: **publish**, **retry** or **escalate**
- **Human review queue.** Uncertain calls go to reviewers, so nothing unverified reaches patients
- **Durable workflows.** Scheduling, waiting for call results, retries and reviews run as event-driven workflows that survive failures
- **Patient-facing map search** with filters by distance, doctor gender and language, plus public clinic pages
- **Admin portal** for campaigns, reviews, call metrics and agent version comparisons
- **Monitoring and alerting** so failures surface in minutes, not days

## Engineering highlights

- **Webhook race conditions.** The call provider sometimes delivered the same webhook twice, and very fast calls finished before the workflow started listening. I made ingestion idempotent, added event deduplication and replaced a long wait with polling plus a stuck-review recovery job, with integration tests reproducing both races. No calls are lost since.
- **Constrained AI output.** Extracted answers are validated and normalised against allowed values per question, so the LLM can't publish free-form data.
- **Verified-only publishing.** Patients only ever see data from a dedicated published view that's refreshed when a call is verified.

## Screenshots

| Map search | Nearby clinics |
| --- | --- |
| ![Map search](screenshots/map-search.png) | ![Nearby clinics](screenshots/nearby-clinics.png) |

![Case study page](screenshots/case-study.png)

## Architecture

```mermaid
flowchart LR
    subgraph Calling pipeline
        C[Campaign scheduler] --> I[Inngest<br/>durable workflows]
        I --> R[Retell<br/>AI voice agent]
        R -- webhook --> API
        I --> L[LLM review<br/>publish / retry / escalate]
    end
    API[FastAPI backend<br/>Python] --> DB[(PostgreSQL<br/>Supabase)]
    L --> DB
    L -- escalate --> H[Human review queue]
    DB --> PV[Published view<br/>verified data only]
    PV --> FE[Next.js frontend<br/>map search + admin]
    FE --> API
    A[Firebase Auth<br/>Google SSO + roles] --> FE
    M[Logfire + Slack alerts] -. monitors .-> API
```

## Tech stack

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat-square&logo=supabase&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase_Auth-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-0080FF?style=flat-square&logo=digitalocean&logoColor=white)
![Pulumi](https://img.shields.io/badge/Pulumi-8A3391?style=flat-square&logo=pulumi&logoColor=white)

Retell for voice agents, Inngest for durable workflows, SQLAlchemy and Alembic, Mapbox, Logfire for observability, testcontainers for integration tests.

## Contact

**Muhammad Izhar** · [izhar.dev](https://izhar.dev) · [LinkedIn](https://linkedin.com/in/izharrr99) · muhammad.izhar11@gmail.com
