# TaxLens

> **Status:** Specification & Architecture Complete | Implementation Backlog Defined  
> **Scope:** India income-tax filing-support SaaS for salaried ITR-1 filers (AY 2025-26)  
> **Author:** [Sahil Zafar](https://github.com/netizen-1201) — BCom (Hons) Accounting | Data Analytics & Full-Stack Development

---

## What This Is

TaxLens is a full product specification for a regulated fintech SaaS that helps Indian salaried individuals upload tax documents (Form 16, AIS, 26AS), extract structured data via OCR, compute old-vs-new regime tax estimates, and generate CA-ready PDF reports.

This repository contains the complete design-phase output: PRD, system architecture, database schema, API contracts, Temporal workflows, compliance controls (DPDP Act 2023 / IT Act 2000), frontend screen specs, admin console design, and an 83-chunk delivery backlog.

---

## Repository Structure

| Folder | Contents |
|--------|----------|
| `docs/01_product/` | Scope clarification, MVP definition, Product Requirements Document |
| `docs/02_architecture/` | System architecture & stitching reports (consistency checks across artifacts) |
| `docs/03_data/` | PostgreSQL schema design (15+ tables, field-level encryption, partitioning) |
| `docs/04_api/` | REST API contracts (NestJS, 30+ endpoints, JWT auth, rate limiting) |
| `docs/05_backend/` | Temporal workflows, OCR extraction pipelines, tax rules engine |
| `docs/06_compliance/` | DPDP Act controls, consent gate logic, audit taxonomy, non-ERI boundary |
| `docs/07_frontend/` | Next.js screen specs, navigation flows, mobile-first vs desktop-first decisions |
| `docs/08_admin/` | Operations console, RBAC, maker-checker approvals, health monitoring |
| `docs/09_delivery/` | 83 implementation chunks across 15 epics, 10-sprint execution plan |

---

## Key Design Decisions

- **Modular monolith** (NestJS + Next.js + Python workers) — extractable to microservices at scale
- **Temporal workflow orchestration** for durable extraction, computation, and audit pipelines
- **Field-level AES-256-GCM encryption** for PAN/phone via AWS KMS
- **Append-only immutability** for extractions, computations, and audit events
- **Explicit non-ERI boundary** — filing-support only, with disclaimers on every output

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 14, TypeScript, Tailwind CSS |
| API | NestJS, TypeScript, TypeORM, class-validator |
| Workers | Python, FastAPI, Temporal, pdf2image, WeasyPrint |
| Database | PostgreSQL 15, Redis 7, PgBouncer |
| Infrastructure | AWS S3 (ap-south-1), KMS, SES, ECS Fargate |
| DevOps | Docker, GitHub Actions, pnpm workspaces |

---

## Compliance & Risk Posture

- **DPDP Act 2023:** consent-first data collection, right to erasure via Temporal W7
- **IT Act 2000 / SPDI Rules 2011:** reasonable security practices, 7-year audit retention
- **Non-ERI boundary:** never claims to file returns; all outputs labeled *"ESTIMATE ONLY"*

---

## Quick-Start for Reviewers

- **Recruiters / Hiring Managers:** Start with [`ARCHITECTURE.md`](ARCHITECTURE.md) for a 2-minute system overview.
- **Engineers:** Jump to [`docs/09_delivery/14a_index.md`](docs/09_delivery/14a_index.md) for the 83-chunk implementation roadmap.
- **Compliance / Legal:** See [`docs/06_compliance/10_compliance.md`](docs/06_compliance/10_compliance.md) for the full controls matrix.

---

## Delivery Backlog

See [`docs/09_delivery/`](docs/09_delivery/) for the implementation backlog:
- `14a_index.md` — Master context + chunk index (C01–C83)
- `14b_epics_01_05.md` — Epics E01–E05 (Infrastructure → OCR Workers)
- `14c_epics_06_10.md` — Epics E06–E10 (Extraction UI → Frontend Auth)
- `14d_epics_11_15.md` — Epics E11–E15 (Frontend Docs → Security Hardening)

---

**Contact:** sahilzafar1201@gmail.com | Jabalpur, Madhya Pradesh, India
