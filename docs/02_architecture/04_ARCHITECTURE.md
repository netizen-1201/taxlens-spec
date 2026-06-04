# 04 — System Architecture

> **Role:** Principal Solutions Architect, Regulated Fintech SaaS
> **Inputs:** Master context, PRD (03_PRD.md), MVP definition (02_MVP.md)
> **Stack confirmed:** Next.js + TypeScript (FE), NestJS + TypeScript (BE), Python workers, PostgreSQL, S3-compatible storage, Redis, Temporal

---

## Architecture Overview

TaxLens is designed as a **modular monolith for v1**, with well-defined internal module boundaries that allow extraction into independent services as scale demands. The system has six functional layers:

```
┌──────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
│             Next.js (TypeScript) — Web App (SSR/SPA)             │
└──────────────────────────┬───────────────────────────────────────┘
                           │ HTTPS / REST + WebSocket
┌──────────────────────────▼───────────────────────────────────────┐
│                       API GATEWAY LAYER                          │
│                NestJS (TypeScript) — REST API                    │
│     Auth · Rate Limiting · Request Validation · RBAC Guard       │
└────┬───────────┬──────────┬──────────────┬───────────────────────┘
     │           │          │              │
┌────▼────┐ ┌───▼────┐ ┌───▼──────┐ ┌────▼──────────┐
│Document │ │ Tax    │ │ Report   │ │  Audit /      │
│Ingestion│ │Profile │ │Generation│ │  Compliance   │
│ Module  │ │+ Engine│ │ Module   │ │  Module       │
└────┬────┘ └───┬────┘ └───┬──────┘ └────┬──────────┘
     │          │          │              │
┌────▼──────────▼──────────▼──────────────▼──────────┐
│                    DATA ACCESS LAYER                 │
│           PostgreSQL (primary) + Redis (cache/queue) │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                WORKER / ORCHESTRATION LAYER           │
│   Python FastAPI workers + Temporal workflow engine   │
│   OCR Vendor SDK  ·  PDF Generator  ·  Virus Scanner │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                  EXTERNAL SERVICES                    │
│   S3-compatible Storage · OCR/Doc AI Vendor ·        │
│   Email (SES/SMTP) · KMS · Antivirus                 │
└──────────────────────────────────────────────────────┘
```

---

## Module Breakdown

### 1. Client Layer — Next.js Frontend
**Responsibility:** Render all user-facing screens; handle upload, review, deduction input, tax estimate display, and report download.

Key choices:
- Server-side rendering (SSR) for initial page load performance on mobile.
- Client-side state management (Zustand or React Query) for extraction review and computation state.
- Pre-signed URL upload — frontend uploads directly to S3; backend is not in the file upload path (avoids proxying large files through the API).
- WebSocket or Server-Sent Events for long-running extraction job progress updates.

### 2. API Gateway Layer — NestJS Backend
**Responsibility:** Auth, request routing, validation, RBAC enforcement, job dispatching, and response serialization.

Modules inside NestJS:
- `AuthModule` — JWT access tokens, refresh tokens, OTP, 2FA (TOTP).
- `UserModule` — user profiles, tax year preferences, regime selection.
- `DocumentModule` — pre-signed URL issuance, document metadata, version management.
- `ExtractionModule` — job creation, status polling, extraction result CRUD, correction recording.
- `TaxProfileModule` — deduction inputs, profile CRUD.
- `ComputationModule` — trigger computation, read results, version management.
- `ReportModule` — trigger report generation, download URL issuance.
- `AuditModule` — append-only audit record ingestion.
- `ConsentModule` — consent capture, withdrawal, version tracking.
- `AdminModule` — internal ops endpoints (protected by admin role).

### 3. Document Ingestion Module
**Responsibility:** Validate, virus-scan, store, and queue uploaded documents for extraction.

Flow:
1. Client requests pre-signed S3 upload URL from API.
2. Client uploads file directly to S3.
3. S3 event notification (or webhook) triggers NestJS to confirm upload.
4. NestJS creates `Document` record (metadata, version 1, status=`PENDING_SCAN`).
5. NestJS enqueues a `VirusScanJob` via Redis queue (BullMQ).
6. Worker scans file; on pass → status=`PENDING_EXTRACTION`; on fail → status=`QUARANTINED`, user notified.
7. NestJS enqueues `ExtractionJob` via Temporal.

### 4. Extraction Worker Layer — Python
**Responsibility:** Call OCR vendor API, map raw OCR output to domain-specific field schemas, score confidence, produce structured JSON.

Key design points:
- Each document type has a dedicated field mapper (Form16Mapper, AISMapper, Form26ASMapper, SalarySlipMapper, InterestCertMapper).
- Mapper applies regex, positional heuristics, and LLM-assisted field extraction for ambiguous layouts.
- Output: `ExtractionResult` JSON with field values, confidence scores, bounding box references.
- Confidence < 0.75 on any Must-Have field → job status = `NEEDS_REVIEW`.
- Stores extraction result versioned in PostgreSQL; original file unchanged in S3.

### 5. Tax Rules Engine (inside NestJS ComputationModule)
**Responsibility:** Accept normalized income/deduction inputs, apply versioned tax rules for the selected AY and regime, return traceable computation output.

Key design points:
- Rules are data-driven objects (not hardcoded `if` chains), versioned by AY.
- Supported AY for v1: AY 2025-26 (FY 2024-25).
- Computes: old regime (with all inputted deductions) and new regime (with permitted deductions only).
- Output includes: full slab-wise breakdown, TDS reconciliation, net payable/refundable.
- Every computation is stored as an immutable `ComputationRun` record.

