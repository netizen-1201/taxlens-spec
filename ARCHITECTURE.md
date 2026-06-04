# TaxLens — System Architecture Summary

> **Purpose:** 2-minute read for recruiters, hiring managers, and technical reviewers.  
> **Full spec:** See [`docs/02_architecture/04_architecture.md`](docs/02_architecture/04_architecture.md)

---

## System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│  CLIENT LAYER                                               │
│  Next.js 14 (TypeScript) — SSR/SPA hybrid, Tailwind CSS     │
└──────────────────────┬────────────────────────────────────────┘
                       │ HTTPS / REST + SSE
┌──────────────────────▼────────────────────────────────────────┐
│  API GATEWAY (NestJS + TypeScript)                            │
│  Auth · Rate Limiting · Validation · RBAC · Consent Guard       │
└────┬──────────┬──────────┬──────────────┬─────────────────────┘
     │          │          │              │
┌────▼────┐ ┌───▼────┐ ┌───▼──────┐ ┌────▼──────────┐
│Document │ │ Tax    │ │ Report   │ │  Audit /      │
│Ingestion│ │Profile │ │Generation│ │  Compliance   │
│ Module  │ │+ Engine│ │ Module   │ │  Module       │
└────┬────┘ └───┬────┘ └───┬──────┘ └────┬──────────┘
     │          │          │              │
┌────▼──────────▼──────────▼──────────────▼──────────┐
│  DATA ACCESS (PostgreSQL 15 + Redis 7 + PgBouncer)  │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  WORKER / ORCHESTRATION (Temporal + Python)         │
│  OCR Vendor SDK · PDF Generator · Virus Scanner     │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│  EXTERNAL SERVICES                                    │
│  AWS S3 (ap-south-1) · KMS · SES · ClamAV · OCR AI │
└───────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Monolith vs Microservices | **Modular monolith** | Clean module boundaries allow extraction later; avoids v1 ops overhead |
| Extraction language | **Python workers** | OCR vendor SDKs, PDF manipulation, and ML libs are Python-native |
| Job queue | **Temporal** (workflows) + **BullMQ/Redis** (fire-and-forget) | Temporal handles durable retries; BullMQ handles lightweight tasks |
| File upload | **Direct S3 pre-signed URL** | Client → S3 bypasses API memory pressure on large PDFs |
| State mutation | **Append-only** (extractions, computations, audit) | Immutability enables audit trails without extra engineering |
| Tax rules | **Database-driven JSON objects** | Update rules without code deploy for mid-year CBDT amendments |
| PII storage | **Field-level AES-256-GCM via AWS KMS** | DPDP Act compliance; limits blast radius on DB breach |
| Data residency | **ap-south-1 (Mumbai)** | Cross-border transfer prohibition under DPDP Act 2023 |

---

## Core Data Flow (Upload → Estimate → PDF)

1. **Upload** — Client requests pre-signed S3 URL → uploads PDF directly → API confirms → virus scan (BullMQ)
2. **Extract** — Temporal W1 triggers W2 → Python worker calls OCR vendor → Form16/AIS/26AS mappers apply regex heuristics → confidence scoring → `NEEDS_REVIEW` or `COMPLETED`
3. **Review** — User corrects fields in side-by-side PDF viewer → locks extraction → triggers computation
4. **Compute** — Temporal W4 assembles inputs → loads active tax rules → slab-wise tax + surcharge + marginal relief + 87A rebate + TDS reconciliation → immutable `ComputationRun` stored
5. **Report** — Temporal W5 assembles result → WeasyPrint renders HTML/CSS → stores PDF in S3 → returns 10-min pre-signed download URL

---

## Module Responsibilities

| Module | Language | Runtime | Key Duty |
|--------|----------|---------|----------|
| Web Frontend | TypeScript | Next.js 14 / Vercel | Upload, review, deduction input, estimate dashboard, admin console |
| API Gateway | TypeScript | NestJS / ECS Fargate | Auth, validation, RBAC, consent gate, job dispatch, CRUD |
| Extraction Worker | Python | FastAPI + Temporal | OCR vendor adapter, field mappers, confidence scorer |
| Report Worker | Python | FastAPI + Temporal | Jinja2 template → WeasyPrint → S3 upload |
| Database | SQL | PostgreSQL 15 + TypeORM | 15+ tables, partitioned audit_events, immutable rows |
| Cache / Queue | — | Redis 7 | BullMQ job queue, session cache, rate-limit counters |
| Orchestrator | — | Temporal Server | W1–W9: ingestion, extraction, review, computation, report, compliance, deletion, retry, maker-checker |

---

## Failure Handling

| Failure | Mitigation |
|---------|-----------|
| OCR vendor API down | Temporal exponential backoff; user notified after timeout |
| Tax engine validation error | Fail fast with field-level details; no partial result stored |
| PDF render OOM | Retry 2×; alert ops if persistent |
| DB unavailable | PgBouncer connection pooling; read replicas for reporting |
| Virus scan timeout | 3 retries → quarantine + ops alert |
| Audit write failure | Temporal W6 retries **indefinitely** up to 24h; compliance-critical |

---

## Compliance Architecture

- **Consent Gate** — 5-case middleware blocks all document/computation/report actions without active consent
- **Audit Events** — Append-only, partitioned by month, retained 7+ years, masked PII in operator views
- **Maker-Checker** — Admin actions (tax rule activation, user deletion, extraction unlock) require second-admin approval via Temporal W9
- **Non-ERI Boundary** — Every UI screen and PDF footer carries "ESTIMATE ONLY — NOT A LEGAL FILING"

---

*For the full architecture document, see [`docs/02_architecture/04_architecture.md`](docs/02_architecture/04_architecture.md).*