### 6. Report Generation Module — Python worker
**Responsibility:** Accept a `ComputationRun` ID, assemble all relevant data, render a PDF using a template engine, store to S3, return a signed download URL.

Key design points:
- PDF library: WeasyPrint or ReportLab (Python).
- Template: HTML/CSS → PDF; makes design changes easy.
- Mandatory elements: disclaimer, version tag, computation date.
- Report is linked to the exact `ComputationRun` and `ExtractionVersion` that produced it.

### 7. Audit / Compliance Module
**Responsibility:** Record every domain event as an append-only audit record. Support admin inspection and user-facing activity timeline.

Design:
- All audit writes go through a dedicated service (never direct DB writes from business logic).
- Audit records stored in a separate `audit_events` table (append-only via row-level trigger or application constraint).
- Events published to audit log topic (for future SIEM integration).

### 8. Temporal Orchestration
**Responsibility:** Manage long-running, retry-safe, multi-step workflows for extraction, report generation, and compliance events.

Workflows:
- `DocumentIngestionWorkflow` — scan → extract → notify
- `ReportGenerationWorkflow` — collect data → render → store → notify
- `DataDeletionWorkflow` — consent withdrawal → delete all PII → audit → confirm

---

## Data Flow: Core Upload-to-Estimate Flow

```
User uploads Form 16
  → Pre-signed URL (NestJS) → S3 upload (direct)
  → S3 event → NestJS confirms upload → creates Document record
  → BullMQ VirusScanJob → Python scanner → pass
  → Temporal DocumentIngestionWorkflow starts
  → OCR Vendor API called → raw response
  → Form16Mapper applied → ExtractionResult (JSON) stored
  → confidence scoring → job status updated
  → WebSocket event → Frontend shows Review screen
  → User reviews/corrects → locks extraction
  → ComputationRun triggered (NestJS)
  → Tax engine applies rules → ComputationRun stored
  → User views estimate → downloads PDF
  → ReportGenerationWorkflow → PDF stored in S3 → signed URL returned
```

---

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Monolith vs microservices | **Modular monolith** for v1 | Too early for service mesh overhead; module boundaries are clean enough to extract later |
| Extraction language | **Python** workers | OCR vendor SDKs, ML libs, PDF manipulation are Python-native; NestJS is the API layer only |
| Job queue | **Temporal** for workflows, **BullMQ/Redis** for lightweight jobs (virus scan) | Temporal handles durable workflows with retry; BullMQ handles fire-and-forget tasks |
| File upload path | **Direct S3 pre-signed URL** (client → S3, not through API) | Avoids API memory pressure on large PDFs; standard fintech pattern |
| State mutation strategy | **Append-only for extractions, computations, audit** | Immutability enables audit trail without extra effort |
| Tax rules storage | **Database-driven rule objects** versioned by AY | Allows rules to be updated without code deploy for minor changes |
| PAN/Aadhaar storage | **Field-level encrypted, KMS-wrapped key** | DPDP Act compliance; limits blast radius on DB breach |

---

## Internal and External Dependencies

### Internal
| Module | Depends On |
|--------|-----------|
| Extraction workers | Document module (for file retrieval), OCR vendor API |
| Tax engine | Extraction module (for normalized income data), Tax profile (for deductions) |
| Report generator | Computation module (for run data), Document module (for source doc references) |
| Audit module | All other modules (event publisher interface) |

### External
| Service | Purpose | Fallback |
|---------|---------|---------|
| OCR/Doc AI vendor | Document extraction | Manual extraction entry (UI fallback) |
| S3-compatible storage (AWS S3 / MinIO) | File storage | Cannot fallback; SLA-dependent |
| Redis | Job queues, session cache | Service degradation; queued jobs pause |
| Email provider (SES/SMTP) | Notifications | Non-blocking; retry queue |
| KMS | Key management for PII fields | Emergency: software-based key store (not for prod) |
| ClamAV / antivirus | File scanning | Block upload on scanner unavailability |

---

## Failure Handling

| Failure Point | Impact | Mitigation |
|---------------|--------|-----------|
| OCR vendor API down | Extraction jobs stall | Temporal retry with exponential backoff; user notified after timeout |
| S3 upload failure | User cannot upload | Pre-signed URL expiry (15 min); retry prompt on frontend |
| Tax engine throws exception | No computation result | Error returned with field-level validation messages; no partial result stored |
| Report generation fails | No PDF | Retry via Temporal; user notified; manual download retry available |
| Database unavailable | All writes blocked | Connection pooling (PgBouncer); read replicas for reporting queries |
| Virus scan timeout | Upload stuck | Max 3 retries then quarantine and alert ops |
| Redis failure | Job queue down | BullMQ persists jobs to Redis AOF; short downtime recoverable |

---

## Recommended v1 Architecture

**Modular monolith** deployed as:
- 1 NestJS API service
- 1 Python extraction worker service (separate process, shared DB)
- 1 Python report generation worker service
- 1 Temporal server (managed or self-hosted)
- 1 PostgreSQL instance (RDS or managed)
- 1 Redis instance (ElastiCache or managed)
- S3 bucket (ap-south-1 for data residency)

All services containerized (Docker); deployed on ECS Fargate or a small Kubernetes cluster. No serverless for v1 — predictable workload, cold start unacceptable for extraction jobs.

---

*Artifact status: ARCHITECTURE APPROVED.*
*Next: Stitching check (Prompts 1–4), then Prompt 5 (Database Schema).*
