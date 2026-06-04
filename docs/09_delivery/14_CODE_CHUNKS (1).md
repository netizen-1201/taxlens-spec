# 14 — Claude-Sized Coding Chunks

> **Role:** Senior Software Engineer
> **Inputs:** 03_PRD, 04_ARCHITECTURE, 05_SCHEMA, 06_API, 07_WORKFLOWS, 08_EXTRACTION, 09_TAX_ENGINE, 10_COMPLIANCE, 11_FRONTEND, 12_ADMIN, 13_BACKLOG, STITCH_01–03
> **Chunk size:** Each chunk fits in one Claude session (one module, one service, one workflow, or 1–3 closely related files).
> **Dependency rule:** Chunks are ordered; do not skip prerequisites.
> **AI-assist legend:** ✅ Claude first draft | ⚠️ Claude draft + human review | 🔒 human-led

---

## How to Use This Document

1. Start at C01. Complete each chunk fully before moving to the next.
2. Save every output file before opening a new session.
3. Each chunk's **Implementation Prompt** is copy-paste ready. Paste the master context block first (below), then the chunk prompt.
4. For chunks marked ⚠️ or 🔒, treat Claude's output as a first draft and validate against the referenced artifacts before committing.

---

## Master Context Block (Paste at Top of Every Session)

```
Project: TaxLens — India-focused income-tax SaaS for salaried individuals (ITR-1 segment).
AY scope: AY 2025-26 primary. Stack: NestJS + TypeScript (API), Next.js + TypeScript (frontend),
Python + FastAPI/Temporal workers (extraction + reports), PostgreSQL, Redis, S3, Temporal.
Key rules: Modular monolith for v1. Append-only for extractions, computations, audit_events.
Consent gate on all CS-05 to CS-14 actions. Non-ERI: never claim to file returns.
All computation output carries disclaimer. PAN/phone field-level encrypted via KMS.
Coding style: TypeScript strict mode. DTOs with class-validator. snake_case JSON.
UUIDs for all PKs. ISO 8601 UTC timestamps.
```

---

## Chunk Index

| Chunk | Epic | Task(s) | Size | AI |
|-------|------|---------|------|----|
| C01 | E01 | T01.1 | M | ✅ |
| C02 | E01 | T01.2 | M | ✅ |
| C03 | E01 | T01.3, T01.4 | S | ✅ |
| C04 | E01 | T01.5 | M | ✅ |
| C05 | E01 | T01.6, T01.7 | M | ⚠️ |
| C06 | E01 | T01.8 | L | ✅ |
| C07 | E01 | T01.9 part 1 | L | ✅ |
| C08 | E01 | T01.9 part 2 | L | ✅ |
| C09 | E01 | T01.9 part 3 | M | ✅ |
| C10 | E02 | T02.1 | M | ✅ |
| C11 | E02 | T02.2 | M | ✅ |
| C12 | E02 | T02.3 | M | ⚠️ |
| C13 | E02 | T02.4 | S | ✅ |
| C14 | E02 | T02.5 | M | 🔒 |
| C15 | E02 | T02.6 | M | ⚠️ |
| C16 | E02 | T02.7, T02.8 | S | ✅ |
| C17 | E02 | T02.9 | S | ✅ |
| C18 | E03 | T03.1 | M | ⚠️ |
| C19 | E03 | T03.2, T03.4 | M | ✅ |
| C20 | E03 | T03.3 | M | ✅ |
| C21 | E03 | T03.5 | M | ✅ |
| C22 | E03 | T03.6 | M | ✅ |
| C23 | E03 | T03.7 | M | ✅ |
| C24 | E04 | T04.1 | M | ✅ |
| C25 | E04 | T04.2, T04.3 | M | ⚠️ |
| C26 | E04 | T04.4 | M | ✅ |
| C27 | E04 | T04.5 | S | ✅ |
| C28 | E05 | T05.1 | M | ⚠️ |
| C29 | E05 | T05.2 part A | L | ⚠️ |
| C30 | E05 | T05.2 part B | L | ⚠️ |
| C31 | E05 | T05.3 PDF | L | ⚠️ |
| C32 | E05 | T05.3 CSV | M | ⚠️ |
| C33 | E05 | T05.4 | M | ⚠️ |
| C34 | E05 | T05.5 | L | ⚠️ |
| C35 | E05 | T05.6 | M | ✅ |
| C36 | E05 | T05.7, T05.8 | L | ✅ |
| C37 | E05 | T05.9–T05.11 | M | ✅ |
| C38 | E05/E06 | T06.1–T06.4 | M | ⚠️ |
| C39 | E06 | T06.5, T06.6 | M | ✅ |
| C40 | E07 | T07.1, T07.2 | M | ✅ |
| C41 | E07 | T07.3 | L | 🔒 |
| C42 | E07 | T07.4 | S | ✅ |
| C43 | E08 | T08.1, T08.2 | M | ⚠️ |
| C44 | E08 | T08.3, T08.4 | M | ⚠️ |
| C45 | E08 | T08.5 | M | ⚠️ |
| C46 | E08 | T08.6, T08.7 | M | ✅ |
| C47 | E08 | T08.8 | M | ✅ |
| C48 | E08 | T08.9 | M | ✅ |
| C49 | E08 | T08.10 | S | ✅ |
| C50 | E08 | T08.11 | L | ⚠️ |
| C51 | E09 | T09.1 | L | ✅ |
| C52 | E09 | T09.2 | M | ✅ |
| C53 | E09 | T09.3 | M | ✅ |
| C54 | E09 | T09.4 | S | ✅ |
| C55 | E10 | T10.1 | M | ✅ |
| C56 | E10 | T10.2 | M | ✅ |
| C57 | E10 | T10.3 | M | 🔒 |
| C58 | E10 | T10.4 | S | ✅ |
| C59 | E11 | T11.1 | M | ✅ |
| C60 | E11 | T11.2 | L | ⚠️ |
| C61 | E11 | T11.3 | M | ✅ |
| C62 | E11 | T11.4 part 1 | L | ⚠️ |
| C63 | E11 | T11.4 part 2 | L | ⚠️ |
| C64 | E12 | T12.1 | L | ✅ |
| C65 | E12 | T12.2 part 1 | L | ⚠️ |
| C66 | E12 | T12.2 part 2 | L | ⚠️ |
| C67 | E12 | T12.3 | M | ✅ |
| C68 | E12 | T12.4 | M | ✅ |
| C69 | E12 | T12.5, T12.6 | M | ✅ |
| C70 | E13 | T13.1 | M | 🔒 |
| C71 | E13 | T13.2 | L | ✅ |
| C72 | E13 | T13.3 | L | ✅ |
| C73 | E13 | T13.4 | L | ⚠️ |
| C74 | E13 | T13.5, T13.6 | M | ✅ |
| C75 | E13 | T13.7, T13.8 | M | ⚠️ |
| C76 | E13 | T13.9, T13.10 | M | ✅ |
| C77 | E14 | T14.1 part 1 | XL | 🔒 |
| C78 | E14 | T14.1 part 2 | XL | 🔒 |
| C79 | E14 | T14.2, T14.3 | M | ⚠️ |
| C80 | E14 | T14.4 | M | ✅ |
| C81 | E15 | T15.1, T15.4, T15.5 | S | ✅ |
| C82 | E15 | T15.2 | M | 🔒 |
| C83 | E15 | T15.3 | S | ✅ |

---

## E01 — Project Infrastructure & DevOps

---

### C01 — Monorepo Scaffolding

**Epic:** E01 | **Backlog task:** T01.1 | **AI:** ✅

**Goal:** Initialize the TaxLens monorepo with three workspaces (NestJS API, Next.js frontend, Python workers) and a shared TypeScript types package.

**Prerequisites:** None.

**Implementation Prompt:**
```
[Paste master context block above]

Create the monorepo scaffold for TaxLens with the following structure:
taxlens/
  apps/
    api/          # NestJS + TypeScript backend (skeleton only)
    web/          # Next.js 14 + TypeScript frontend (skeleton only)
  workers/
    extraction/   # Python FastAPI + Temporal extraction worker (skeleton)
    reports/      # Python report generation worker (skeleton)
  packages/
    types/        # Shared TypeScript interfaces
  package.json    # Root pnpm workspace config
  tsconfig.base.json
  .env.example

Requirements:
- Use pnpm workspaces.
- NestJS: TypeScript strict mode; include @nestjs/config, @nestjs/typeorm, @nestjs/bull,
  class-validator, class-transformer, @nestjs/jwt, @nestjs/passport, @nestjs/schedule.
- Next.js: create-next-app with TypeScript + Tailwind + App Router.
- packages/types: export shared interfaces from 05_SCHEMA (table shapes), 06_API (DTOs),
  09_TAX_ENGINE (ComputationInput, ComputationResult), 08_EXTRACTION (ExtractionResult).
- Python workers: pyproject.toml with fastapi, temporalio, boto3, sqlalchemy, psycopg2-binary,
  pydantic, python-dotenv, pdfminer.six, pdf2image, Pillow, WeasyPrint.
- Root .env.example covering: DATABASE_URL, REDIS_URL, TEMPORAL_ADDRESS, AWS_REGION,
  S3_BUCKET_DOCUMENTS, S3_BUCKET_REPORTS, S3_BUCKET_OCR_RAW, KMS_KEY_ARN,
  JWT_SECRET, JWT_REFRESH_SECRET, OCR_VENDOR, CONSENT_VERSION, FROM_EMAIL_ADDRESS.
Generate all files. Scaffold only — no business logic. Each service must have a working
dev entry point.
```

**Expected Files:**
```
taxlens/package.json + pnpm-workspace.yaml + tsconfig.base.json + .env.example
apps/api/package.json + src/main.ts + src/app.module.ts
apps/web/package.json + next.config.js + src/app/layout.tsx
packages/types/src/index.ts (schema.types.ts, api.types.ts, engine.types.ts, extraction.types.ts)
workers/extraction/pyproject.toml + main.py
workers/reports/pyproject.toml + main.py
```

**Acceptance Criteria:**
- `pnpm install` from root installs all workspaces without errors.
- `pnpm --filter api dev` starts NestJS on port 3001.
- `pnpm --filter web dev` starts Next.js on port 3000.
- `packages/types` exports compile without errors when imported in the API.
- `.env.example` lists every environment variable used across all services.

---

### C02 — Docker Setup

**Epic:** E01 | **Backlog task:** T01.2 | **AI:** ✅

**Goal:** Production-ready Dockerfiles for each service and a docker-compose for local development.

**Prerequisites:** C01.

**Implementation Prompt:**
```
[Paste master context block]

Create Docker configuration for TaxLens.

1. apps/api/Dockerfile — Multi-stage: node:20-alpine builder → node:20-alpine runner.
   Non-root user. Expose 3001.

2. workers/extraction/Dockerfile — Python 3.11-slim. Install poppler-utils, libpoppler-cpp-dev
   for pdf2image. Non-root. Expose 8001.

3. workers/reports/Dockerfile — Python 3.11-slim. Install WeasyPrint deps:
   pango, cairo, gdk-pixbuf. Non-root. Expose 8002.

4. docker-compose.yml (root):
   - api (hot-reload via volume), web (hot-reload), extraction-worker, reports-worker
   - postgres:15-alpine (port 5432; volume; POSTGRES_DB=taxlens)
   - redis:7-alpine (port 6379; appendonly yes)
   - temporalio/autosetup:1.22 (ports 7233 + 8088; depends_on postgres, redis)
   - mkodockx/docker-clamav (port 3310)
   All services reference .env.example vars.
   Healthchecks on postgres (pg_isready) and redis (redis-cli ping).
   Named volumes for postgres and redis data.

5. docker-compose.override.yml — dev volume mounts for hot-reload, DEBUG env vars.
```

**Expected Files:**
```
apps/api/Dockerfile
workers/extraction/Dockerfile
workers/reports/Dockerfile
docker-compose.yml
docker-compose.override.yml
.dockerignore (root)
```

**Acceptance Criteria:**
- `docker-compose up` starts all services without errors.
- `docker-compose ps` shows all services healthy.
- API reachable at http://localhost:3001/health.
- Temporal UI reachable at http://localhost:8088.

---

### C03 — Database & Redis Config

**Epic:** E01 | **Backlog tasks:** T01.3, T01.4 | **AI:** ✅

**Goal:** Configure TypeORM connection for NestJS and BullMQ for Redis queue.

**Prerequisites:** C01, C02.

**Implementation Prompt:**
```
[Paste master context block]

Configure DB and Redis for TaxLens NestJS API.

1. apps/api/src/database/database.module.ts:
   TypeOrmModule.forRootAsync using ConfigService:
   type='postgres', synchronize=false, logging=['error','warn'] in dev,
   ssl enabled in production, PgBouncer-compatible (keepConnectionAlive: true, pool max 10).

2. apps/api/src/queue/queue.module.ts:
   BullModule.forRootAsync using ConfigService. Connection from REDIS_URL.
   defaultJobOptions: { removeOnComplete: 100, removeOnFail: 500, attempts: 3,
   backoff: { type: 'exponential', delay: 10000 } }.
   Export VIRUS_SCAN_QUEUE constant.

3. apps/api/src/app.module.ts: ConfigModule with Joi validation for all required env vars
   (DATABASE_URL, REDIS_URL, JWT_SECRET min 32 chars, JWT_REFRESH_SECRET, AWS_REGION,
   S3_BUCKET_DOCUMENTS, KMS_KEY_ARN; TEMPORAL_ADDRESS default 'localhost:7233').
```

**Expected Files:**
```
apps/api/src/database/database.module.ts
apps/api/src/queue/queue.module.ts
apps/api/src/config/config.validation.ts
apps/api/src/app.module.ts (updated imports)
```

**Acceptance Criteria:**
- API starts without errors when docker-compose is running.
- TypeORM connects to PostgreSQL (migration:run runs without connection error).
- BullMQ queue registered and visible in Redis (`KEYS bull:*`).

---

### C04 — Temporal Worker Setup

**Epic:** E01 | **Backlog task:** T01.5 | **AI:** ⚠️

**Goal:** Register Temporal workers in NestJS (workflow starters/signals) and Python extraction worker.

**Prerequisites:** C02, C03.

**Implementation Prompt:**
```
[Paste master context block]

Set up Temporal workflow workers for TaxLens.

1. apps/api/src/temporal/temporal.module.ts:
   Use @temporalio/client. TemporalModule.forRootAsync: inject ConfigService;
   create Connection + WorkflowClient.
   Export TemporalService with methods:
   - startWorkflow(workflowType, workflowId, args)
   - signalWorkflow(workflowId, signalName, args)
   - getWorkflowHandle(workflowId)
   WorkflowIDs: '{workflowType}/{resourceId}'.

2. workers/extraction/temporal_worker.py:
   Use temporalio SDK. Connect to TEMPORAL_ADDRESS.
   Task queue: 'extraction-queue'.
   Register activity stubs (implementation in later chunks):
   update_extraction_job_status, fetch_document_from_s3, call_ocr_vendor,
   map_fields, store_extraction_result, evaluate_confidence, notify_user.
   max_concurrent_activities=10.

3. workers/reports/temporal_worker.py:
   Task queue: 'reports-queue'. Register: assemble_report_data, render_pdf,
   store_pdf_to_s3, store_report_record.

4. apps/api/src/temporal/workflows/index.ts:
   Stub definitions for W1–W9 (async functions, names only).
```

**Expected Files:**
```
apps/api/src/temporal/temporal.module.ts
apps/api/src/temporal/temporal.service.ts
apps/api/src/temporal/workflows/index.ts
workers/extraction/temporal_worker.py
workers/reports/temporal_worker.py
```

**Acceptance Criteria:**
- NestJS starts and TemporalService connects without error.
- Python extraction worker registers with Temporal (check Temporal UI → Task Queues → 'extraction-queue').
- `startWorkflow` on a stubbed workflow creates a visible run in Temporal UI.

---

### C05 — S3 Bucket & KMS Config Scripts

**Epic:** E01 | **Backlog tasks:** T01.6, T01.7 | **AI:** ⚠️

**Goal:** AWS CLI scripts to provision S3 buckets with correct policies, lifecycle rules, and KMS key.

**Prerequisites:** C01.

**Implementation Prompt:**
```
[Paste master context block]

Write infrastructure scripts for TaxLens S3 and KMS setup. Use AWS CLI shell scripts.

1. infra/s3-setup.sh — Creates three buckets (taxlens-documents-{ENV},
   taxlens-reports-{ENV}, taxlens-ocr-raw-{ENV}) in ap-south-1.
   Each bucket: block public access, SSE-KMS using KMS_KEY_ARN,
   deny non-SSL requests, deny replication outside ap-south-1 (DPDP Act).
   Lifecycle rules: documents → GLACIER after 730 days; reports → delete after 1095 days;
   ocr-raw → delete after 90 days.

2. infra/kms-setup.sh — Symmetric KMS key for PAN/phone/TOTP encryption.
   Enable annual key rotation. Allow taxlens-api-role to encrypt/decrypt.
   Output key ARN to stdout.

3. infra/iam-setup.sh — Creates taxlens-api-role with:
   S3: GetObject, PutObject, DeleteObject, HeadObject on all three buckets.
   KMS: Encrypt, Decrypt, GenerateDataKey. SES: SendEmail.

All scripts accept ENV arg (dev/staging/prod) and are idempotent.
Include infra/README.md with usage instructions.
```

**Expected Files:**
```
infra/s3-setup.sh
infra/kms-setup.sh
infra/iam-setup.sh
infra/README.md
```

**Acceptance Criteria:**
- `./infra/s3-setup.sh dev` creates three buckets in ap-south-1 with correct policies.
- KMS key created and ARN echoed to stdout.
- `aws s3 cp test.pdf s3://taxlens-documents-dev/test.pdf` succeeds with the role.

---

### C06 — CI/CD Pipeline

**Epic:** E01 | **Backlog task:** T01.8 | **AI:** ✅

**Goal:** GitHub Actions workflows for lint + test + build on PR; Docker push + deploy on merge.

**Prerequisites:** C01, C02.

**Implementation Prompt:**
```
[Paste master context block]

Write GitHub Actions CI/CD workflows for TaxLens monorepo.

1. .github/workflows/ci.yml — on every PR targeting main:
   Jobs (parallel): lint-and-typecheck (node:20, pnpm lint + tsc --noEmit for api and web);
   api-tests (services: postgres + redis; pnpm test + test:e2e);
   python-tests (python:3.11; pytest workers/extraction/tests/).
   PR merge blocked if any job fails.

2. .github/workflows/deploy-staging.yml — on push to main:
   Build and push Docker images (api, extraction-worker, reports-worker) to ECR via OIDC role.
   Image tag: git SHA short. ECS Fargate task update (aws ecs update-service).
   Run DB migrations after deploy. Notify Slack (SLACK_WEBHOOK_URL).

3. .github/workflows/deploy-prod.yml — manual trigger (workflow_dispatch) with confirmation
   input ("type DEPLOY to confirm"). Same steps as staging targeting prod cluster.

AWS auth via OIDC (no long-lived keys). Document required secrets in README.
```

**Expected Files:**
```
.github/workflows/ci.yml
.github/workflows/deploy-staging.yml
.github/workflows/deploy-prod.yml
.github/CODEOWNERS
```

**Acceptance Criteria:**
- Opening a PR triggers ci.yml with all three parallel jobs.
- Merging to main triggers deploy-staging.yml.
- deploy-prod.yml only runs when manually triggered with correct confirmation input.

---

### C07 — DB Migrations Part 1: Identity, Consent, Documents

**Epic:** E01 | **Backlog task:** T01.9 part 1 | **AI:** ✅

**Goal:** Migration creating `organizations`, `users`, `sessions`, `consent_records`, `documents`, `document_versions`.

**Prerequisites:** C03.

**Implementation Prompt:**
```
[Paste master context block]

Write TypeORM migration V001 for TaxLens — part 1 of 3. Reference 05_SCHEMA.md exactly.

Tables (in FK dependency order):
1. organizations (id uuid PK, name text, created_at, updated_at, deleted_at)
2. users (all columns from 05_SCHEMA: pan_encrypted bytea, phone_encrypted bytea,
   totp_secret_encrypted bytea, totp_enabled bool default false, role text default 'user',
   is_email_verified bool default false, plus created_at, updated_at, deleted_at)
3. sessions (id, user_id FK, refresh_token_hash text unique, ip_address inet,
   user_agent text, created_at, expires_at, revoked_at)
4. consent_records (id, user_id FK, consent_version text, action text
   CHECK IN ('granted','withdrawn'), ip_address inet, user_agent text, created_at)
5. documents (id, org_id FK, user_id FK, document_type text, assessment_year text,
   display_name text, latest_version_id uuid nullable, created_at, deleted_at)
6. document_versions (id, document_id FK, version_number int, s3_bucket text, s3_key text,
   file_name_original text, mime_type text, file_size_bytes bigint, checksum_sha256 text,
   virus_scan_status text default 'pending' CHECK IN ('pending','passed','quarantined','scan_timeout'),
   virus_scan_at timestamptz, upload_ip inet, created_at)

Add all indexes from 05_SCHEMA. Add partial index on document_versions.virus_scan_status='passed'.
Add DB comments on pan_encrypted and phone_encrypted: 'AES-256-GCM KMS-encrypted; never log'.
Add trigger or RLS prohibiting DELETE on consent_records and document_versions (immutable).
Include down migration.
```

**Expected Files:**
```
apps/api/src/migrations/V001__identity_consent_documents.sql
apps/api/src/migrations/V001__identity_consent_documents.down.sql
```

**Acceptance Criteria:**
- `migration:run` completes without errors.
- `\d users` shows all columns including bytea fields.
- `DELETE FROM consent_records WHERE id = '...'` is blocked with clear error.

---

### C08 — DB Migrations Part 2: Extraction, Tax, Computation, Audit

**Epic:** E01 | **Backlog task:** T01.9 part 2 | **AI:** ✅

**Goal:** Migration for extraction, tax profile, computation, reports, and audit tables.

**Prerequisites:** C07.

**Implementation Prompt:**
```
[Paste master context block]

Write TypeORM migration V002 — part 2 of 3. Reference 05_SCHEMA.md exactly.

1. extraction_jobs (id, document_version_id FK, temporal_workflow_id text, status text
   default 'queued' CHECK IN ('queued','processing','completed','needs_review','failed'),
   ocr_vendor text, ocr_vendor_job_id text, started_at, completed_at, failure_reason, created_at)
2. extraction_results (id, extraction_job_id FK unique, document_type text, assessment_year text,
   extracted_fields jsonb NOT NULL, confidence_scores jsonb NOT NULL, low_confidence_fields text[],
   overall_confidence numeric(5,4), created_at) — immutable
3. extraction_corrections (id, extraction_result_id FK, user_id FK, field_name text,
   original_value text, corrected_value text NOT NULL, note text, created_at)
4. extraction_locks (id, extraction_result_id FK unique, user_id FK, locked_at timestamptz
   default now()) — immutable; one per result
5. tax_profiles (id, user_id FK, assessment_year text, regime_preference text default 'compare'
   CHECK IN ('old','new','compare'), residential_status text default 'resident',
   employment_type text default 'salaried', created_at, updated_at; UNIQUE(user_id, assessment_year))
6. deduction_inputs (id, tax_profile_id FK, section_code text NOT NULL, sub_item text,
   amount numeric(15,2), source text CHECK IN ('user_input','extracted'), notes text,
   created_at, updated_at, deleted_at)
7. computation_runs (id, user_id FK, tax_profile_id FK, assessment_year text, regime text
   CHECK IN ('old','new'), input_snapshot jsonb NOT NULL, result jsonb NOT NULL,
   rule_version text, status text default 'completed', created_at) — immutable
8. tax_rules (id, assessment_year text, regime text, version text, is_active bool default true,
   rules_json jsonb NOT NULL, effective_from date, notes text, created_at;
   UNIQUE(assessment_year, regime, version))
9. report_records (id, user_id FK, computation_run_id FK, report_type text default 'tax_summary',
   s3_bucket text, s3_key text, version_number int, generated_at timestamptz default now()) — immutable
10. audit_events — monthly range partitioning by created_at; trigger blocking UPDATE and DELETE;
    all columns from 05_SCHEMA; add composite indexes per indexing strategy.

Add DB comment on deduction_inputs.section_code: 'Validated at app layer against PERMITTED_SECTION_CODES'.
```

**Expected Files:**
```
apps/api/src/migrations/V002__extraction_tax_audit.sql
apps/api/src/migrations/V002__extraction_tax_audit.down.sql
```

**Acceptance Criteria:**
- Full migration stack (V001 + V002) runs cleanly.
- `audit_events` has monthly partitions visible via `\d+ audit_events`.
- `UPDATE audit_events SET action='tampered'` is blocked by trigger.

---

### C09 — DB Migrations Part 3: Admin Tables + Safe View

**Epic:** E01 | **Backlog task:** T01.9 part 3 | **AI:** ✅

**Goal:** Migration for `admin_pending_actions` table (STITCH-03 G11) and `v_audit_events_safe` view (G12).

**Prerequisites:** C08.

**Implementation Prompt:**
```
[Paste master context block]

Write TypeORM migration V003 — part 3 of 3.

1. admin_pending_actions table (STITCH-03 G11):
   id uuid PK, action_type text NOT NULL CHECK IN
   ('activate_tax_rule','delete_user_data','unlock_extraction','update_user_role'),
   initiated_by uuid FK → users.id NOT NULL, workflow_id text,
   target_resource_type text NOT NULL, target_resource_id uuid NOT NULL,
   payload_json jsonb NOT NULL, status text NOT NULL default 'pending'
   CHECK IN ('pending','approved','rejected','expired'),
   initiated_at timestamptz NOT NULL default now(),
   expires_at timestamptz NOT NULL DEFAULT (now() + interval '48 hours'),
   resolved_by uuid FK → users.id nullable, resolved_at timestamptz nullable.
   Indexes: (status, initiated_at), (initiated_by), partial on expires_at where status='pending'.

2. v_audit_events_safe view (STITCH-03 G12):
   SELECT all columns from audit_events but:
   - Mask before_value/after_value: replace any key 'pan_encrypted' with '[MASKED]'
   - ip_address: mask last octet (host(set_masklen(ip_address::inet, 24))::text || '.0')
   - user_agent: left(user_agent, 50) || '...'
   View comment: 'Safe view for operator-role. PII fields masked. Admins query audit_events directly.'

3. PL/pgSQL function expire_pending_admin_actions(): sets status='expired' where
   status='pending' AND expires_at < now(). Document as pg_cron target.

Include down migration.
```

**Expected Files:**
```
apps/api/src/migrations/V003__admin_tables_and_views.sql
apps/api/src/migrations/V003__admin_tables_and_views.down.sql
```

**Acceptance Criteria:**
- All three migrations (V001–V003) run in sequence without errors.
- `SELECT * FROM v_audit_events_safe` shows masked IP after inserting a test event.
- admin_pending_actions.expires_at defaults to initiated_at + 48 hours.

---

## E02 — Auth & Identity

---

### C10 — User Registration + OTP Generation

**Epic:** E02 | **Backlog task:** T02.1 | **AI:** ✅

**Goal:** `POST /auth/register` with bcrypt, email uniqueness check, and OTP generation.

**Prerequisites:** C07, C03.

**Implementation Prompt:**
```
[Paste master context block]

Implement user registration for TaxLens NestJS API.

1. apps/api/src/auth/dto/register.dto.ts:
   email: IsEmail, transform to lowercase.
   password: IsString, MinLength(8), Matches(/^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*])/).
   full_name: IsString, IsNotEmpty, MaxLength(255).

2. apps/api/src/auth/auth.service.ts — register() method:
   - Check for existing email → 409 ConflictException if exists.
   - bcrypt.hash(password, 12).
   - Create Organization row (one per user in v1).
   - Create User row (is_email_verified=false).
   - Generate 6-digit OTP: crypto.randomInt(100000, 999999).toString().
   - Store in Redis: 'otp:{email}' TTL 600s.
   - Log OTP to console in dev (email sending wired in C11).
   - Emit audit event: auth.registered (via AuditEventService stub).
   - Return { user_id, email, message: 'Verification email sent.' }.

3. POST /auth/register in auth.controller.ts (public, no auth guard).
```

**Expected Files:**
```
apps/api/src/auth/auth.module.ts
apps/api/src/auth/auth.controller.ts
apps/api/src/auth/auth.service.ts
apps/api/src/auth/dto/register.dto.ts
apps/api/src/users/user.entity.ts
apps/api/src/users/users.module.ts
apps/api/src/users/users.service.ts
```

**Acceptance Criteria:**
- POST /auth/register with valid payload returns 201 `{ user_id, email, message }`.
- Same email returns 409. Weak password returns 422.
- OTP stored in Redis with correct TTL.

---

### C11 — Email Verification (OTP)

**Epic:** E02 | **Backlog task:** T02.2 | **AI:** ✅

**Goal:** SES email sending for OTP and `POST /auth/verify-email` endpoint.

**Prerequisites:** C10.

**Implementation Prompt:**
```
[Paste master context block]

Implement email verification for TaxLens.

1. apps/api/src/email/email.service.ts:
   sendOtpEmail(to, otp): in dev: log to console. In production: SES SendEmailCommand.
   Subject: "Your TaxLens verification code". Body includes OTP + 10-min expiry note.

2. Update auth.service.ts register() to call emailService.sendOtpEmail() after OTP storage.

3. auth.service.ts verifyEmail(email, otp):
   - Fetch from Redis 'otp:{email}'; if missing → 410 OTP expired.
   - If mismatch → 400 Invalid OTP.
   - If match: set users.is_email_verified=true; delete Redis key.
   - Emit audit event: auth.email_verified.
   - Return { message: 'Email verified.' }.

4. POST /auth/verify-email (public) with dto: email IsEmail, otp IsString Length(6,6) Matches /^\d{6}$/.
```

**Expected Files:**
```
apps/api/src/email/email.module.ts
apps/api/src/email/email.service.ts
apps/api/src/auth/dto/verify-email.dto.ts
apps/api/src/auth/auth.service.ts (updated)
apps/api/src/auth/auth.controller.ts (updated)
```

**Acceptance Criteria:**
- Register → check console for OTP → verify-email with correct OTP → 200 response.
- Second call with same OTP returns 410. is_email_verified=true in DB.

---

### C12 — JWT Auth (Login, Refresh, Logout)

**Epic:** E02 | **Backlog task:** T02.3 | **AI:** ⚠️

**Goal:** Login issuing JWT access (15 min) + refresh (7 days) tokens; refresh endpoint; logout.

**Prerequisites:** C10, C11.

**Implementation Prompt:**
```
[Paste master context block]

Implement JWT authentication for TaxLens. Security-critical — follow exactly.

1. jwt.strategy.ts: PassportStrategy from @nestjs/jwt. Extract from Bearer header.
   Validate against JWT_SECRET. Payload: { sub: user_id, email, role, jti }.

2. jwt-refresh.strategy.ts: Same but uses JWT_REFRESH_SECRET. Also checks sessions table
   that the hashed refresh token is not revoked.

3. auth.service.ts login(email, password, ip, userAgent):
   - Find user by email (case-insensitive); check is_email_verified; check deleted_at.
   - bcrypt.compare constant-time. On failure: call recordFailedAttempt() (C17 lockout).
   - On success: clearFailedAttempts().
   - Access token: { sub, email, role, jti: uuid() } expiresIn '15m'.
   - Refresh token: { sub, jti: uuid() } expiresIn '7d'.
   - SHA-256 hash refresh token; store in sessions table with ip, userAgent, expires_at.
   - Emit audit event: auth.login.
   - Return { access_token, refresh_token, expires_in: 900 }.

4. refreshToken(): validate refresh JWT; find non-revoked session; issue new access token.
5. logout(refreshTokenHash): set revoked_at=now() in sessions; emit auth.logout.
6. JwtAuthGuard and @Public() decorator for opt-out.
7. Errors: 401 invalid credentials (same message always — no enumeration), 403 unverified, 423 locked.
```

**Expected Files:**
```
apps/api/src/auth/strategies/jwt.strategy.ts
apps/api/src/auth/strategies/jwt-refresh.strategy.ts
apps/api/src/auth/guards/jwt-auth.guard.ts
apps/api/src/auth/decorators/public.decorator.ts
apps/api/src/auth/dto/login.dto.ts
apps/api/src/auth/auth.service.ts (updated)
apps/api/src/auth/auth.controller.ts (login, refresh, logout routes)
```

**Acceptance Criteria:**
- POST /auth/login returns `{ access_token, refresh_token, expires_in: 900 }`.
- POST /auth/refresh with valid refresh token issues new access token.
- POST /auth/logout → POST /auth/refresh with same token returns 401.

---

### C13 — Password Reset

**Epic:** E02 | **Backlog task:** T02.4 | **AI:** ✅

**Goal:** Forgot password + reset password endpoints with tokenized link, 1-hour expiry.

**Prerequisites:** C11, C12.

**Implementation Prompt:**
```
[Paste master context block]

Implement password reset for TaxLens.

1. auth.service.ts forgotPassword(email):
   Always return 200 (prevents email enumeration).
   If user found: crypto.randomBytes(32).toString('hex') → store 'pwd_reset:{token}' in Redis TTL 3600s.
   Send email via EmailService with reset link. Emit auth.password_reset_requested.

2. auth.service.ts resetPassword(token, newPassword):
   Fetch from Redis. If missing → 400. Validate newPassword. Hash; update users.password_hash.
   Delete Redis key. Revoke all user sessions (security: force re-login). Emit auth.password_reset.

3. DTOs: ForgotPasswordDto (email), ResetPasswordDto (token, new_password).
4. POST /auth/forgot-password (public), POST /auth/reset-password (public).
```

**Expected Files:**
```
apps/api/src/auth/dto/forgot-password.dto.ts
apps/api/src/auth/dto/reset-password.dto.ts
apps/api/src/auth/auth.service.ts (updated)
apps/api/src/auth/auth.controller.ts (updated)
```

**Acceptance Criteria:**
- POST /auth/forgot-password with unknown email still returns 200.
- POST /auth/reset-password with valid token updates password; old password no longer works.
- Using reset token twice returns 400 on second use. All sessions revoked after reset.

---

### C14 — TOTP 2FA

**Epic:** E02 | **Backlog task:** T02.5 | **AI:** 🔒

**Goal:** TOTP (RFC 6238) 2FA: setup, verify, disable; login flow update for 2FA users.

**Prerequisites:** C12, C15 (KMS stub).

**Implementation Prompt:**
```
[Paste master context block]

Implement TOTP 2FA for TaxLens. Security-sensitive — follow exactly.
Dependency: otplib npm package.

1. auth.service.ts setup2FA(userId):
   authenticator.generateSecret() from otplib.
   Encrypt secret via EncryptionService.encrypt() → store as users.totp_secret_encrypted.
   Set Redis 'totp_pending_{userId}' (2FA not active until verified).
   Return { secret (plaintext for QR), qr_uri: authenticator.keyuri(email, 'TaxLens', secret) }.

2. auth.service.ts verify2FA(userId, token):
   Fetch + decrypt users.totp_secret_encrypted.
   authenticator.verify({ token, secret }), window: 1.
   If valid: clear pending key; set users.totp_enabled=true. Return { message: '2FA enabled.' }.

3. auth.service.ts disable2FA(userId, currentPassword):
   Verify currentPassword before disabling. Set totp_secret_encrypted=NULL, totp_enabled=false.
   Emit auth.2fa_disabled.

4. Login flow update: if user.totp_enabled: after password check, issue partial_token
   (short-lived JWT with sub + requires_2fa:true). POST /auth/login/2fa: validate partial_token
   + totp_token → issue full JWT pair.

Note: totp_enabled column already in V001 migration (add if not present as V001b migration).
```

**Expected Files:**
```
apps/api/src/auth/auth.service.ts (updated)
apps/api/src/auth/auth.controller.ts (/auth/2fa/setup, /2fa/verify, /2fa/disable, /login/2fa)
apps/api/src/auth/dto/verify-2fa.dto.ts
apps/api/src/auth/dto/disable-2fa.dto.ts
apps/api/src/migrations/V001b__add_totp_enabled.sql
```

**Acceptance Criteria:**
- Setup returns valid `otpauth://` URI scannable by Google Authenticator.
- Verifying with correct TOTP enables 2FA.
- Login with 2FA enabled requires TOTP token; wrong token returns 401.
- Disable requires password confirmation.

---

### C15 — User Profile CRUD + PAN Encryption

**Epic:** E02 | **Backlog task:** T02.6 | **AI:** ⚠️

**Goal:** `GET /users/me`, `PATCH /users/me` with KMS-encrypted PAN; return masked PAN only.

**Prerequisites:** C12.

**Implementation Prompt:**
```
[Paste master context block]

Implement user profile endpoints with PAN field-level encryption.

1. EncryptionService stub (apps/api/src/encryption/encryption.service.ts):
   encrypt(plaintext: string): Promise<Buffer> → Buffer.from(plaintext, 'utf8') (stub; replaced in C82).
   decrypt(ciphertext: Buffer): Promise<string> → ciphertext.toString('utf8') (stub).
   maskPan(pan: string): string → `${pan.slice(0,5)}****${pan.slice(-1)}` e.g. ABCDE****F.

2. users.service.ts getProfile(userId): return { user_id, email, full_name, role,
   pan_last4 (maskPan if pan_encrypted not null), is_email_verified, created_at }.
   Never return pan_encrypted raw bytes.

3. update-profile.dto.ts: full_name optional; pan optional with regex AAAAA9999A.

4. users.service.ts updateProfile(userId, dto):
   If full_name: update + emit user.profile_updated.
   If pan: EncryptionService.encrypt(pan.toUpperCase()) → pan_encrypted; emit user.pan_updated
   (before/after both masked with maskPan).

5. GET /users/me and PATCH /users/me (both JWT-protected).
```

**Expected Files:**
```
apps/api/src/encryption/encryption.module.ts
apps/api/src/encryption/encryption.service.ts
apps/api/src/users/users.controller.ts
apps/api/src/users/dto/update-profile.dto.ts
apps/api/src/users/users.service.ts (updated)
```

**Acceptance Criteria:**
- GET /users/me does not expose pan_encrypted bytes.
- PATCH with PAN ABCDE1234F stores encrypted; GET shows pan_last4: "234F".
- Invalid PAN format returns 422. Audit event has masked values.

---

### C16 — RBAC Guard + Session Management

**Epic:** E02 | **Backlog tasks:** T02.7, T02.8 | **AI:** ✅

**Goal:** `@Roles()` decorator + `RolesGuard`; session cleanup cron job.

**Prerequisites:** C12.

**Implementation Prompt:**
```
[Paste master context block]

Implement RBAC and session management for TaxLens.

1. auth/decorators/roles.decorator.ts: @Roles(...roles: string[]) using SetMetadata.

2. auth/guards/roles.guard.ts: Implements CanActivate; reads roles from metadata;
   compares req.user.role. No @Roles → allow any authenticated user.
   @Roles('admin') → deny if role !== 'admin'. Return 403 ForbiddenException.

3. Apply JwtAuthGuard and RolesGuard globally in AppModule (APP_GUARD providers).
   Use @Public() to opt routes out.

4. auth/sessions.service.ts:
   createSession(userId, refreshTokenHash, ip, userAgent): insert sessions row.
   revokeSession(refreshTokenHash): set revoked_at=now().
   revokeAllUserSessions(userId): revoke all active sessions (used in W7).
   isSessionValid(refreshTokenHash): check exists + not revoked + not expired.
   cleanupExpiredSessions(): DELETE WHERE expires_at < now() - interval '7 days'.

5. auth/sessions.cron.ts: @Cron(EVERY_DAY_AT_3AM) calls cleanupExpiredSessions().
   Register ScheduleModule in AppModule.
```

**Expected Files:**
```
apps/api/src/auth/decorators/roles.decorator.ts
apps/api/src/auth/guards/roles.guard.ts
apps/api/src/auth/sessions.service.ts
apps/api/src/auth/sessions.cron.ts
apps/api/src/app.module.ts (updated with global guards + ScheduleModule)
```

**Acceptance Criteria:**
- @Roles('admin') route returns 403 for user-role JWT.
- @Public() route returns 200 with no JWT.
- Cron job runs without error.

---

### C17 — Account Lockout

**Epic:** E02 | **Backlog task:** T02.9 | **AI:** ✅

**Goal:** Track failed login attempts per email; lock after 10 failures for 10 minutes.

**Prerequisites:** C12, C16.

**Implementation Prompt:**
```
[Paste master context block]

Implement account lockout for TaxLens login brute-force protection.

Add to auth.service.ts:

1. checkLockout(email): if Redis key 'login_lock:{email}' exists → throw 423
   HttpException('Account temporarily locked. Try again in 10 minutes.', 423).

2. recordFailedAttempt(email):
   Increment 'login_attempts:{email}' in Redis.
   If count >= 10: set 'login_lock:{email}' TTL 600s; delete attempts key;
   emit auth.account_locked.
   Else: set TTL on attempts key to 1800s (rolling window).

3. clearFailedAttempts(email): delete 'login_attempts:{email}' and 'login_lock:{email}'.

4. Update login(): call checkLockout(email) BEFORE password compare.
   On failed compare: recordFailedAttempt(email). On success: clearFailedAttempts(email).

5. GET /auth/unlock/:email (@Roles('admin') only): delete both Redis keys; emit admin.account_unlocked.
   Note: 423 and lockout error message same whether user exists or not (no enumeration).
```

**Expected Files:**
```
apps/api/src/auth/auth.service.ts (updated lockout methods)
apps/api/src/auth/auth.controller.ts (admin unlock route)
```

**Acceptance Criteria:**
- 10 failed logins → 11th returns 423 immediately (without checking password).
- After 10 min (or admin unlock), login works again with correct credentials.
- Successful login clears failed attempt counter.

---

## E03 — Consent & Compliance Foundation

---

### C18 — Consent API + Consent Gate Middleware

**Epic:** E03 | **Backlog task:** T03.1 | **AI:** ⚠️

**Goal:** `POST /consent`, `GET /consent/status` + NestJS guard implementing 5-case consent gate logic.

**Prerequisites:** C08, C12.

**Implementation Prompt:**
```
[Paste master context block]

Implement the consent system for TaxLens. Compliance-critical — follow 10_COMPLIANCE Section 2 exactly.

1. consent.service.ts:
   grantConsent(userId, consentVersion, ip, userAgent): insert consent_records row (action='granted');
   emit consent.granted; return created record.
   getConsentStatus(userId): latest consent_records row by user_id DESC.
   hasActiveConsent(userId): returns boolean + reason.

2. consent.guard.ts (CanActivate): 5-case logic from 10_COMPLIANCE:
   Case 1: No record → 403 { code: 'NO_ACTIVE_CONSENT' }.
   Case 2: Latest action='withdrawn' → 403 { code: 'CONSENT_WITHDRAWN' }.
   Case 3: Latest='granted' AND version matches CURRENT_CONSENT_VERSION (from ConfigService) → PASS.
   Case 4: Latest='granted' but version outdated → 403 { code: 'CONSENT_REFRESH_REQUIRED' }.
   CURRENT_CONSENT_VERSION from env CONSENT_VERSION default 'v1.0'.

3. @RequiresConsent() decorator — apply to all CS-05 to CS-14 routes.

4. consent.controller.ts:
   POST /consent (auth required, no consent gate): { consent_version }.
   GET /consent/status (auth required): { has_active_consent, consent_version, consented_at }.

5. Error shape: { statusCode: 403, error: 'Forbidden', code: 'CODE', message: 'human readable' }.
```

**Expected Files:**
```
apps/api/src/consent/consent.module.ts
apps/api/src/consent/consent.service.ts
apps/api/src/consent/consent.guard.ts
apps/api/src/consent/consent.controller.ts
apps/api/src/consent/decorators/requires-consent.decorator.ts
apps/api/src/consent/dto/grant-consent.dto.ts
```

**Acceptance Criteria:**
- GET /consent/status with no record returns `{ has_active_consent: false }`.
- @RequiresConsent() endpoint without consent returns `{ code: 'NO_ACTIVE_CONSENT' }`.
- After POST /consent, the endpoint succeeds. Withdrawal blocks with CONSENT_WITHDRAWN.

---

### C19 — Audit Event Service + User Audit API

**Epic:** E03 | **Backlog tasks:** T03.2, T03.4 | **AI:** ✅

**Goal:** `AuditEventService` — central typed service for all audit writes; `GET /audit/me` endpoint.

**Prerequisites:** C08.

**Implementation Prompt:**
```
[Paste master context block]

Implement AuditEventService for TaxLens. Single write path for all audit_events rows.

1. audit/audit-event.types.ts:
   AuditEventPayload interface: { userId?, orgId?, actorRole, action, resourceType?,
   resourceId?, beforeValue?, afterValue?, ipAddress?, userAgent? }.
   Export AUDIT_ACTIONS const with all action strings from 10_COMPLIANCE taxonomy
   (auth.*, consent.*, document.*, extraction.*, computation.*, report.*,
   tax_profile.*, user.*, admin.*) — prevents typos.

2. audit/audit.service.ts:
   write(payload): Promise<void> — inserts audit_events row.
   Before storing: regex-replace PAN patterns in beforeValue/afterValue with '[MASKED_PAN]'.
   Never throws — wraps in try/catch; logs error to console on failure.
   Returns void (callers do not await audit writes).

3. audit/audit.controller.ts:
   GET /audit/me (JWT required): paginated cursor-based; limit default 50 max 200.
   Events for req.user.userId only, ordered by created_at DESC.
   Never return before_value/after_value in user-facing response (admin only).
   Response: { events: [{id, action, resource_type, resource_id, created_at, ip_address}], next_cursor }.

4. Register AuditModule as global module.
```

**Expected Files:**
```
apps/api/src/audit/audit.module.ts
apps/api/src/audit/audit.service.ts
apps/api/src/audit/audit.controller.ts
apps/api/src/audit/audit-event.types.ts
```

**Acceptance Criteria:**
- audit.service.write() failure does not throw or crash the caller.
- GET /audit/me returns only events for the authenticated user, without before/after values.
- PAN in beforeValue stored as [MASKED_PAN] in DB.
- AUDIT_ACTIONS.DOCUMENT_UPLOADED equals 'document.uploaded'.

---

### C20 — ComplianceEventWorkflow (W6)

**Epic:** E03 | **Backlog task:** T03.3 | **AI:** ✅

**Goal:** Temporal workflow W6 — durable audit writer with unlimited retries.

**Prerequisites:** C04, C19.

**Implementation Prompt:**
```
[Paste master context block]

Implement ComplianceEventWorkflow (W6). Reference 07_WORKFLOWS.md W6:
"Retry indefinitely with exponential backoff up to 24h. Audit records must not be lost."

1. apps/api/src/temporal/workflows/compliance-event.workflow.ts:
   complianceEventWorkflow(payload: AuditEventPayload): single activity writeAuditEventActivity.
   Retry policy: { maximumAttempts: 0 (unlimited), initialInterval: '1s',
   maximumInterval: '1h', backoffCoefficient: 2 }.
   If not written after 24h: log CRITICAL error (for ops paging integration).

2. workers/extraction/activities/write_audit_event.py:
   write_audit_event_activity(payload: dict): insert into audit_events via SQLAlchemy.
   Same PAN masking regex as TypeScript service.

3. Add TemporalService.startComplianceEvent(payload) helper:
   starts W6 with workflowId: `compliance/${payload.action}/${Date.now()}`.

Usage guideline (add as comment): use startComplianceEvent() for critical compliance events
(computation.completed, report.generated). Use AuditService.write() directly for synchronous
actions bundled in the same DB transaction.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/compliance-event.workflow.ts
apps/api/src/temporal/temporal.service.ts (updated with startComplianceEvent)
workers/extraction/activities/write_audit_event.py
```

**Acceptance Criteria:**
- W6 with valid payload creates an audit_events row.
- Temporarily stopping DB causes W6 to retry; row written after DB recovers.
- startComplianceEvent() callable from any module.

---

### C21 — Admin Pending Actions Service + Cron

**Epic:** E03 | **Backlog task:** T03.5 | **AI:** ✅

**Goal:** Service for creating, approving, rejecting `admin_pending_actions`; expiry cron.

**Prerequisites:** C09, C12, C16, C19.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin_pending_actions service for TaxLens maker-checker flows.

1. admin/pending-actions.service.ts:
   create(payload: CreatePendingActionDto): insert row; expires_at = now() + 48h;
   emit admin.approval_initiated; return created row.

   approve(actionId, approverId):
   Fetch action; check status='pending'; check not expired.
   Self-approval prevention: if action.initiated_by === approverId → 403 'Cannot approve your own action'.
   Execute action via switch on action_type (dispatch to appropriate service).
   Set status='approved', resolved_by, resolved_at; emit admin.approval_granted.

   reject(actionId, approverId, reason):
   Check pending; check not expired.
   Set status='rejected'; emit admin.approval_rejected.

   expireStale(): UPDATE status='expired' WHERE status='pending' AND expires_at < now().
   Returns count; emits admin.approval_expired per row.

2. DTOs: CreatePendingActionDto, ApprovePendingActionDto (approverId from JWT).

3. Cron: @Cron(EVERY_HOUR) calling expireStale(); log count.
```

**Expected Files:**
```
apps/api/src/admin/pending-actions.service.ts
apps/api/src/admin/dto/pending-action.dto.ts
apps/api/src/admin/admin.module.ts
```

**Acceptance Criteria:**
- Creating a pending action inserts row with expires_at = now() + 48h.
- approve() by initiator throws 403. Expired action cannot be approved (409).
- Cron sets status='expired' on overdue rows.

---

### C22 — MakerCheckerWorkflow (W9)

**Epic:** E03 | **Backlog task:** T03.6 | **AI:** ✅

**Goal:** Temporal workflow W9 orchestrating the full maker-checker approval sequence.

**Prerequisites:** C04, C21.

**Implementation Prompt:**
```
[Paste master context block]

Implement MakerCheckerWorkflow (W9) for TaxLens. Reference STITCH-03 C8/G14.

1. apps/api/src/temporal/workflows/maker-checker.workflow.ts:
   makerCheckerWorkflow(input: { actionType, pendingActionId, initiatedBy, targetResourceId, payload }):

   a. Activity: notifySecondAdminActivity(pendingActionId) — email to all admins except initiator.
   b. Wait for signal 'approvalDecision' ({ decision: 'approved'|'rejected', approverId }) up to 48h
      OR timer: 48h timeout.
   c. On approval: executeActionActivity(actionType, payload) — dispatch to correct service.
   d. On rejection: recordRejectionActivity(pendingActionId).
   e. On timeout: recordExpiredActivity(pendingActionId).
   f. Emit W6 compliance event in all terminal paths.

2. pending-actions.service.ts (update):
   approve() and reject() send 'approvalDecision' signal to the workflow stored in
   pending_action.workflow_id. Add V003b migration for workflow_id column if needed.

3. TemporalService.startMakerCheckerWorkflow(input) helper.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/maker-checker.workflow.ts
apps/api/src/temporal/temporal.service.ts (updated)
apps/api/src/admin/pending-actions.service.ts (updated to signal workflow)
apps/api/src/migrations/V003b__add_workflow_id_to_pending_actions.sql
```

**Acceptance Criteria:**
- Starting W9 creates a workflow visible in Temporal UI waiting for signal.
- Approval signal causes workflow to proceed to execute action.
- Timeout path (tested with short timeout) sets status='expired'.

---

### C23 — Admin Approvals API

**Epic:** E03 | **Backlog task:** T03.7 | **AI:** ✅

**Goal:** `GET /admin/approvals`, `POST /admin/approvals/{id}/approve`, `POST /admin/approvals/{id}/reject`.

**Prerequisites:** C21, C22.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin approvals API endpoints. All routes @Roles('admin').

1. admin/pending-actions.controller.ts:

   GET /admin/approvals: query params status (default 'pending'), action_type.
   Return paginated list ordered by initiated_at ASC.
   Include 'is_own_action' flag (true if pending_action.initiated_by === req.user.userId).
   Response: { pending_actions: [{id, action_type, initiated_by_email (masked), initiated_at,
   expires_at, status, target_resource_type, target_resource_id, is_own_action}] }.

   POST /admin/approvals/:id/approve:
   approverId = req.user.userId. Call pendingActionsService.approve().
   Return 200 { message: 'Action approved and executed.' }.
   Self-approval: 403. Expired/not found: 404/409.

   POST /admin/approvals/:id/reject:
   Body: { reason: string, MinLength(10) }.
   Call pendingActionsService.reject(). Return 200.

2. Register PendingActionsController in AdminModule. Add AdminModule to AppModule.
```

**Expected Files:**
```
apps/api/src/admin/pending-actions.controller.ts
apps/api/src/admin/admin.module.ts (updated)
apps/api/src/app.module.ts (updated)
```

**Acceptance Criteria:**
- GET /admin/approvals with operator role returns 403.
- POST /approve by same initiator returns 403.
- POST /approve by different admin executes and sets status='approved'.

---

## E04 — Document Ingestion

---

### C24 — Pre-Signed Upload URL Endpoint

**Epic:** E04 | **Backlog task:** T04.1 | **AI:** ✅

**Goal:** `POST /documents/upload-url` — validate params, create document records, return S3 pre-signed PUT URL.

**Prerequisites:** C05, C18, C19, C08.

**Implementation Prompt:**
```
[Paste master context block]

Implement the document upload URL endpoint for TaxLens.

1. documents/dto/upload-url.dto.ts:
   document_type: IsEnum(['form_16','ais','form_26as','salary_slip','interest_cert','other']).
   assessment_year: Matches(/^\d{4}-\d{2}$/). file_name: MaxLength(255).
   mime_type: IsEnum(['application/pdf','image/jpeg','image/png','text/csv']).
   file_size_bytes: IsInt, Min(1), Max(20971520).

2. documents.service.ts getUploadUrl(userId, orgId, dto, ipAddress):
   Create documents row OR find existing for same document_type+AY.
   Create document_versions row: version_number = max(existing) + 1; status='pending'.
   Generate S3 pre-signed PUT URL: key='documents/{orgId}/{documentId}/{versionId}.{ext}',
   conditions: Content-Type and Content-Length match dto values, expiry 900s.
   Emit audit event: document.upload_url_issued.
   Return { document_id, document_version_id, upload_url, upload_url_expires_at, fields: {} }.

3. POST /documents/upload-url with @RequiresConsent() in documents.controller.ts.
   Use @aws-sdk/s3-request-presigner.
```

**Expected Files:**
```
apps/api/src/documents/documents.module.ts
apps/api/src/documents/documents.service.ts
apps/api/src/documents/documents.controller.ts
apps/api/src/documents/dto/upload-url.dto.ts
apps/api/src/documents/document.entity.ts
apps/api/src/documents/document-version.entity.ts
```

**Acceptance Criteria:**
- Without active consent: 403 NO_ACTIVE_CONSENT.
- Valid params: returns 200 with pre-signed URL.
- `curl -T sample.pdf "{upload_url}"` successfully uploads to S3.
- Second upload same type+AY creates version_number=2. File > 20 MB returns 422.

---

### C25 — Upload Confirm + Virus Scan Queue

**Epic:** E04 | **Backlog tasks:** T04.2, T04.3 | **AI:** ⚠️

**Goal:** `POST /documents/{version_id}/confirm-upload` enqueues virus scan; BullMQ consumer calls ClamAV.

**Prerequisites:** C24, C03.

**Implementation Prompt:**
```
[Paste master context block]

Implement upload confirmation and virus scanning for TaxLens.

1. documents.service.ts confirmUpload(versionId, userId):
   Verify document_version belongs to userId.
   HeadObjectCommand to verify S3 object exists. Check idempotent (not already confirmed).
   Compute + store checksum_sha256 from S3 ETag.
   Add job to VIRUS_SCAN_QUEUE: { document_version_id }.
   Create extraction_jobs row: status='queued'.
   Emit document.upload_confirmed. Return { extraction_job_id, status: 'queued' }.

2. documents/virus-scan.processor.ts (@Processor for VIRUS_SCAN_QUEUE)):
   Download file from S3 to /tmp (stream). Run ClamAV via TCP socket to CLAMAV_HOST:3310.
   PASS: update virus_scan_status='passed'; emit document.scan_passed; trigger W1 via TemporalService.
   FAIL: status='quarantined'; emit document.quarantined; notify user via email.
   TIMEOUT (3 retries): status='scan_timeout'; alert ops via console.error.
   Clean up /tmp file in finally block.

3. POST /documents/:version_id/confirm-upload in documents.controller.ts.
```

**Expected Files:**
```
apps/api/src/documents/documents.service.ts (updated)
apps/api/src/documents/documents.controller.ts (confirm-upload route)
apps/api/src/documents/virus-scan.processor.ts
apps/api/src/documents/dto/confirm-upload.dto.ts
```

**Acceptance Criteria:**
- POST confirm-upload creates BullMQ job (verify in Redis: `LRANGE bull:virus-scan:wait 0 -1`).
- Clean PDF → virus_scan_status='passed' and extraction_jobs.status='queued'.
- Calling confirm-upload twice is idempotent.

---

### C26 — DocumentIngestionWorkflow (W1)

**Epic:** E04 | **Backlog task:** T04.4 | **AI:** ✅

**Goal:** Temporal W1 — durable status tracking and ExtractionWorkflow (W2) startup after virus scan.

**Prerequisites:** C04, C25.

**Implementation Prompt:**
```
[Paste master context block]

Implement DocumentIngestionWorkflow (W1). Reference 07_WORKFLOWS.md W1.
Note: Virus scan done by BullMQ (C25) before W1. W1 starts after scan passes.

1. apps/api/src/temporal/workflows/document-ingestion.workflow.ts:
   documentIngestionWorkflow({ documentVersionId, extractionJobId }):
   a. updateDocumentStatusActivity({ versionId, status: 'SCAN_PASSED' }).
   b. startExtractionWorkflowActivity({ extractionJobId, documentVersionId }).
   On any failure: updateDocumentStatusActivity(status: 'INGESTION_FAILED'); alert ops.
   WorkflowExecutionTimeout: 5 minutes.

2. Activities (NestJS side):
   updateDocumentStatusActivity(versionId, status): update document_versions.virus_scan_status.
   startExtractionWorkflowActivity(extractionJobId, documentVersionId):
   TemporalService.startWorkflow('extraction', extractionJobId, { extractionJobId, documentVersionId });
   store temporal_workflow_id back to extraction_jobs.

3. WorkflowIDs: W1='documentIngestion/{documentVersionId}', W2='extraction/{extractionJobId}'.
   Retry: 3 attempts, 10s exponential backoff.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/document-ingestion.workflow.ts
apps/api/src/temporal/activities/document-ingestion.activities.ts
```

**Acceptance Criteria:**
- After virus scan passes and W1 starts: extraction_jobs.temporal_workflow_id populated.
- W1 visible in Temporal UI as completed. W2 appears on extraction-queue.

---

### C27 — Document List & Detail API

**Epic:** E04 | **Backlog task:** T04.5 | **AI:** ✅

**Goal:** `GET /documents`, `GET /documents/{id}`, `DELETE /documents/{id}` endpoints.

**Prerequisites:** C24, C19.

**Implementation Prompt:**
```
[Paste master context block]

Implement document list and detail endpoints for TaxLens.

1. documents.service.ts:
   getDocuments(userId, filters: { assessment_year?, document_type? }):
   Query documents JOIN document_versions JOIN extraction_jobs; filter deleted_at IS NULL.
   Include latest_version extraction_status. Return per 06_API.md GET /documents spec.

   getDocument(userId, documentId): document + all versions with scan + extraction status.
   404 if not found or different user.

   softDeleteDocument(userId, documentId): set documents.deleted_at=now();
   emit document.soft_deleted. S3 file NOT deleted (retained for audit).

2. documents.controller.ts:
   GET /documents (requires consent; query params: assessment_year, document_type).
   GET /documents/:id (requires consent).
   DELETE /documents/:id (requires consent).
```

**Expected Files:**
```
apps/api/src/documents/documents.service.ts (updated)
apps/api/src/documents/documents.controller.ts (updated)
apps/api/src/documents/dto/get-documents-query.dto.ts
```

**Acceptance Criteria:**
- GET /documents returns array with correct shape; filters work.
- GET /documents/{id} returns document + versions array.
- DELETE sets deleted_at; document no longer in list.
- Deleting another user's document returns 404 (not 403 — no enumeration).

---

## E05 — OCR Extraction Workers

---

### C28 — OCR Vendor Adapter Interface + AWS Textract

**Epic:** E05 | **Backlog task:** T05.1 | **AI:** ⚠️

**Goal:** Abstract `OCRVendorAdapter` Python class + AWS Textract concrete implementation.

**Prerequisites:** C02, C05.

**Implementation Prompt:**
```
[Paste master context block]

Implement the OCR vendor adapter for TaxLens. Reference 08_EXTRACTION.md:
"Pipeline is vendor-agnostic at the interface level."

1. workers/extraction/ocr/base_adapter.py: Abstract OCRVendorAdapter:
   analyze_document(s3_bucket, s3_key, document_type) -> RawOCRResponse (abstract).
   get_text_blocks(raw_response) -> list[TextBlock] (abstract).
   Pydantic models: TextBlock { text, confidence, bounding_box, page, block_type }.
   RawOCRResponse { vendor, job_id, blocks: list[TextBlock], raw_json_s3_key, analyzed_at }.

2. workers/extraction/ocr/textract_adapter.py:
   Use boto3 textract. For single-page (<5 pages): analyze_document (sync).
   For multi-page: start_document_analysis (async) + poll GetDocumentAnalysis (max 5 min, 5s backoff).
   Convert Textract Blocks to TextBlock list. Store raw JSON to S3 ocr-raw bucket.
   Return RawOCRResponse with raw_json_s3_key. Do NOT hold large raw response in memory after S3 store.

3. ocr/__init__.py: get_ocr_adapter() factory from OCR_VENDOR env var.
```

**Expected Files:**
```
workers/extraction/ocr/base_adapter.py
workers/extraction/ocr/textract_adapter.py
workers/extraction/ocr/__init__.py
workers/extraction/ocr/models.py
```

**Acceptance Criteria:**
- get_ocr_adapter() returns TextractAdapter for OCR_VENDOR='aws_textract'.
- analyze_document on a Form 16 PDF returns RawOCRResponse with blocks populated.
- Raw response saved to S3 ocr-raw bucket. Not held in Python memory after save.

---

### C29 — Form 16 Part A Mapper

**Epic:** E05 | **Backlog task:** T05.2 part A | **AI:** ⚠️

**Goal:** `Form16PartAMapper` — extract all Part A fields with confidence scoring.

**Prerequisites:** C28.

**Implementation Prompt:**
```
[Paste master context block]

Implement Form 16 Part A field mapper. Reference 08_EXTRACTION.md Part A field table.

1. workers/extraction/mappers/base_mapper.py: Abstract BaseMapper:
   map(blocks: list[TextBlock], document_type) -> ExtractionResult (abstract).
   Helper methods: _extract_field(blocks, regex_patterns, field_name) -> (value, confidence).
   _normalize_currency(value) -> float (strip ₹, commas; return float).
   _normalize_pan(value) -> str|None (uppercase, validate AAAAA9999A).
   _normalize_tan(value) -> str|None (validate AAAA99999A).
   _normalize_date(value) -> str|None (YYYY-MM-DD). _normalize_ay(value) -> str|None (YYYY-YY).
   Pydantic ExtractionResult: { document_type, assessment_year, extracted_fields,
   confidence_scores, low_confidence_fields, overall_confidence, source_categories }.
   source_categories: maps each field to 'exact_extraction'|'inferred'|'classified'|'user_confirmed'.

2. workers/extraction/mappers/form16_part_a_mapper.py: Form16PartAMapper extends BaseMapper.
   Must-Have fields with 2-3 regex patterns each:
   employer_tan: r'TAN\s*[:\s]\s*([A-Z]{4}[0-9]{5}[A-Z])' and variants.
   employee_pan: r'PAN\s*[:\s]\s*([A-Z]{5}[0-9]{4}[A-Z])'.
   total_tds_deducted: r'Total\s+TDS\s+[Dd]eposited[^\d]*([\d,]+(?:\.\d{2})?)' and variants.
   assessment_year: r'Assessment\s+Year\s*[:\s]\s*(\d{4}-\d{2})'.
   period_from/to: r'Period[^:]*:\s*(\d{2}/\d{2}/\d{4})\s*to\s*(\d{2}/\d{2}/\d{4})'.
   Confidence scoring: vendor_confidence × heuristic_match boost (per C36 confidence scorer).
   overall_confidence = min(confidence of Must-Have fields). If any Must-Have < 0.5: overall=0.0.
```

**Expected Files:**
```
workers/extraction/mappers/base_mapper.py
workers/extraction/mappers/form16_part_a_mapper.py
workers/extraction/mappers/models.py (ExtractionResult Pydantic model)
workers/extraction/tests/test_form16_part_a_mapper.py
```

**Acceptance Criteria:**
- Mapper returns ExtractionResult with all Must-Have Part A fields populated.
- PAN in AAAAA9999A format. TAN in AAAA99999A. Monetary values as float.
- overall_confidence = weakest Must-Have field (not average).

---

### C30 — Form 16 Part B Mapper

**Epic:** E05 | **Backlog task:** T05.2 part B | **AI:** ⚠️

**Goal:** `Form16PartBMapper` — extract salary income, deductions, tax, TDS from Part B.

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement Form 16 Part B mapper. Reference 08_EXTRACTION.md Part B field table.

workers/extraction/mappers/form16_part_b_mapper.py: Form16PartBMapper.
Must-Have regex patterns:
gross_salary: r'(?:Gross\s+[Ss]alary|Section\s+17\(1\))[^\d]*([\d,]+(?:\.\d{2})?)'.
standard_deduction: r'[Ss]tandard\s+[Dd]eduction[^\d]*([\d,]+(?:\.\d{2})?)'.
taxable_income: r'[Tt]axable\s+[Ii]ncome[^\d]*([\d,]+(?:\.\d{2})?)'.
total_tax_payable: r'[Tt]otal\s+[Tt]ax\s+(?:[Pp]ayable|[Ll]iability)[^\d]*([\d,]+(?:\.\d{2})?)'.
balance_tax_payable: r'[Bb]alance\s+[Tt]ax\s+[Pp]ayable[^\d]*([-\d,]+(?:\.\d{2})?)' — allow negative (refund).

Cross-validation (inferred): expected_taxable = gross_salary - standard_deduction - total_deductions.
If extracted taxable_income differs by > 5%: add reconciliation_warning.
Tag gross_salary, standard_deduction as 'exact_extraction'; derived checks as 'inferred'.
Handle Part A+B in same PDF: detect "PART A" and "PART B" markers; split blocks accordingly.

workers/extraction/mappers/form16_mapper.py: orchestrator calling Part A + Part B mappers;
merges into one ExtractionResult.
```

**Expected Files:**
```
workers/extraction/mappers/form16_part_b_mapper.py
workers/extraction/mappers/form16_mapper.py
workers/extraction/tests/test_form16_part_b_mapper.py
```

**Acceptance Criteria:**
- Merged Form16Mapper returns both Part A and Part B fields in one ExtractionResult.
- Cross-validation warning triggered when Part B taxable_income is inconsistent.
- balance_tax_payable handles negative values (refund scenario).

---

### C31 — AIS Mapper (PDF)

**Epic:** E05 | **Backlog task:** T05.3 PDF | **AI:** ⚠️

**Goal:** `AISPdfMapper` — extract salary income, TDS entries, interest income from AIS PDF.

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement AIS PDF mapper. Reference 08_EXTRACTION.md AIS field table.
Must-Have: pan, assessment_year, salary_income_entries, tds_entries.

workers/extraction/mappers/ais_pdf_mapper.py:
AIS PDFs from IT portal have multi-section layout.
Section identifiers: "Salary", "Interest from savings", "Interest from deposit",
"TDS on salary", "TDS on other than salary".

Extraction strategy:
1. Identify section boundaries by header text patterns.
2. For each Salary entry: { source: deductor_name, amount: float, tds: float }.
3. For each TDS entry: { tan, deductor_name, amount, section }.
   TDS section code: 'TDS on Salary' → '192'; 'TDS on Other' → '194'; others: 'unknown'.
4. Interest: single values under headers.
5. PAN: "PAN: ABCDE1234F" pattern in header area.
Multiple salary entries common (mid-year job change) — extract all as array.
```

**Expected Files:**
```
workers/extraction/mappers/ais_pdf_mapper.py
workers/extraction/tests/test_ais_pdf_mapper.py
```

**Acceptance Criteria:**
- AIS PDF with 2 salary entries returns salary_income_entries of length 2.
- TDS entries array includes all deductors. overall_confidence reflects pan and salary entry quality.

---

### C32 — AIS Mapper (CSV) + Dispatcher

**Epic:** E05 | **Backlog task:** T05.3 CSV | **AI:** ⚠️

**Goal:** `AISCsvMapper` for IT portal CSV export; `AISMapper` dispatcher routing PDF vs CSV.

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement AIS CSV mapper and dispatcher.

1. workers/extraction/mappers/ais_csv_mapper.py:
   Parse with Python csv module. Expected columns (IT portal AIS CSV FY 2024-25):
   'Information Category', 'Information Description', 'Amount (Rs.)', 'TDS/TCS Amount (Rs.)',
   'Year', 'Deductor/Collector TAN/PAN', 'Deductor/Collector Name'.
   If columns don't match: overall_confidence=0.0, add 'AIS_CSV_SCHEMA_MISMATCH' warning.
   Map Salary rows → salary_income_entries.
   Interest rows → appropriate interest bucket.
   TDS rows → tds_entries.
   CSV has no OCR uncertainty: base confidence = 0.95 for all fields from valid CSV.

2. workers/extraction/mappers/ais_mapper.py (dispatcher):
   Detect file type: PDF (magic bytes %PDF) or CSV (text with commas).
   Route to AISPdfMapper or AISCsvMapper accordingly.
   Return same ExtractionResult shape from both.
```

**Expected Files:**
```
workers/extraction/mappers/ais_csv_mapper.py
workers/extraction/mappers/ais_mapper.py
workers/extraction/tests/test_ais_csv_mapper.py
```

**Acceptance Criteria:**
- Valid AIS CSV returns overall_confidence >= 0.90.
- CSV with wrong column names returns overall_confidence=0.0 with schema mismatch warning.
- AISMapper auto-detects PDF vs CSV and routes accordingly.

---

### C33 — Form 26AS Mapper

**Epic:** E05 | **Backlog task:** T05.4 | **AI:** ⚠️

**Goal:** `Form26ASMapper` — Part A (salary TDS), Part B (non-salary), Part C (advance tax).

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement Form 26AS mapper. Reference 08_EXTRACTION.md Form 26AS field table.
Must-Have: pan, assessment_year, tds_entries_part_a.

workers/extraction/mappers/form26as_mapper.py:
Form 26AS structure — split blocks by PART markers.
Part A: salary TDS. Columns: Deductor Name | TAN | Amount Paid | Tax Deducted | TDS Deposited.
Part B: non-salary TDS. Part C: advance tax/self-assessment.

For Part A/B rows: extract using column-position heuristics with TAN pattern as anchor.
Part C: challan-based entries { bsr_code, date, challan_serial, amount }.

Cross-document reconciliation hint (with Form 16):
After extracting both Form 16 and 26AS: compare Form16 total_tds_deducted vs sum of 26AS Part A.
If difference > ₹1000: add reconciliation_warning: 'TDS_MISMATCH_FORM16_VS_26AS'.
Store in ExtractionResult; displayed to user in review UI.

Must-Have failure: if Part A has zero entries → overall_confidence = 0.0.
```

**Expected Files:**
```
workers/extraction/mappers/form26as_mapper.py
workers/extraction/tests/test_form26as_mapper.py
```

**Acceptance Criteria:**
- Form 26AS with 2 deductors in Part A returns 2 entries in tds_entries_part_a.
- TAN values validated against pattern.
- Reconciliation warning generated when Form 16 TDS ≠ 26AS TDS.

---

### C34 — Salary Slip Mapper

**Epic:** E05 | **Backlog task:** T05.5 | **AI:** ⚠️

**Goal:** `SalarySlipMapper` — layout-agnostic extraction handling Zoho, Keka, generic formats.

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement salary slip mapper. Reference 08_EXTRACTION.md salary slip field table.
Must-Have: month, employer_name, employee_name, gross_salary, net_pay.
Format challenge: layouts vary widely by payroll vendor.

workers/extraction/mappers/salary_slip_mapper.py:
Label proximity search (not column positions):
For each field, define label synonyms:
gross_salary: ['Gross Salary', 'Gross Earnings', 'Total Gross', 'Gross CTC Earned'].
net_pay: ['Net Pay', 'Net Salary', 'Take Home', 'Net Amount'].
basic_salary: ['Basic', 'Basic Salary', 'Basic Pay'].
hra_component: ['HRA', 'House Rent Allowance'].
pf_deduction: ['PF', 'Provident Fund', 'EPF Employee'].

Find label in blocks; search nearby blocks (same row or next row) for monetary value.
Month detection: month name + year pattern near top ('April 2025', 'Mar-2025', '2025-03').
Normalize month to YYYY-MM string.

Format detection heuristics:
Zoho: 'ZOHO' watermark or URL. Keka: 'Keka HR' or keka.com. GreytHR: 'Greytip'.
Generic: fall back to label proximity search.

overall_confidence = min(gross_salary_conf, net_pay_conf, month_conf).
If gross_salary == net_pay: add warning 'SALARY_SLIP_POSSIBLY_INCOMPLETE'.
```

**Expected Files:**
```
workers/extraction/mappers/salary_slip_mapper.py
workers/extraction/tests/test_salary_slip_mapper.py
```

**Acceptance Criteria:**
- Mapper extracts gross_salary and net_pay from at least 3 different payroll PDF formats.
- Month extracted as YYYY-MM string. Warning generated when gross == net.

---

### C35 — Interest Certificate Mapper

**Epic:** E05 | **Backlog task:** T05.6 | **AI:** ✅

**Goal:** `InterestCertMapper` — extract bank name, account type, interest amount, TDS.

**Prerequisites:** C29.

**Implementation Prompt:**
```
[Paste master context block]

Implement interest certificate mapper. Reference 08_EXTRACTION.md interest cert field table.
Must-Have: bank_name, account_type, interest_amount.

workers/extraction/mappers/interest_cert_mapper.py:
bank_name: prominent in header; common Indian banks list as hints:
SBI, HDFC, ICICI, Axis, Kotak, PNB, BoB, Canara, Union Bank, IDBI, IndusInd, Federal.
account_type: 'Savings Account'|'Fixed Deposit'|'FD'|'SB A/c' etc. → normalize to 'savings'|'fd'|'rd'.
interest_amount: labels 'Interest paid/credited', 'Total Interest', 'Net Interest' + currency value.
tds_deducted: 'TDS Deducted', 'Tax Deducted at Source' + value; 0.0 if pan_linked=true
and interest < 40000/50000 threshold.
period_from/to: date range in statement period.
pan_linked: detect 'PAN: XXXXXX' or 'PAN linked: Yes/No'.
Expect overall_confidence > 0.85 for digital bank PDFs.
```

**Expected Files:**
```
workers/extraction/mappers/interest_cert_mapper.py
workers/extraction/tests/test_interest_cert_mapper.py
```

**Acceptance Criteria:**
- Extracts bank_name, account_type='fd', interest_amount, tds_deducted from sample FD certificate.
- account_type normalized to lowercase. interest_amount returned as float.

---

### C36 — ExtractionWorkflow (W2) + Confidence Evaluation

**Epic:** E05 | **Backlog tasks:** T05.7, T05.8 | **AI:** ✅

**Goal:** Python Temporal W2 — OCR → map → score → store → trigger human review.

**Prerequisites:** C28–C35, C04.

**Implementation Prompt:**
```
[Paste master context block]

Implement ExtractionWorkflow (W2). Reference 07_WORKFLOWS.md W2 exactly.

1. workers/extraction/workflows/extraction_workflow.py:
@workflow.defn class ExtractionWorkflow:
  @workflow.run async def run(self, input: ExtractionWorkflowInput):
    await update_extraction_job_status(job_id, 'processing')
    doc_metadata = await workflow.execute_activity(fetch_document_metadata, ...)
    ocr_result = await workflow.execute_activity(call_ocr_vendor, {s3_key, document_type}, ...)
    extraction_result = await workflow.execute_activity(map_fields, {ocr_result, document_type}, ...)
    result_id = await workflow.execute_activity(store_extraction_result, extraction_result, ...)
    status = await workflow.execute_activity(evaluate_and_update_confidence, {result_id, extraction_result}, ...)
    await workflow.execute_activity(trigger_compliance_event, {action: 'extraction.completed', result_id}, ...)

2. Activities:
   fetch_document_metadata: returns s3_bucket, s3_key from extraction_jobs.
   call_ocr_vendor: instantiates adapter via get_ocr_adapter(); returns RawOCRResponse.
   map_fields: routes to correct mapper by document_type; returns ExtractionResult JSON.
   store_extraction_result: inserts extraction_results; updates extraction_jobs.completed_at.
   evaluate_and_update_confidence: applies thresholds per 08_EXTRACTION; updates status;
   if NEEDS_REVIEW: starts HumanReviewNotificationWorkflow (W3) child workflow.

Large payload rule: never put OCR raw response in Temporal history. Pass s3_key only.
Retry policies per 07_WORKFLOWS retry table.
```

**Expected Files:**
```
workers/extraction/workflows/extraction_workflow.py
workers/extraction/activities/fetch_document.py
workers/extraction/activities/call_ocr_vendor.py
workers/extraction/activities/map_fields.py
workers/extraction/activities/store_extraction_result.py
workers/extraction/activities/evaluate_confidence.py
workers/extraction/workflows/input_models.py
```

**Acceptance Criteria:**
- W2 completes end-to-end for Form 16 PDF: extraction_jobs.status='completed' + result row created.
- status='needs_review' when Must-Have field 0.50–0.75. status='failed' when < 0.50.
- OCR raw response in S3 ocr-raw bucket; NOT in DB or Temporal history.

---

### C37 — HumanReview + ExtractionRetry Workflows (W3, W8) + Extraction API

**Epic:** E05/E06 | **Backlog tasks:** T05.9–T05.11, T06.1–T06.4 | **AI:** ✅

**Goal:** W3 (notify + 72h wait for user review), W8 (escalate failures), all extraction API endpoints.

**Prerequisites:** C36.

**Implementation Prompt:**
```
[Paste master context block]

Implement W3, W8, and the complete extraction API. Reference 07_WORKFLOWS.md W3 and W8.

1. workers/extraction/workflows/human_review_workflow.py (W3):
   a. notify_user(user_id, 'review_required', { fields: low_confidence_fields }) — email.
   b. Timer: await workflow.sleep(timedelta(hours=72)).
   c. Signal 'extraction_locked': if received before timer → complete; emit extraction.review_completed.
   d. Timer fires first: send_reminder(user_id); wait further 48h.
   e. No action: emit extraction.review_abandoned; complete (non-blocking).

2. workers/extraction/workflows/extraction_retry_workflow.py (W8):
   a. update_extraction_job_status(job_id, 'failed').
   b. create_ops_alert(job_id, failure_reason) — insert into ops_alerts table.
   c. notify_user(user_id, 'extraction_failed').
   d. Emit extraction.failed.

3. apps/api/src/extraction/extraction.service.ts:
   getJobStatus(userId, jobId): return extraction_jobs row (verify user ownership via document chain).
   getExtractionResult(userId, jobId): return extracted_fields with corrections merged over originals.
   submitCorrections(userId, resultId, corrections[]): upsert extraction_corrections; emit extraction.field_corrected.
   lockExtraction(userId, resultId):
   - Check not locked; check ownership.
   - Check: no Must-Have field with confidence < 0.50 AND no correction applied → 422.
   - Insert extraction_locks row.
   - Send 'extraction_locked' signal to W3 via TemporalService.
   - Emit extraction.locked. Return { locked_at, message }.
   Cross-doc reconciliation: in getExtractionResult(), if both Form16 + 26AS present same AY,
   compute TDS mismatch and include reconciliation_warnings[].

4. extraction.controller.ts (all @RequiresConsent()):
   GET /extractions/:extraction_job_id, GET /extractions/:extraction_job_id/result,
   PATCH /extractions/:result_id/corrections, POST /extractions/:result_id/lock.
```

**Expected Files:**
```
workers/extraction/workflows/human_review_workflow.py
workers/extraction/workflows/extraction_retry_workflow.py
apps/api/src/extraction/extraction.module.ts
apps/api/src/extraction/extraction.service.ts
apps/api/src/extraction/extraction.controller.ts
apps/api/src/extraction/dto/submit-corrections.dto.ts
apps/api/src/extraction/extraction-job.entity.ts + extraction-result.entity.ts
apps/api/src/extraction/extraction-correction.entity.ts + extraction-lock.entity.ts
```

**Acceptance Criteria:**
- GET result returns corrections merged over original extracted_fields.
- PATCH corrections on locked result returns 409.
- POST lock with Must-Have at 0.3 confidence, no correction → 422.
- POST lock success sends signal to W3 (W3 completes in Temporal UI).
- W8 updates status to 'failed' and creates ops alert record.

---

## E06 — Extraction Review UI

---

### C38 — Extraction Review Screen (Part 1: Layout + PDF Viewer + Field List)

**Epic:** E06 | **Backlog tasks:** T06.1, T06.2 | **AI:** ⚠️

**Goal:** Side-by-side PDF viewer and extraction field list panel with confidence bars.

**Prerequisites:** C37 (API), C27 (document list).

**Implementation Prompt:**
```
[Paste master context block]

Implement extraction review screen Part 1 for TaxLens. Reference 11_FRONTEND.md S-08.

1. Install PDF.js: pnpm --filter web add pdfjs-dist.
   apps/web/src/components/extraction/PdfViewer.tsx:
   Renders PDF from pre-signed URL. Page navigation and zoom controls.
   On mobile: component fills full width (toggle mode from parent).

2. apps/web/src/components/extraction/FieldListPanel.tsx:
   Props: extractedFields, confidenceScores, lowConfidenceFields, corrections, isLocked, sourceCategories.
   For each field:
   - Field label (human-readable mapping from field_name).
   - Value display (merged: corrected value if correction exists, else extracted).
   - Confidence bar: green >= 0.75, yellow 0.50–0.74, red < 0.50.
   - 'Low confidence' chip if in lowConfidenceFields[].
   - Source chip: 'Extracted' | 'Computed' | 'User-entered' from sourceCategories.
   - Edit button (calls onEditField prop).
   Section headers per document section (Part A / Part B for Form 16).

3. apps/web/src/app/(app)/extraction/[extraction_job_id]/page.tsx:
   Fetch extraction result via GET /extractions/:id/result.
   Desktop: side-by-side layout (PdfViewer left, FieldListPanel right).
   Mobile: single column with "View Document" / "Review Fields" toggle button.
   Display compliance banner D-05.
   Show TDS reconciliation warning if reconciliation_warnings[] present (orange banner).
```

**Expected Files:**
```
apps/web/src/components/extraction/PdfViewer.tsx
apps/web/src/components/extraction/FieldListPanel.tsx
apps/web/src/app/(app)/extraction/[extraction_job_id]/page.tsx
apps/web/src/lib/field-labels.ts (human-readable field name mapping)
```

**Acceptance Criteria:**
- PDF renders inline with page navigation.
- Low-confidence fields have yellow background + chip.
- Source chips display correctly for each field.
- TDS mismatch orange banner appears when reconciliation_warnings present.

---

### C39 — Extraction Review Screen (Part 2: Editing, Lock, History)

**Epic:** E06 | **Backlog tasks:** T06.3, T06.4, T06.5, T06.6 | **AI:** ✅

**Goal:** Inline field editing, correction history drawer, lock flow, TDS warning UI.

**Prerequisites:** C38.

**Implementation Prompt:**
```
[Paste master context block]

Implement extraction review Part 2 for TaxLens.

1. apps/web/src/components/extraction/FieldEditor.tsx:
   Inline edit mode: click field value → input appears; Save/Cancel buttons; note input (optional).
   On Save: PATCH /extractions/:result_id/corrections with { field_name, corrected_value, note }.
   Optimistic update: show corrected value immediately; revert on error.

2. apps/web/src/components/extraction/CorrectionHistoryDrawer.tsx:
   Side drawer (right slide-in). Per field: list corrections with original_value → corrected_value,
   timestamp, note. Triggered by clicking a "History" icon on each field row.

3. Lock Extraction CTA:
   Sticky bottom bar on extraction review page.
   Button: "Lock Extraction → Enable Computation".
   Disabled when: any Must-Have field has confidence < 0.50 AND no correction applied.
   Tooltip on disabled state: lists unresolved fields by name.
   On click: confirmation modal "Once locked, this extraction cannot be edited."
   → POST /extractions/:result_id/lock → redirect to /tax-profile/:ay.

4. HumanReviewNotificationWorkflow (W3) backend trigger:
   Already handled in C37 (W2 triggers W3 when status=NEEDS_REVIEW).
   Frontend: on page load, if extraction_status='needs_review', show yellow banner:
   "Review required — X fields need attention before you can compute your tax estimate."
```

**Expected Files:**
```
apps/web/src/components/extraction/FieldEditor.tsx
apps/web/src/components/extraction/CorrectionHistoryDrawer.tsx
apps/web/src/components/extraction/LockExtractionBar.tsx
apps/web/src/app/(app)/extraction/[extraction_job_id]/page.tsx (updated)
```

**Acceptance Criteria:**
- Editing a field updates the value inline and calls PATCH API.
- Correction history drawer shows before/after values with timestamps.
- Lock button disabled when unresolved low-confidence Must-Have field exists.
- Lock confirmation → POST lock → redirect to tax profile page.

---

## E07 — Tax Profile & Deduction Inputs

---

### C40 — Tax Profile CRUD + Deduction Inputs API

**Epic:** E07 | **Backlog tasks:** T07.1, T07.2 | **AI:** ✅

**Goal:** Full tax profile API: create/get profile; add/delete deductions.

**Prerequisites:** C37, C18, C19.

**Implementation Prompt:**
```
[Paste master context block]

Implement tax profile and deduction inputs API. Reference 06_API.md tax profile endpoints.

1. tax-profile/tax-profile.service.ts:
   createOrUpdateProfile(userId, dto: { assessment_year, regime_preference, residential_status }):
   Upsert (UNIQUE user_id+AY). In v1: residential_status='nri' → 422.
   Emit tax_profile.created or tax_profile.updated.

   addDeduction(userId, profileId, dto: { section_code, sub_item?, amount, source, notes? }):
   Validate section_code from PERMITTED_SECTION_CODES:
   ['80C','80D_SELF','80D_PARENTS','80TTA','80TTB','80G','80CCD1B','80CCD2','HRA','PROFESSIONAL_TAX','LTA'].
   amount > 0. section_code='80CCD2': source must be 'extracted' (not user-entered).
   Insert deduction_inputs. Emit tax_profile.deduction_added.

   deleteDeduction(userId, deductionId): set deleted_at; emit tax_profile.deduction_removed.

2. Routes (all @RequiresConsent()): POST /tax-profiles, GET /tax-profiles/:assessment_year,
   POST /tax-profiles/:assessment_year/deductions, DELETE /tax-profiles/:ay/deductions/:id.

3. Export PERMITTED_SECTION_CODES constant (imported by tax engine in C43).
```

**Expected Files:**
```
apps/api/src/tax-profile/tax-profile.module.ts
apps/api/src/tax-profile/tax-profile.service.ts
apps/api/src/tax-profile/tax-profile.controller.ts
apps/api/src/tax-profile/dto/create-tax-profile.dto.ts + add-deduction.dto.ts
apps/api/src/tax-profile/constants/permitted-section-codes.ts
apps/api/src/tax-profile/tax-profile.entity.ts + deduction-input.entity.ts
```

**Acceptance Criteria:**
- POST /tax-profiles with residential_status='nri' returns 422.
- POST .../deductions with invalid section_code returns 422.
- POST .../deductions with 80CCD2 and source='user_input' returns 422.
- GET returns profile with deduction_inputs array.

---

### C41 — Tax Rules Seed Data (AY 2025-26)

**Epic:** E07 | **Backlog task:** T07.3 | **AI:** 🔒

**Goal:** Define and seed both regime rule objects for AY 2025-26 per Finance Act 2024.

**Prerequisites:** C08.

**Implementation Prompt:**
```
[Paste master context block]

Define and seed tax rule data for AY 2025-26. Reference 09_TAX_ENGINE.md Section 2 (rule object structure).
CRITICAL: Validate all rule values against Finance Act 2024 before committing. Human review required.

Create apps/api/src/tax-rules/seeds/ay-2025-26.seed.ts with TWO rule objects:

1. OLD REGIME (AY 2025-26):
   standard_deduction: 50000, basic_exemption_limit: 250000.
   rebate_87a: { applicable_income_limit: 500000, max_rebate: 12500 }.
   tax_slabs: [0-250000@0%, 250001-500000@5%, 500001-1000000@20%, 1000001+@30%].
   surcharge: [50L-1Cr@10%, 1-2Cr@15%, 2-5Cr@25%, 5Cr+@37%].
   cess_rate: 0.04.
   permitted_deductions: ['standard_deduction','80C','80D','HRA','80TTA','80TTB','80G',
   '80CCD1B','80CCD2','professional_tax','LTA'].

2. NEW REGIME (AY 2025-26, default from FY 2024-25):
   standard_deduction: 75000, basic_exemption_limit: 300000.
   rebate_87a: { applicable_income_limit: 1200000, max_rebate: 60000 }.
   tax_slabs: [0-300000@0%, 300001-700000@5%, 700001-1000000@10%, 1000001-1200000@15%,
   1200001-1500000@20%, 1500001+@30%].
   Same surcharge table. cess_rate: 0.04.
   permitted_deductions: ['standard_deduction','nps_employer_contribution_80ccd2'].
   disallowed_deductions: ['80C','80D','HRA','80TTA','80G','80CCD1B'].

Create a seed script (apps/api/src/tax-rules/seeds/run-seed.ts) that calls
POST /admin/tax-rules for both objects (bypasses maker-checker for seed only).
Document: "Seed script for initial deployment. Verify rule values against CBDT circular
before running in production."
```

**Expected Files:**
```
apps/api/src/tax-rules/seeds/ay-2025-26.seed.ts
apps/api/src/tax-rules/seeds/run-seed.ts
apps/api/src/tax-rules/tax-rules.service.ts (basic CRUD for rule management)
apps/api/src/tax-rules/tax-rules.entity.ts
```

**Acceptance Criteria:**
- Seed script creates 2 rows in tax_rules (one per regime, both is_active=true).
- `SELECT * FROM tax_rules` returns AY 2025-26 old and new with correct slab structures.
- Rule JSON validates against 09_TAX_ENGINE Section 2 rule object schema.

---

### C42 — Tax Rules Admin API

**Epic:** E07 | **Backlog task:** T07.4 | **AI:** ✅

**Goal:** Admin endpoints to list, create, and activate versioned tax rule configurations.

**Prerequisites:** C41, C23.

**Implementation Prompt:**
```
[Paste master context block]

Implement tax rules admin API. All routes @Roles('admin').

1. tax-rules/tax-rules.service.ts (update):
   listRules(filters): return all tax_rules rows with is_active flag.
   createDraftRule(dto: { assessment_year, regime, version, rules_json, effective_from, notes }):
   Insert with is_active=false. Emit admin.tax_rule_created.
   Return { rule_id, message: 'Rule draft created. Requires admin approval to activate.' }.

   activateRule(ruleId, approverId): (called by MakerCheckerWorkflow executeActionActivity)
   Set is_active=true on new rule. Set is_active=false on previous active rule for same (AY, regime).
   Emit admin.tax_rule_activated.

2. tax-rules/tax-rules.controller.ts:
   GET /admin/tax-rules: list all with filters.
   POST /admin/tax-rules: create draft → startMakerCheckerWorkflow for 'activate_tax_rule'.
   Returns 202 { pending_action_id, message: 'Draft created. Awaiting second admin approval.' }.
```

**Expected Files:**
```
apps/api/src/tax-rules/tax-rules.service.ts (updated)
apps/api/src/tax-rules/tax-rules.controller.ts
apps/api/src/admin/admin.module.ts (updated with TaxRulesController)
```

**Acceptance Criteria:**
- GET /admin/tax-rules returns all rule versions.
- POST /admin/tax-rules creates inactive draft + pending approval.
- Approval activates new rule and deactivates previous active rule for same (AY, regime).

---

## E08 — Tax Engine

---

### C43 — AssembleComputationInputs + Slab Computation

**Epic:** E08 | **Backlog tasks:** T08.1, T08.2 | **AI:** ⚠️

**Goal:** Assemble `ComputationInput` from locked extractions + tax profile; implement slab-wise tax calculation.

**Prerequisites:** C37, C40, C41.

**Implementation Prompt:**
```
[Paste master context block]

Implement computation input assembly and slab tax calculation. Reference 09_TAX_ENGINE.md Sections 3 and 5.

1. computation/computation.service.ts — assembleInputs(userId, assessmentYear, regime):
   Validate: at least one locked extraction exists for given AY. Active tax profile exists.
   Collect all locked ExtractionResults for user+AY. Merge deduction_inputs from tax profile.
   Produce ComputationInput typed object (per 09_TAX_ENGINE Section 3).
   Validate: gross_salary > 0, regime is 'old'|'new', residential_status = 'resident' (v1).
   Load active tax_rules for (assessmentYear, regime). Record rule_version.
   Store assembled object as input_snapshot.

2. computation/tax-engine/slab-calculator.ts:
   computeSlabTax(taxableIncome: number, slabs: SlabRule[]): { slabBreakdown: SlabBreakdownItem[], taxBeforeRebate: number }.
   Iterate slabs from rules JSON; compute tax per slab; build slab_breakdown[] per 09_TAX_ENGINE Section 5.
   Example for ₹12,50,000 new regime: returns exactly the breakdown shown in the spec.
   Each SlabBreakdownItem: { slab_label, taxable_in_slab, rate, tax }.
```

**Expected Files:**
```
apps/api/src/computation/computation.module.ts
apps/api/src/computation/computation.service.ts
apps/api/src/computation/tax-engine/slab-calculator.ts
apps/api/src/computation/dto/trigger-computation.dto.ts
```

**Acceptance Criteria:**
- assembleInputs() fails with 400 if no locked extraction for AY.
- computeSlabTax(1250000, newRegimeSlabs) returns slab_breakdown matching 09_TAX_ENGINE Section 5 example exactly.
- input_snapshot stored as JSONB with all required fields.

---

### C44 — Deduction Application (Old + New Regime)

**Epic:** E08 | **Backlog tasks:** T08.3, T08.4 | **AI:** ⚠️

**Goal:** Apply all deductions for old regime; filter and annotate for new regime.

**Prerequisites:** C43.

**Implementation Prompt:**
```
[Paste master context block]

Implement deduction application for TaxLens tax engine. Reference 09_TAX_ENGINE.md Section 3 (input) and Section 4 (output deductions_applied).

1. computation/tax-engine/deduction-calculator.ts:

applyDeductions(deductions: ComputationInput['deductions'], regime: 'old'|'new', rules: TaxRules):
  { deductionsApplied: DeductionsApplied, notes: string[] }

Old regime: apply all deduction categories with statutory caps:
- 80C: cap at Math.min(input.section_80c, 150000); if capped: add note '80C capped at ₹1,50,000'.
- 80D_SELF: cap at 25000 (non-senior) per rules; note if capped.
- 80D_PARENTS: cap at 25000 (non-senior); note if capped.
- 80TTA: cap at 10000 (savings interest). If 80TTB applicable (senior): use 80TTB cap 50000 instead.
- 80G: apply 50% or 100% sub-limit; for v1: treat all 80G as 50% with no absolute limit.
- 80CCD1B: cap at 50000; add note if capped.
- HRA: use input.hra_deduction as-is (already computed by user; engine trusts it).
- professional_tax: use actual (no cap).
total_deductions = sum of all applied amounts.

New regime: for each deduction input:
- If section_code in rules.permitted_deductions: apply normally.
- Else: set amount to 0; add to notes[] 'Ignored: {section_code} (not permitted under New Regime)'.
Only standard_deduction (75000) and 80CCD2 apply in new regime.
```

**Expected Files:**
```
apps/api/src/computation/tax-engine/deduction-calculator.ts
apps/api/src/computation/tax-engine/types.ts (DeductionsApplied, SlabBreakdownItem interfaces)
```

**Acceptance Criteria:**
- 80C input of 200000 → applied as 150000 with note '80C capped at ₹1,50,000'.
- New regime: 80C input → applied as 0 with 'Ignored: 80C (not permitted under New Regime)' note.
- All deduction caps come from rules JSON, not hardcoded values.

---

### C45 — Surcharge + Marginal Relief

**Epic:** E08 | **Backlog task:** T08.5 | **AI:** ⚠️

**Goal:** Compute surcharge with marginal relief for incomes near threshold. Reference STITCH-02 G10.

**Prerequisites:** C43.

**Implementation Prompt:**
```
[Paste master context block]

Implement surcharge and marginal relief for TaxLens tax engine. Reference 09_TAX_ENGINE.md Section 5.

computation/tax-engine/surcharge-calculator.ts:

computeSurcharge(totalIncome: number, taxBeforeRebate: number, surchargeRules: SurchargeRule[]):
  { surcharge: number, marginalRelief: number }

Step 1: Find applicable surcharge rate from rules:
For each surcharge band: if totalIncome > income_above (and ≤ income_upto if not null): apply rate.
surcharge = taxBeforeRebate * rate.

Step 2: Marginal relief check:
If surcharge > 0 AND totalIncome is within X% of the surcharge threshold it crossed:
Compute tax at the lower threshold (no surcharge) + the incremental income.
If surcharge > (totalIncome - threshold): apply marginal relief.
marginalRelief = Math.max(0, surcharge - (totalIncome - threshold)).
Final surcharge = surcharge - marginalRelief.

Example: totalIncome=50,10,000. Threshold 50,00,000. Surcharge at 10% on tax.
If surcharge > ₹10,000 (the extra income): marginal relief applies.

Note for new regime AY 2025-26: surcharge rate capped at 25% even for highest band.
```

**Expected Files:**
```
apps/api/src/computation/tax-engine/surcharge-calculator.ts
apps/api/src/computation/tax-engine/surcharge-calculator.spec.ts
```

**Acceptance Criteria:**
- Income of ₹50L: no surcharge. Income of ₹50.1L: surcharge computed with marginal relief.
- Marginal relief reduces final surcharge so total tax increase ≤ incremental income above threshold.
- surcharge_marginal_relief field in output matches computed value.

---

### C46 — Section 87A Rebate + TDS Reconciliation

**Epic:** E08 | **Backlog tasks:** T08.6, T08.7 | **AI:** ✅

**Goal:** 87A rebate logic; TDS reconciliation to produce net payable/refundable.

**Prerequisites:** C43.

**Implementation Prompt:**
```
[Paste master context block]

Implement 87A rebate and TDS reconciliation for TaxLens tax engine.

1. computation/tax-engine/rebate-calculator.ts:
computeRebate(taxableIncome: number, taxBeforeRebate: number, rebateRule: RebateRule):
  { rebateApplied: number, notes: string }
If taxableIncome <= rebateRule.applicable_income_limit:
  rebateApplied = Math.min(taxBeforeRebate, rebateRule.max_rebate).
  Note: 'Section 87A rebate applied: ₹{rebateApplied}'.
Else: rebateApplied = 0; note: 'Section 87A rebate not applicable (income > ₹{limit})'.

2. computation/tax-engine/tds-reconciler.ts:
reconcileTDS(totalTaxLiability: number, tdsCredits: TDSCredit[], advanceTax: number, selfAssessmentTax: number):
  TDSReconciliation
totalTaxesPaid = sum(tdsCredits.amount) + advanceTax + selfAssessmentTax.
netPayable = Math.max(0, totalTaxLiability - totalTaxesPaid).
netRefundable = Math.max(0, totalTaxesPaid - totalTaxLiability).
reconciliationWarnings: check for TDS credits with unknown TAN (flag 'UNKNOWN_TAN_DEDUCTOR');
check total tdsCredits from form_26as vs ais if both available (flag if mismatch > 1000).
```

**Expected Files:**
```
apps/api/src/computation/tax-engine/rebate-calculator.ts
apps/api/src/computation/tax-engine/tds-reconciler.ts
```

**Acceptance Criteria:**
- taxableIncome=400000 (old regime): rebate=12500 applied.
- taxableIncome=510000: rebate=0.
- TDS of 100000 against tax of 80000: netRefundable=20000, netPayable=0.

---

### C47 — ComputationResult Assembly + Storage

**Epic:** E08 | **Backlog task:** T08.8 | **AI:** ✅

**Goal:** Assemble full `ComputationResult` object; validate; store as immutable `computation_runs` row.

**Prerequisites:** C43–C46.

**Implementation Prompt:**
```
[Paste master context block]

Implement computation result assembly for TaxLens. Reference 09_TAX_ENGINE.md Section 4 (ComputationResult).

computation/computation.service.ts — runComputation(userId, assessmentYear, regime):
1. Call assembleInputs() → ComputationInput + rule_version.
2. computeSlabTax(taxableIncome) → slab_breakdown, tax_before_rebate.
3. applyDeductions() → deductions_applied, notes.
4. computeSurcharge() → surcharge, marginal_relief.
5. computeRebate() → rebate_87a_applied.
6. tax_after_rebate = tax_before_rebate - rebate_87a_applied.
7. cess = (tax_after_rebate + surcharge) * rules.cess_rate.
8. total_tax_liability = tax_after_rebate + surcharge + cess.
9. reconcileTDS() → tds_reconciliation.
10. Assemble ComputationResult: all fields from 09_TAX_ENGINE Section 4.
    disclaimer: 'ESTIMATE ONLY. NOT A LEGAL FILING.' (hardcoded; cannot be omitted).
11. Store as computation_runs row: immutable; includes input_snapshot (full ComputationInput JSON)
    and result (full ComputationResult JSON). status='completed'.
12. Emit W6 compliance event: computation.completed.
13. Return computation_run_id.

All monetary values in INR (numbers, not strings). All values positive (refund shown as netRefundable).
```

**Expected Files:**
```
apps/api/src/computation/computation.service.ts (updated with runComputation)
apps/api/src/computation/computation-run.entity.ts
```

**Acceptance Criteria:**
- runComputation() produces a ComputationResult with all 09_TAX_ENGINE Section 4 fields.
- Disclaimer field always present in stored result JSON.
- Input snapshot stored in computation_runs.input_snapshot (verify via psql).

---

### C48 — ComputationWorkflow (W4)

**Epic:** E08 | **Backlog task:** T08.9 | **AI:** ✅

**Goal:** Temporal W4 — durable orchestration of the computation pipeline; 'compare' mode.

**Prerequisites:** C04, C47.

**Implementation Prompt:**
```
[Paste master context block]

Implement ComputationWorkflow (W4). Reference 07_WORKFLOWS.md W4 and STITCH-02 C2.

apps/api/src/temporal/workflows/computation.workflow.ts:
computationWorkflow({ userId, assessmentYear, regime: 'old'|'new'|'compare' }):

If regime='compare': start TWO W4 runs in parallel (one for 'old', one for 'new');
return both computation_run_ids. This satisfies STITCH-02 C2.

For each single regime run:
a. Activity: validateComputationPrerequisitesActivity(userId, assessmentYear).
b. Activity: runTaxComputationActivity(userId, assessmentYear, regime) → computation_run_id.
   (calls computationService.runComputation() internally)
c. Activity: startComplianceEventActivity('computation.completed', computation_run_id).

Retry policy: validatePrereqs + runTaxComputation retryable 3× on transient errors;
NOT retryable on validation errors (missing input) — fail immediately with field-level details.

Add TemporalService.startComputationWorkflow(userId, assessmentYear, regime) helper.
WorkflowID: 'computation/{userId}/{assessmentYear}/{regime}/{Date.now()}'.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/computation.workflow.ts
apps/api/src/temporal/activities/computation.activities.ts
apps/api/src/temporal/temporal.service.ts (updated with startComputationWorkflow)
```

**Acceptance Criteria:**
- regime='compare' creates 2 computation_runs rows (one old, one new).
- Temporal UI shows both child workflow runs for a 'compare' request.
- Validation error returns structured error (not a 500).

---

### C49 — Computation API Endpoints

**Epic:** E08 | **Backlog task:** T08.10 | **AI:** ✅

**Goal:** `POST /computations`, `GET /computations/{id}`, `GET /computations?assessment_year=`.

**Prerequisites:** C48.

**Implementation Prompt:**
```
[Paste master context block]

Implement computation API endpoints for TaxLens. Reference 06_API.md computation endpoints exactly.

1. POST /computations (@RequiresConsent()):
   Body: { assessment_year, regime: 'old'|'new'|'compare' }.
   Pre-conditions: locked extractions exist for AY; tax profile exists.
   Start W4 via TemporalService.startComputationWorkflow().
   Return 202: { computation_run_ids: { old?: uuid, new?: uuid }, status: 'processing' }.

2. GET /computations/:computation_run_id (@RequiresConsent()):
   Verify user owns computation_run. Return full ComputationResult per 06_API.md schema.
   Include disclaimer in response (never omit).

3. GET /computations?assessment_year= (@RequiresConsent()):
   Return list of computation runs (summary, no full result JSON): id, regime, status, rule_version, created_at.

All routes require JWT + @RequiresConsent().
```

**Expected Files:**
```
apps/api/src/computation/computation.controller.ts
apps/api/src/computation/dto/trigger-computation.dto.ts
```

**Acceptance Criteria:**
- POST /computations starts W4 and returns 202 with computation_run_ids.
- GET /computations/:id for another user's run returns 404.
- GET /computations/:id response always contains disclaimer field.

---

### C50 — Tax Engine Unit Tests

**Epic:** E08 | **Backlog task:** T08.11 | **AI:** ⚠️

**Goal:** Comprehensive unit tests covering all tax engine edge cases.

**Prerequisites:** C43–C49.

**Implementation Prompt:**
```
[Paste master context block]

Write tax engine unit tests for TaxLens. Reference 09_TAX_ENGINE.md for all examples and edge cases.

Create apps/api/src/computation/tax-engine/*.spec.ts covering:

1. Standard salaried ITR-1 (old regime):
   Gross ₹12.5L, 80C ₹1.5L, 80D ₹25K, standard deduction ₹50K.
   Expected taxable: ₹12.5L - 50K - 1.5L - 25K = ₹10.75L.
   Expected slab tax: (2.5L@0 + 2.5L@5% + 5L@20% + 0.75L@30%) = ₹12,500+100,000+22,500 = ₹1,35,000.
   Cess 4%: ₹5,400. Total: ₹1,40,400.

2. Standard salaried ITR-1 (new regime):
   Same gross ₹12.5L. standard_deduction ₹75K. No other deductions.
   Taxable: ₹11.75L. Slab breakdown per 09_TAX_ENGINE Section 5 example.
   87A: income > ₹12L threshold → 0. Cess 4%.

3. Section 87A rebate edge case: taxable = ₹4,99,999 (old regime) → full rebate ₹12,500.
   taxable = ₹5,00,001 → rebate = 0.

4. New regime 87A: taxable = ₹11,99,999 → rebate = up to ₹60,000.
   taxable = ₹12,00,001 → rebate = 0.

5. Surcharge + marginal relief: income just above ₹50L threshold.

6. 80C cap: input ₹2,00,000 → applied ₹1,50,000 with note.

7. TDS > tax (refund scenario): TDS ₹1,00,000; total tax ₹80,000 → refundable ₹20,000.

8. New regime: 80C/80D inputs marked ignored with notes.
```

**Expected Files:**
```
apps/api/src/computation/tax-engine/slab-calculator.spec.ts
apps/api/src/computation/tax-engine/deduction-calculator.spec.ts
apps/api/src/computation/tax-engine/surcharge-calculator.spec.ts
apps/api/src/computation/tax-engine/rebate-calculator.spec.ts
apps/api/src/computation/tax-engine/tds-reconciler.spec.ts
```

**Acceptance Criteria:**
- All 8 test scenarios pass. `pnpm --filter api test` succeeds.
- Tax calculations match manually verified values from Finance Act 2024.
- Edge case tests confirm no off-by-one errors in slab boundaries.

---

## E09 — Report Generation

---

### C51 — PDF Report Template

**Epic:** E09 | **Backlog task:** T09.1 | **AI:** ✅

**Goal:** HTML/CSS template rendering all sections of the tax summary report with mandatory disclaimer.

**Prerequisites:** None (design task).

**Implementation Prompt:**
```
[Paste master context block]

Design the TaxLens tax summary PDF template. Reference 11_FRONTEND.md S-11 (report contents)
and 10_COMPLIANCE.md D-04 (mandatory disclaimer).

workers/reports/templates/tax_summary.html:
A single HTML file rendering the full report from a JSON data object passed as Jinja2 template vars.

Sections (in order):
1. Header: TaxLens logo, user name, AY, report version, generated date.
2. Non-ERI disclaimer banner (prominent, yellow box):
   "ESTIMATE ONLY — NOT A LEGAL TAX FILING. Generated by TaxLens."
3. Income Summary table: gross salary, exempt allowances, standard deduction, other income,
   total gross income.
4. Deductions Detail table: each section_code, sub_item, amount; total deductions.
   If new regime: show "Not applicable" row for ignored deductions.
5. Slab-wise Tax Breakdown table: slab label, rate, tax per slab, total.
6. Tax Computation Summary: tax before rebate, 87A rebate, surcharge + marginal relief, cess,
   total tax liability.
7. TDS Reconciliation table: deductor, TAN, amount, section; total TDS; net payable or refundable.
8. If regime comparison: side-by-side old vs new final tax; recommended regime.
9. Footer on every page: "ESTIMATE ONLY | TaxLens | AY {year} | Version {n} | Generated: {date}".

CSS: clean professional style; print-friendly; no external fonts (use system fonts).
Use Jinja2 variables: {{ income_summary }}, {{ deductions }}, {{ slab_breakdown }}, etc.
```

**Expected Files:**
```
workers/reports/templates/tax_summary.html
workers/reports/templates/styles.css
workers/reports/template_data_schema.py (Pydantic model for template vars)
```

**Acceptance Criteria:**
- Template renders to visually correct HTML in browser when given sample data.
- Disclaimer appears prominently on page 1 and in footer of every page.
- All monetary values formatted as ₹X,XX,XXX (Indian comma grouping).
- Template is print-safe (no fixed widths that cut off in PDF).

---

### C52 — PDF Rendering Worker

**Epic:** E09 | **Backlog task:** T09.2 | **AI:** ✅

**Goal:** WeasyPrint PDF rendering; store to S3; never pass bytes through Temporal history.

**Prerequisites:** C51, C05.

**Implementation Prompt:**
```
[Paste master context block]

Implement PDF rendering worker for TaxLens.

workers/reports/activities/render_pdf.py:

render_pdf_activity(report_data: dict, report_type: str, template_version: str) -> str:
  (returns s3_key of stored PDF; NOT bytes — large payload rule)

1. Load Jinja2 template (tax_summary.html).
2. Render template with report_data vars → html_string.
3. Generate PDF: from weasyprint import HTML; pdf_bytes = HTML(string=html_string).write_pdf().
4. Upload to S3 (taxlens-reports bucket):
   key = f'reports/{user_id}/{assessment_year}/v{version}_{uuid4().hex[:8]}.pdf'.
   boto3.client('s3').put_object(Body=pdf_bytes, Bucket=REPORTS_BUCKET, Key=key,
   ContentType='application/pdf', ServerSideEncryption='aws:kms').
5. Return s3_key (NOT bytes; WeasyPrint renders in memory but we store and discard immediately).
6. Retry policy: 2 retries (WeasyPrint failures are usually transient OOM).

workers/reports/activities/assemble_report_data.py:
assemble_report_data_activity(computation_run_ids: list[str], user_id: str) -> dict:
Fetch computation_runs rows. Fetch user profile (name). Build template_data_schema object.
```

**Expected Files:**
```
workers/reports/activities/render_pdf.py
workers/reports/activities/assemble_report_data.py
workers/reports/activities/store_report_record.py
```

**Acceptance Criteria:**
- render_pdf_activity returns an S3 key (not bytes).
- PDF stored in S3 reports bucket. Readable with pdf reader.
- Disclaimer appears in generated PDF footer on every page.

---

### C53 — ReportGenerationWorkflow (W5)

**Epic:** E09 | **Backlog task:** T09.3 | **AI:** ✅

**Goal:** Temporal W5 — assemble → render → store → record → audit event.

**Prerequisites:** C04, C52.

**Implementation Prompt:**
```
[Paste master context block]

Implement ReportGenerationWorkflow (W5). Reference 07_WORKFLOWS.md W5.

workers/reports/workflows/report_generation_workflow.py:
@workflow.defn class ReportGenerationWorkflow:
  @workflow.run async def run(self, input: ReportWorkflowInput):
    # input: computation_run_ids (list), user_id, report_type='tax_summary'
    report_data = await workflow.execute_activity(assemble_report_data_activity,
      { computation_run_ids, user_id }, ...)
    s3_key = await workflow.execute_activity(render_pdf_activity,
      { report_data, report_type, template_version: '1.0' }, ...)
    # Store to S3 done inside render_pdf_activity; s3_key returned
    report_record_id = await workflow.execute_activity(store_report_record_activity,
      { user_id, computation_run_ids, s3_key, report_type }, ...)
    await workflow.execute_activity(write_audit_event_activity,
      { action: 'report.generated', resource_id: report_record_id, user_id }, ...)
    return report_record_id

Retry policy per 07_WORKFLOWS: RenderPDFActivity 2×; StorePDFToS3 5× (S3 is reliable).
Large payload rule: pdf_bytes never in Temporal history; only s3_key passed between activities.
```

**Expected Files:**
```
workers/reports/workflows/report_generation_workflow.py
workers/reports/workflows/input_models.py
workers/reports/temporal_worker.py (updated to register workflow)
```

**Acceptance Criteria:**
- W5 completes: report_records row created, S3 key stored, audit event written.
- PDF bytes never appear in Temporal workflow history (verify in Temporal UI).
- W5 visible in Temporal UI reports-queue.

---

### C54 — Report API Endpoints

**Epic:** E09 | **Backlog task:** T09.4 | **AI:** ✅

**Goal:** `POST /reports`, `GET /reports/{id}/download-url`, `GET /reports`.

**Prerequisites:** C53.

**Implementation Prompt:**
```
[Paste master context block]

Implement report API endpoints for TaxLens. Reference 06_API.md report endpoints.

1. reports/reports.service.ts:
   triggerReport(userId, dto: { computation_run_id | computation_run_ids[] }):
   Verify user owns computation_run(s). Start W5 via TemporalService.startWorkflow('reports-queue', ...).
   Return { report_job_id, status: 'queued' }.

   getDownloadUrl(userId, reportId): verify ownership.
   Generate 10-minute pre-signed GET URL for report S3 key.
   Return { download_url, expires_at, version }.

   listReports(userId, assessmentYear): return report_records for user.

2. reports/reports.controller.ts (@RequiresConsent() on all routes):
   POST /reports: body { computation_run_id } or { computation_run_ids: [] }.
   GET /reports/:report_id/download-url: new URL generated on each call (never cache).
   GET /reports?assessment_year=.

Emit report.generation_requested on POST. Emit report.download_url_issued on GET download-url.
```

**Expected Files:**
```
apps/api/src/reports/reports.module.ts
apps/api/src/reports/reports.service.ts
apps/api/src/reports/reports.controller.ts
apps/api/src/reports/dto/trigger-report.dto.ts
apps/api/src/reports/report-record.entity.ts
```

**Acceptance Criteria:**
- POST /reports returns 202 + report_job_id.
- GET /reports/:id/download-url returns a fresh pre-signed URL (10 min expiry).
- Downloading the URL retrieves the PDF.
- Report for another user's computation returns 404.

---

## E10 — Frontend — Auth & Onboarding

---

### C55 — Login Screen

**Epic:** E10 | **Backlog task:** T10.1 | **AI:** ✅

**Goal:** Login screen (S-01) with JWT storage and redirect logic.

**Prerequisites:** C12.

**Implementation Prompt:**
```
[Paste master context block]

Implement login screen for TaxLens Next.js frontend. Reference 11_FRONTEND.md S-01.

1. apps/web/src/lib/api-client.ts:
   Axios instance with BASE_URL = NEXT_PUBLIC_API_URL.
   Request interceptor: attach access_token from memory (not localStorage — XSS risk).
   Response interceptor: on 401, attempt refresh via POST /auth/refresh (httpOnly cookie for refresh token);
   on refresh failure: redirect to /auth/login.
   Store access_token in module-level variable (memory only).

2. apps/web/src/app/(auth)/login/page.tsx:
   Email input + password (show/hide). "Forgot password?" link.
   Submit → POST /auth/login. On success: store access_token in memory; redirect to /dashboard
   (or /onboarding if GET /consent/status returns has_active_consent=false).
   Error states: 401 → "Invalid credentials", 423 → "Account locked. Try again in 10 minutes.",
   403 → "Please verify your email first."
   Loading: button spinner; form disabled. Use React Hook Form + zod.
```

**Expected Files:**
```
apps/web/src/lib/api-client.ts
apps/web/src/app/(auth)/login/page.tsx
apps/web/src/app/(auth)/layout.tsx
apps/web/src/hooks/useAuth.ts
```

**Acceptance Criteria:**
- Valid credentials → stored access_token in memory → redirect to dashboard.
- 423 response shows lockout message, not generic error.
- Refresh token stored in httpOnly cookie (verify: no refresh_token in localStorage).

---

### C56 — Register + Email Verification Screens

**Epic:** E10 | **Backlog task:** T10.2 | **AI:** ✅

**Goal:** Register (S-02) and email verification (S-03) screens.

**Prerequisites:** C55.

**Implementation Prompt:**
```
[Paste master context block]

Implement register and email verification screens for TaxLens. Reference 11_FRONTEND.md S-02, S-03.

1. apps/web/src/app/(auth)/register/page.tsx:
   Fields: full_name, email, password (with strength indicator), confirm_password.
   Client validation: passwords match; password meets requirements (1 uppercase, 1 digit, 1 special).
   Submit → POST /auth/register. Success: redirect to /auth/verify-email?email={email}.
   Error: 409 → "An account with this email already exists."

2. apps/web/src/app/(auth)/verify-email/page.tsx:
   "Enter the 6-digit code sent to {email}" headline.
   6-digit OTP input: split into 6 individual input boxes; auto-advance on each digit.
   "Resend code" link: enabled after 60s countdown (use useEffect timer).
   Resend → POST /auth/register again (re-sends OTP without creating new user — handle 409 gracefully).
   Submit → POST /auth/verify-email. Success: redirect to /auth/login with toast "Email verified!".
   Errors: 400 "Invalid code", 410 "Code expired — click Resend".
```

**Expected Files:**
```
apps/web/src/app/(auth)/register/page.tsx
apps/web/src/app/(auth)/verify-email/page.tsx
apps/web/src/components/auth/OtpInput.tsx (6-digit auto-advance component)
```

**Acceptance Criteria:**
- Registration form validates passwords match before submit.
- OTP input auto-advances between boxes on digit entry.
- Resend countdown works correctly (60s before enabled).
- Successful OTP verification redirects to login with success toast.

---

### C57 — Onboarding / Consent Screen

**Epic:** E10 | **Backlog task:** T10.3 | **AI:** 🔒

**Goal:** Full-screen consent modal (S-04) with dual checkbox, D-01 + D-02 disclaimers. Cannot be skipped.

**Prerequisites:** C55, C18.

**Implementation Prompt:**
```
[Paste master context block]

Implement onboarding consent screen for TaxLens. Reference 11_FRONTEND.md S-04 and
10_COMPLIANCE.md D-01, D-02. COMPLIANCE-CRITICAL — legal review required before shipping.

apps/web/src/app/(onboarding)/onboarding/page.tsx:

Full-screen layout (no navigation bar). Cannot be dismissed or back-navigated away from.

Content:
1. TaxLens logo.
2. D-01 disclaimer block (non-ERI notice — exact wording from 10_COMPLIANCE D-01).
3. D-02 consent statement (exact wording from 10_COMPLIANCE D-02).
4. Privacy Policy link + Terms of Service link (open new tab).
5. Checkbox 1: "I understand TaxLens provides estimates, not official tax filings."
6. Checkbox 2: "I agree to the Privacy Policy and Terms of Service (v{CONSENT_VERSION})."
7. "Continue" button: DISABLED until BOTH checkboxes checked. No exceptions.

On Continue: POST /consent with { consent_version: NEXT_PUBLIC_CONSENT_VERSION }.
On success: redirect to /dashboard.

IMPORTANT: If user navigates to /onboarding with active consent: redirect to /dashboard immediately.
If user navigates to any protected route without consent: middleware redirects to /onboarding.
The consent version shown must match NEXT_PUBLIC_CONSENT_VERSION env var.
```

**Expected Files:**
```
apps/web/src/app/(onboarding)/onboarding/page.tsx
apps/web/src/app/(onboarding)/layout.tsx
apps/web/src/middleware.ts (Next.js middleware: check consent status; redirect to /onboarding if missing)
```

**Acceptance Criteria:**
- "Continue" button remains disabled until both checkboxes checked.
- Clicking Continue with both checked → POST /consent → redirect to /dashboard.
- Navigating to /documents without consent → redirect to /onboarding.
- Exact disclaimer text matches 10_COMPLIANCE D-01 and D-02 word-for-word.

---

### C58 — Client-Side Auth Guards

**Epic:** E10 | **Backlog task:** T10.4 | **AI:** ✅

**Goal:** Next.js middleware and layout-level auth guards; token refresh on page load.

**Prerequisites:** C55–C57.

**Implementation Prompt:**
```
[Paste master context block]

Implement auth guards for TaxLens Next.js frontend.

1. apps/web/src/middleware.ts (Next.js middleware):
   On every request to protected routes (/dashboard, /documents, /extraction, /estimate, /reports,
   /activity, /settings, /tax-profile):
   - Check for access_token cookie or Authorization header.
   - If missing or expired: redirect to /auth/login.
   - Check consent status via GET /consent/status:
     if has_active_consent=false → redirect to /onboarding.
     if CONSENT_REFRESH_REQUIRED → redirect to /onboarding?refresh=true.
   Public routes: /auth/*, /help — no check.

2. apps/web/src/app/(app)/layout.tsx (app shell layout):
   On mount: call GET /users/me to verify session is live.
   If deleted_at not null: show full-page overlay "Account deletion in progress."
   Global navigation bar (from C69).

3. apps/web/src/hooks/useCurrentUser.ts:
   React Query hook wrapping GET /users/me. Used throughout app for user context.
```

**Expected Files:**
```
apps/web/src/middleware.ts
apps/web/src/app/(app)/layout.tsx
apps/web/src/hooks/useCurrentUser.ts
apps/web/src/components/layout/NavBar.tsx (top navigation bar)
```

**Acceptance Criteria:**
- Unauthenticated request to /dashboard → redirect to /auth/login.
- Authenticated user without consent → redirect to /onboarding.
- Deleted account → full-page overlay shown; no other navigation available.

---

## E11 — Frontend — Documents & Extraction

---

### C59 — Document List Screen + Upload Modal

**Epic:** E11 | **Backlog tasks:** T11.1, T11.2 | **AI:** ⚠️

**Goal:** Document list (S-06) and upload modal with direct S3 XHR upload and progress bar.

**Prerequisites:** C58, C24, C25.

**Implementation Prompt:**
```
[Paste master context block]

Implement document list and upload modal for TaxLens. Reference 11_FRONTEND.md S-06.

1. apps/web/src/app/(app)/documents/page.tsx:
   Filter bar: AY selector, document type filter. Document list table (collapses to cards on mobile).
   Columns: Document Type, AY, Filename, Upload Date, Extraction Status badge, Actions (View / Delete).
   Status badges: color-coded (Pending Scan / Extracting / Review Needed / Completed / Failed / Quarantined).
   Empty state with "Upload your Form 16 to start" CTA.
   "Upload New Document" button → opens UploadModal.

2. apps/web/src/components/documents/UploadModal.tsx:
   Document type selector, AY selector (default 2025-26; 2024-25 disabled in v1).
   Drag-and-drop + browse input. Accept: .pdf,.jpg,.png,.csv. Max 20 MB.
   File size indicator; MIME type validation client-side.
   Upload flow:
   a. POST /documents/upload-url → { upload_url, document_version_id }.
   b. XHR PUT to upload_url with Content-Type header; track progress (onprogress event).
   c. On XHR complete: POST /documents/:version_id/confirm-upload.
   d. Show "Extraction starting…" status. Close modal; refresh document list.
   Error handling: file too large → client error before API call; network error → retry button.
```

**Expected Files:**
```
apps/web/src/app/(app)/documents/page.tsx
apps/web/src/components/documents/UploadModal.tsx
apps/web/src/components/documents/DocumentStatusBadge.tsx
apps/web/src/lib/upload.ts (XHR upload utility with progress callback)
```

**Acceptance Criteria:**
- Document list shows correct status badges. Filters work.
- Upload modal drag-and-drop works; progress bar shows during XHR upload.
- File > 20 MB: client error shown before API call.
- After upload: extraction_jobs.status='queued' visible in document list.

---

### C60 — Extraction Job Progress (SSE/Polling)

**Epic:** E11 | **Backlog task:** T11.3 | **AI:** ✅

**Goal:** Real-time extraction job status updates via Server-Sent Events or polling fallback.

**Prerequisites:** C59.

**Implementation Prompt:**
```
[Paste master context block]

Implement extraction progress updates for TaxLens frontend.

1. apps/api/src/extraction/extraction-events.controller.ts:
   GET /extractions/:extraction_job_id/events (SSE endpoint):
   Use NestJS @Sse() decorator + Observable. Poll DB every 3s for job status.
   Emit SSE event when status changes. Complete stream when status is terminal
   ('completed', 'needs_review', 'failed').

2. apps/web/src/hooks/useExtractionStatus.ts:
   React hook using EventSource (SSE). Falls back to polling (GET /extractions/:id) every 5s
   if EventSource not supported.
   Returns: { status, resultId, error }.
   Closes connection on terminal status.

3. apps/web/src/components/documents/ExtractionStatusPoller.tsx:
   Wrapper component using useExtractionStatus hook.
   On status='completed': update document list status badge; show toast "Extraction ready for review."
   On status='needs_review': show toast "Extraction needs your review."
   On status='failed': show toast with retry prompt.
```

**Expected Files:**
```
apps/api/src/extraction/extraction-events.controller.ts
apps/web/src/hooks/useExtractionStatus.ts
apps/web/src/components/documents/ExtractionStatusPoller.tsx
```

**Acceptance Criteria:**
- Status badge updates automatically when extraction completes (no page refresh).
- SSE connection closes after terminal status received.
- Polling fallback activates if SSE throws error.

---

### C61 — Extraction Review Screen (Frontend)

**Epic:** E11 | **Backlog task:** T11.4 | **AI:** ⚠️

**Goal:** Complete extraction review page (S-08) integrating PdfViewer, FieldListPanel, FieldEditor, CorrectionHistoryDrawer, LockExtractionBar. Combines C38 + C39 into a complete page.

**Prerequisites:** C38, C39, C59.

**Implementation Prompt:**
```
[Paste master context block]

Complete the extraction review page for TaxLens integrating all components from C38 and C39.
Reference 11_FRONTEND.md S-08 for complete spec.

apps/web/src/app/(app)/extraction/[extraction_job_id]/page.tsx (complete implementation):

1. Data fetching: GET /extractions/:id/result via React Query. Poll every 10s if status='processing'.

2. Layout (desktop): CSS Grid with two columns: PdfViewer (left 55%), FieldListPanel (right 45%).
   Mobile: single column with toggle button "View Document" | "Review Fields".

3. Compliance banner D-05 at top (sticky, cannot be dismissed).

4. TDS reconciliation warning: if reconciliation_warnings[] non-empty, show orange banner
   with specific warning messages. Link to relevant document for context.

5. FieldListPanel integration:
   - Sections by document part (Part A / Part B for form_16; Salary / TDS for ais).
   - Clicking Edit icon → renders FieldEditor inline (replaces value display).
   - Clicking History icon → opens CorrectionHistoryDrawer.

6. LockExtractionBar: sticky bottom. Shows: "X of Y fields verified."
   On mobile: full-width button above bottom nav.

7. After lock: redirect to /tax-profile/:assessment_year with a
   "Extraction locked! Set up your deductions next." toast.
```

**Expected Files:**
```
apps/web/src/app/(app)/extraction/[extraction_job_id]/page.tsx (complete)
apps/web/src/components/extraction/ExtractionSectionHeader.tsx
apps/web/src/components/extraction/ReconciliationWarningBanner.tsx
```

**Acceptance Criteria:**
- Page works end-to-end: upload Form 16 → extraction completes → open review → correct a field → lock → redirected to tax profile.
- Mobile toggle between PDF view and field list works.
- Low-confidence fields remain highlighted after editing (confidence score unchanged by correction; only value changes).

---

## E12 — Frontend — Estimate & Reports

---

### C62 — Dashboard Screen

**Epic:** E12 | **Backlog task:** T12.6 | **AI:** ✅

**Goal:** Central dashboard (S-05) showing journey status, document chips, and quick actions.

**Prerequisites:** C58, C59.

**Implementation Prompt:**
```
[Paste master context block]

Implement the TaxLens dashboard. Reference 11_FRONTEND.md S-05.

apps/web/src/app/(app)/dashboard/page.tsx:

1. AY selector: dropdown showing AY 2025-26 (active); AY 2024-25 (disabled, "Coming Soon" tooltip).

2. Status card (primary CTA — drives user through journey):
   Compute current state by checking:
   - No documents: "Upload your documents to start" → CTA "Upload Form 16".
   - Documents uploaded but no locked extraction: "Review extraction for [doc_type]" → CTA.
   - All Must-Have docs locked, no computation: "Ready to compute" → CTA "Compute Tax Estimate".
   - Computation done: "View your tax estimate" → CTA "View Estimate" → /estimate/:ay.

3. Document summary row: pill/chip per document type. Green check if extraction locked;
   yellow if needs_review; grey if not uploaded.

4. Quick action buttons: "Upload Document", "Review Extraction" (if applicable), "Compute Tax".

5. Estimate summary card (visible after computation): regime comparison mini-view.

6. Recent activity strip: last 3 audit events from GET /audit/me.
```

**Expected Files:**
```
apps/web/src/app/(app)/dashboard/page.tsx
apps/web/src/components/dashboard/JourneyStatusCard.tsx
apps/web/src/components/dashboard/DocumentSummaryRow.tsx
apps/web/src/components/dashboard/EstimateSummaryCard.tsx
apps/web/src/components/dashboard/RecentActivityStrip.tsx
```

**Acceptance Criteria:**
- Status card shows correct CTA for each journey stage.
- Document chips reflect extraction status. Estimate card only visible after computation.
- Clicking status card CTA navigates to correct screen.

---

### C63 — Tax Profile + Deduction Form

**Epic:** E12 | **Backlog task:** T12.1 | **AI:** ✅

**Goal:** Tax profile (regime selector) and deduction accordion form (S-09).

**Prerequisites:** C40, C61.

**Implementation Prompt:**
```
[Paste master context block]

Implement tax profile and deduction form for TaxLens. Reference 11_FRONTEND.md S-09.

apps/web/src/app/(app)/tax-profile/[assessment_year]/page.tsx:
Two tabs: "Profile" and "Deductions".

Profile Tab:
Regime selector (radio group): Old Regime | New Regime | Compare Both (Recommended).
Residential status: Resident (only active; NRI shows tooltip "NRI support coming soon").
Save → PATCH /tax-profiles/:ay.

Deductions Tab:
Compliance disclaimer D-06 at top.
Accordion sections per deduction category:
- 80C: itemized rows (sub_item dropdown + amount input); progress bar toward ₹1.5L cap.
- 80D (self + parents): separate amount inputs; show cap per field.
- HRA: rent paid per month, city type (metro/non-metro), employer HRA component.
- 80TTA/80TTB: interest amounts (pre-filled from AIS extraction if available).
- 80G: charity name + amount.
- 80CCD(1B): NPS contribution.
- Professional Tax: actual amount.
- 80CCD(2): read-only field if extracted from Form 16; tooltip "Auto-filled from Form 16 Part B."

New Regime selected: gray out non-permitted sections; show tooltip "Not applicable under New Regime."
Auto-save each section on accordion close. Global "Save" button at bottom.
"Compute Tax Estimate →" CTA at page bottom (only enabled when tax profile saved).
```

**Expected Files:**
```
apps/web/src/app/(app)/tax-profile/[assessment_year]/page.tsx
apps/web/src/components/tax-profile/RegimeSelector.tsx
apps/web/src/components/tax-profile/DeductionAccordion.tsx
apps/web/src/components/tax-profile/Section80CPanel.tsx
apps/web/src/hooks/useTaxProfile.ts
```

**Acceptance Criteria:**
- 80C progress bar updates as amounts are entered; turns red at cap.
- New regime selection grays out 80C/80D sections with tooltip.
- 80CCD(2) field read-only if source='extracted'; editable if source='user_input'.
- "Compute Tax Estimate" CTA triggers POST /computations → redirects to estimate screen.

---

### C64 — Tax Estimate Dashboard (Part 1: Regime Comparison + Breakdown)

**Epic:** E12 | **Backlog task:** T12.2 part 1 | **AI:** ⚠️

**Goal:** Estimate dashboard S-10: regime comparison card and computation breakdown accordion.

**Prerequisites:** C49, C62.

**Implementation Prompt:**
```
[Paste master context block]

Implement estimate dashboard Part 1 for TaxLens. Reference 11_FRONTEND.md S-10.

apps/web/src/app/(app)/estimate/[assessment_year]/page.tsx:

1. D-03 disclaimer banner: yellow sticky banner at top. Cannot be collapsed or dismissed.
   Text exactly as per 10_COMPLIANCE D-03.

2. Regime comparison card (prominent, top of page):
   Side-by-side: Old Regime ₹X | New Regime ₹Y | Recommended (green badge) on lower.
   "You save ₹Z with [regime]" subtext.
   D-07 regime note as collapsible info tooltip.

3. Breakdown accordion (per regime, switchable with regime tabs):
   Sections: Income Summary | Deductions | Taxable Income | Slab Tax | Rebate & Cess | Net Position.
   Income Summary: table with gross salary, standard deduction, other income, total.
   Deductions: each section_code row with amount; new regime rows show "Not applicable" in grey.
   Slab Tax: table with slab label, rate, tax per slab, subtotal.
   Rebate & Cess: 87A rebate row, surcharge + marginal relief, cess, total tax.
   Net Position: TDS paid, net payable (red) or refundable (green) — prominent.

4. Compute metadata footer: "Computed on [date] | Rule: AY2025-26-v1 | Based on [N] documents".
```

**Expected Files:**
```
apps/web/src/app/(app)/estimate/[assessment_year]/page.tsx
apps/web/src/components/estimate/RegimeComparisonCard.tsx
apps/web/src/components/estimate/ComputationBreakdown.tsx
apps/web/src/components/estimate/SlabBreakdownTable.tsx
apps/web/src/hooks/useComputation.ts
```

**Acceptance Criteria:**
- D-03 disclaimer banner cannot be dismissed.
- Regime comparison card shows correct values and highlights recommended regime.
- Slab breakdown table matches values from C50 unit test scenarios.

---

### C65 — Tax Estimate Dashboard (Part 2: TDS Reconciliation + Actions)

**Epic:** E12 | **Backlog task:** T12.2 part 2 | **AI:** ⚠️

**Goal:** TDS reconciliation section, recompute action, report generation trigger.

**Prerequisites:** C64.

**Implementation Prompt:**
```
[Paste master context block]

Implement estimate dashboard Part 2 for TaxLens.

1. TDS Reconciliation section (bottom of estimate page):
   Table: Deductor Name | TAN | Amount | Section.
   Warning rows if reconciliation_warnings[] present: orange callout with warning message.
   "Reconciliation note: verify these figures match your Form 26AS / AIS."

2. Action bar (bottom of page, sticky on scroll):
   "Download Report" button: → POST /reports { computation_run_id }
   → polling while report generates → "Download PDF" enabled when ready.
   "Recompute" button: warning modal "This will run a fresh computation with your current inputs."
   → POST /computations → refresh page with new results.
   "Edit Deductions" link: → /tax-profile/:ay.

3. Report generation flow:
   POST /reports → get report_job_id → poll GET /reports/:id/download-url every 3s.
   On ready: swap button to "Download PDF" (pre-signed URL; new URL generated on each click).
   Show spinner while generating.

4. Loading/empty states: if no computation exists → "Compute your first tax estimate" CTA.
   If computation in progress → progress indicator.
```

**Expected Files:**
```
apps/web/src/components/estimate/TDSReconciliationTable.tsx
apps/web/src/components/estimate/EstimateActionBar.tsx
apps/web/src/components/estimate/ReportDownloadButton.tsx
apps/web/src/hooks/useReportGeneration.ts
```

**Acceptance Criteria:**
- TDS reconciliation warnings shown as orange callouts.
- Download Report → polling → "Download PDF" enabled after generation.
- Recompute shows warning modal before triggering.

---

### C66 — Reports Screen + Activity Timeline + Settings

**Epic:** E12 | **Backlog tasks:** T12.3, T12.4, T12.5 | **AI:** ✅

**Goal:** Reports list (S-11), activity timeline (S-12), and settings screens (S-13, S-14, S-15).

**Prerequisites:** C54, C58.

**Implementation Prompt:**
```
[Paste master context block]

Implement remaining frontend screens for TaxLens.

1. apps/web/src/app/(app)/reports/page.tsx (S-11):
   Report list: version, AY, generated_at, "Download PDF" button (pre-signed URL, 10-min expiry).
   "Generate Report" modal: AY selector + computation run selector.
   Empty state: "No reports yet. Generate one from the Estimate screen."

2. apps/web/src/app/(app)/activity/page.tsx (S-12):
   Reverse-chronological audit feed. Event icon + label + resource name + timestamp + IP.
   Filter bar: date range + event type. Infinite scroll (cursor-based). Empty state.

3. apps/web/src/app/(app)/settings/profile/page.tsx (S-13):
   Full name editable. Email read-only. PAN: masked display + "Update PAN" button (modal).

4. apps/web/src/app/(app)/settings/security/page.tsx (S-14):
   Change password form. 2FA section: status + enable/disable button (QR code modal for setup).

5. apps/web/src/app/(app)/settings/data/page.tsx (S-15):
   "Download your data" (disabled, "Coming Soon"). Account deletion section with D-08 disclaimer.
   Deletion modal: text input ("DELETE MY ACCOUNT") + confirm button.
```

**Expected Files:**
```
apps/web/src/app/(app)/reports/page.tsx
apps/web/src/app/(app)/activity/page.tsx
apps/web/src/app/(app)/settings/profile/page.tsx
apps/web/src/app/(app)/settings/security/page.tsx
apps/web/src/app/(app)/settings/data/page.tsx
apps/web/src/components/activity/ActivityEventRow.tsx
```

**Acceptance Criteria:**
- Reports list shows download button; clicking generates fresh pre-signed URL.
- Activity feed loads with infinite scroll; filters reduce results correctly.
- Deletion modal requires exact text "DELETE MY ACCOUNT" before enabling confirm.

---

## E13 — Admin Console

---

### C67 — Admin Auth + Overview Dashboard

**Epic:** E13 | **Backlog tasks:** T13.1, T13.2 | **AI:** 🔒

**Goal:** Admin session enforcement (TOTP required); admin overview dashboard (A-01).

**Prerequisites:** C16, C14, C23.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin auth enforcement and overview dashboard for TaxLens.
Reference 12_ADMIN.md A-01 and 10_COMPLIANCE Section 4.

1. Admin auth enforcement:
   apps/api/src/auth/guards/admin-mfa.guard.ts:
   Guard verifying: role IN ('admin','operator') AND totp_enabled=true.
   If totp_enabled=false: 403 { code: 'MFA_REQUIRED', message: 'Admin accounts require 2FA. Enable at /settings/security.' }.
   Apply to all /admin/* routes.
   Admin session timeout: 15 min (enforce via separate JWT expiry for admin tokens or Redis TTL).

2. GET /admin/health (STITCH-03 G13):
   apps/api/src/admin/health.controller.ts:
   Check each service (DB: SELECT 1; Redis: PING; Temporal: WorkflowClient.connection.workflowService; S3: HeadBucket; ClamAV: TCP connect to CLAMAV_HOST:3310; SES: GetSendQuota).
   Return { services: [{name, status: 'up'|'degraded'|'down', latency_ms, checked_at}], overall }.

3. apps/web/src/app/(admin)/layout.tsx (admin shell — separate from app shell):
   D-09 sticky compliance banner on all pages.
   Sidebar navigation: Overview | Extractions | Users | Computations | Tax Rules | Approvals | Audit Log | Health | Security.

4. apps/web/src/app/(admin)/page.tsx (A-01 overview):
   Service health banner (traffic lights from GET /admin/health).
   Job queue summary cards (pending/failed extraction jobs count, computation count, report count).
   Alerts panel: maker-checker items > 24h old; failed extractions not re-triggered.
   Recent audit events strip (last 10).
```

**Expected Files:**
```
apps/api/src/auth/guards/admin-mfa.guard.ts
apps/api/src/admin/health.controller.ts
apps/web/src/app/(admin)/layout.tsx
apps/web/src/app/(admin)/page.tsx
apps/web/src/components/admin/ServiceHealthBanner.tsx
apps/web/src/components/admin/JobQueueCards.tsx
```

**Acceptance Criteria:**
- Admin user without 2FA enabled gets 403 MFA_REQUIRED on all /admin routes.
- GET /admin/health returns correct service status for all services.
- Overview dashboard shows queue counts and health lights.

---

### C68 — Admin Extraction Queue + User Management

**Epic:** E13 | **Backlog tasks:** T13.3, T13.4 | **AI:** ✅

**Goal:** Extraction job queue (A-02) with detail drawer; user management (A-03) with compliance banner.

**Prerequisites:** C67.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin extraction queue and user management screens for TaxLens.
Reference 12_ADMIN.md A-02, A-03.

1. apps/web/src/app/(admin)/extractions/page.tsx (A-02):
   Filter bar: status, document type, date range.
   Table: Job ID (short), User (email masked), Document Type, AY, Status badge, OCR Vendor, Created At, Actions.
   Actions: "View Detail" (opens drawer), "Re-trigger" (POST /admin/extraction-jobs/:id/retry).
   Detail drawer (right slide-in): full extraction result fields + confidence scores;
   raw OCR S3 link (admin only); failure reason; Temporal workflow ID (deep link to Temporal UI URL).
   Batch re-trigger: checkbox select + "Re-trigger Selected" button.

2. apps/web/src/app/(admin)/users/page.tsx (A-03 user list):
   Search by email. Table with: email (masked), role badge, consent status, last login, actions.

3. apps/web/src/app/(admin)/users/[user_id]/page.tsx (A-03 user detail):
   D-09 compliance banner (sticky, every page).
   Tabs: Documents | Extractions | Computations | Reports | Activity | Consent History.
   Each tab fetches from admin endpoints (GET /admin/users/:id endpoint needed — reference 06_API.md).
   Admin action sidebar: Re-trigger extraction, Re-run computation, Initiate deletion (opens W9 flow).
   Every page view emits admin.user_data_viewed via API (backend triggers audit event on GET /admin/users/:id).
```

**Expected Files:**
```
apps/web/src/app/(admin)/extractions/page.tsx
apps/web/src/components/admin/ExtractionDetailDrawer.tsx
apps/web/src/app/(admin)/users/page.tsx
apps/web/src/app/(admin)/users/[user_id]/page.tsx
apps/web/src/components/admin/UserDetailTabs.tsx
apps/web/src/components/admin/AdminActionSidebar.tsx
```

**Acceptance Criteria:**
- Re-trigger button calls admin API and shows success toast.
- D-09 banner visible on every user detail page load.
- user.detail page emits admin.user_data_viewed audit event on each load (verify in audit log).

---

### C69 — Admin Computation, Tax Rules, Approvals, Audit

**Epic:** E13 | **Backlog tasks:** T13.5, T13.6, T13.7, T13.8 | **AI:** ✅

**Goal:** Computation management (A-04), tax rules (A-05), pending approvals (A-06), audit log (A-07).

**Prerequisites:** C67.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin management screens for TaxLens. Reference 12_ADMIN.md A-04 through A-07.

1. apps/web/src/app/(admin)/computations/page.tsx (A-04):
   Filter by rule_version, AY, regime. Table + re-run action with reason modal.

2. apps/web/src/app/(admin)/tax-rules/page.tsx (A-05):
   Rule version table: AY, regime, version, is_active badge, effective_from, notes.
   "View Rule" → JSON rendered as readable tables (slabs table, surcharge table, rebate config).
   "Create New Version" → form: AY, regime, version tag, effective_from, notes, rules JSON editor
   (with basic schema validation). Submit → POST /admin/tax-rules → shows "Pending approval" state.
   "Activate" button on inactive rules → initiates maker-checker via POST /admin/approvals.

3. apps/web/src/app/(admin)/approvals/page.tsx (A-06):
   Approval queue sorted by initiated_at ASC. Action type badge. "Cannot approve own action" label
   if is_own_action=true. Approve/Reject buttons. Reason input (required for reject).

4. apps/web/src/app/(admin)/audit/page.tsx (A-07):
   Search/filter (user, action type, resource type, date range, IP).
   Table: event rows with expandable before/after JSON.
   PII view toggle (admin only): "Show Raw Values" → confirm modal →
   POST /admin/audit/enable-raw-view (returns short-lived raw token; emits admin.user_data_viewed).
   CSV export button (admin only; max 90 days per export; triggers download).
```

**Expected Files:**
```
apps/web/src/app/(admin)/computations/page.tsx
apps/web/src/app/(admin)/tax-rules/page.tsx
apps/web/src/components/admin/TaxRuleJsonViewer.tsx
apps/web/src/app/(admin)/approvals/page.tsx
apps/web/src/app/(admin)/audit/page.tsx
apps/web/src/components/admin/AuditEventRow.tsx
```

**Acceptance Criteria:**
- Tax rule create form validates rules JSON before submit.
- Approval queue shows "Cannot approve" for initiator's own actions.
- Audit log PII toggle shows confirmation modal before revealing raw values.
- CSV export generates file download (max 90 days enforced client-side in date picker).

---

### C70 — Admin Health Monitor + Security Monitor

**Epic:** E13 | **Backlog tasks:** T13.9, T13.10 | **AI:** ✅

**Goal:** System health (A-08) with throughput charts; security monitoring (A-09) with lockout management.

**Prerequisites:** C67.

**Implementation Prompt:**
```
[Paste master context block]

Implement admin health and security screens for TaxLens. Reference 12_ADMIN.md A-08, A-09.

1. apps/web/src/app/(admin)/health/page.tsx (A-08):
   Service status grid: each service row with status light (green/yellow/red), latency, last check.
   Auto-refreshes every 30s (React Query refetchInterval).
   Worker throughput charts (24h): extractions/hr, computations/hr, reports/hr, failed job rate.
   Use recharts LineChart. Data from a new endpoint: GET /admin/metrics (returns time-series from
   aggregated audit_events counts — implement simple aggregation query in NestJS).
   OCR vendor error log (last 20 errors from extraction_jobs where status='failed', last 24h).

2. apps/web/src/app/(admin)/security/page.tsx (A-09):
   Login failure summary: failed auth.login_failed events per IP per hour (query audit_events).
   Locked accounts: users where Redis 'login_lock:{email}' exists.
   GET /admin/security/locked-accounts endpoint: query Redis SCAN for 'login_lock:*' keys.
   "Unlock" button per row → DELETE /admin/auth/unlock/:email (from C17).
   Active admin sessions table: user + IP + user_agent + created_at + "Revoke" action.
```

**Expected Files:**
```
apps/web/src/app/(admin)/health/page.tsx
apps/web/src/app/(admin)/security/page.tsx
apps/api/src/admin/metrics.controller.ts (GET /admin/metrics with time-series data)
apps/api/src/admin/security.controller.ts (GET /admin/security/locked-accounts)
apps/web/src/components/admin/ServiceStatusGrid.tsx
apps/web/src/components/admin/ThroughputChart.tsx
```

**Acceptance Criteria:**
- Health page auto-refreshes every 30s. Service status changes reflect within one refresh cycle.
- Throughput charts render with recharts LineChart.
- Locked accounts list shows correct Redis keys. Unlock button clears lock and resets counter.

---

## E14 — Data Deletion & DPDP Controls

---

### C71 — DataDeletionWorkflow (W7) — Part 1: S3 Delete + Extraction PII Nullification

**Epic:** E14 | **Backlog task:** T14.1 part 1 | **AI:** 🔒

**Goal:** W7 steps 1–5: record initiation, delete S3 files, soft-delete document records, nullify extraction PII.

**Prerequisites:** C04, C20.

**Implementation Prompt:**
```
[Paste master context block]

Implement DataDeletionWorkflow (W7) Part 1 for TaxLens. Reference 07_WORKFLOWS.md W7 exactly.
CRITICAL: PII nullification must not be skipped. Non-retryable errors on PII steps halt workflow and alert ops.

apps/api/src/temporal/workflows/data-deletion.workflow.ts (steps 1–5):

Step 1: Activity: recordDeletionInitiatedActivity(userId, reason)
   Write audit event: user.deletion_requested.

Step 2: Activity: deleteS3DocumentsActivity(userId)
   List all document_versions for user. Hard-delete all S3 objects (s3_key in each row).
   S3 delete is idempotent (missing object → success). Batch delete (max 1000 per call).

Step 3: Activity: softDeleteDocumentRecordsActivity(userId)
   UPDATE documents SET deleted_at=now() WHERE user_id=$1.
   UPDATE document_versions SET virus_scan_status='deleted' WHERE document_id IN (SELECT id FROM documents WHERE user_id=$1).

Step 4: Activity: nullifyExtractionPIIActivity(userId)
   UPDATE extraction_results SET extracted_fields = extracted_fields - 'employee_pan' - 'employee_name'
   WHERE extraction_job_id IN (SELECT ej.id FROM extraction_jobs ej JOIN document_versions dv ON ... WHERE user_id=$1).
   Non-retryable-on-DB-error: halt workflow; emit CRITICAL alert.

Step 5: Activity: nullifyComputationPIIActivity(userId)
   UPDATE computation_runs SET input_snapshot = jsonb_set(input_snapshot, '{user_id}', '"DELETED"'::jsonb) WHERE user_id=$1.
   Application-layer regex on input_snapshot to null PAN-like patterns before update.

Each activity: 5 retries, 10s exponential backoff.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/data-deletion.workflow.ts (steps 1–5)
apps/api/src/temporal/activities/data-deletion.activities.ts (steps 1–5)
apps/api/src/migrations/V003c__add_deleted_at_to_versions.sql
```

**Acceptance Criteria:**
- After steps 1-5: all S3 objects for user deleted (verify via aws s3 ls).
- document_versions.deleted_at is set.
- extracted_fields no longer contains employee_pan.
- Activity failure mid-workflow leaves already-completed activities intact (Temporal re-runs from failure point).

---

### C72 — DataDeletionWorkflow (W7) — Part 2: User Nullification + Audit Retention

**Epic:** E14 | **Backlog task:** T14.1 part 2 | **AI:** 🔒

**Goal:** W7 steps 6–11: nullify user PII, revoke sessions, retain audit records, send final email.

**Prerequisites:** C71.

**Implementation Prompt:**
```
[Paste master context block]

Implement DataDeletionWorkflow (W7) Part 2 — steps 6–11. Reference 07_WORKFLOWS.md W7.
CRITICAL: audit_events and consent_records MUST NOT be deleted under any circumstances.

Continuing data-deletion.workflow.ts:

Step 6: Activity: nullifyUserPIIActivity(userId)
   UPDATE users SET pan_encrypted=NULL, phone_encrypted=NULL, totp_secret_encrypted=NULL,
   full_name='DELETED_USER', updated_at=now() WHERE id=$1.
   Non-retryable on error: halt; alert ops.

Step 7: Activity: revokeAllSessionsActivity(userId)
   UPDATE sessions SET revoked_at=now() WHERE user_id=$1 AND revoked_at IS NULL.

Step 8: Activity: softDeleteUserActivity(userId)
   UPDATE users SET deleted_at=now() WHERE id=$1.

Step 9: Activity: retainAuditEventsActivity(userId)
   SELECT COUNT(*) FROM audit_events WHERE user_id=$1 — if 0, log warning.
   DO NOT delete audit_events or consent_records. This step is a VERIFICATION GATE only.
   Log: "Audit events retained for user_id={userId}: {count} records".

Step 10: Activity: notifyUserDeletionCompleteActivity(userId, emailAddress)
   emailAddress stored from workflow input (before nullification removes it from DB).
   Send via EmailService: "Your TaxLens account has been deleted."
   Include: data deleted, audit logs retained as required by law.

Step 11: Activity: emitFinalAuditEventActivity(userId)
   Write audit event user.data_deleted via W6 (for durability).
   actor_role='system'. Event persists even after user is soft-deleted.
```

**Expected Files:**
```
apps/api/src/temporal/workflows/data-deletion.workflow.ts (completed with steps 6–11)
apps/api/src/temporal/activities/data-deletion.activities.ts (steps 6–11 added)
```

**Acceptance Criteria:**
- After full W7: users.full_name='DELETED_USER', pan_encrypted=NULL, deleted_at is set.
- audit_events for user still exist (not deleted).
- consent_records for user still exist (not deleted).
- user.data_deleted audit event written with actor_role='system'.

---

### C73 — User Deletion API Endpoints

**Epic:** E14 | **Backlog tasks:** T14.2, T14.3 | **AI:** ⚠️

**Goal:** `DELETE /users/me` and `DELETE /admin/users/{id}/data` (admin with maker-checker).

**Prerequisites:** C71, C72, C21.

**Implementation Prompt:**
```
[Paste master context block]

Implement user data deletion API endpoints. Reference 06_API.md DELETE /users/me.

1. users.service.ts initiateUserDeletion(userId, confirmation):
   Validate: confirmation === 'DELETE MY ACCOUNT' (exact, case-sensitive) → 422 if not.
   Check user exists and not already deleted.
   Store user's email in Redis 'deletion_email:{userId}' TTL 7d (before W7 nullifies it).
   Start dataDeletionWorkflow(userId, 'user_request') via TemporalService.
   Emit audit event: user.deletion_requested.
   Return { message: 'Deletion scheduled...', deletion_job_id }.

2. DELETE /users/me in users.controller.ts:
   Body: { confirmation: 'DELETE MY ACCOUNT' }. Returns 202.

3. Admin deletion: POST /admin/users/:id/initiate-deletion (admin only):
   Body: { reason: string required }.
   Does NOT start W7 directly. Creates pending_action via PendingActionsService.
   Starts MakerCheckerWorkflow (W9) for action_type='delete_user_data'.
   Returns 202 { pending_action_id }.
   W9's executeActionActivity dispatches to dataDeletionWorkflow on approval.

4. Update apps/web settings/data/page.tsx: wire DELETE /users/me call to deletion modal confirm.
```

**Expected Files:**
```
apps/api/src/users/users.service.ts (updated)
apps/api/src/users/users.controller.ts (DELETE /users/me)
apps/api/src/admin/admin.controller.ts (POST /admin/users/:id/initiate-deletion)
```

**Acceptance Criteria:**
- DELETE /users/me with wrong confirmation string returns 422.
- With correct string: returns 202 and starts W7 in Temporal.
- Admin initiate-deletion creates pending_action (not immediate deletion).
- Admin approval starts W7 via W9 dispatch.

---

### C74 — Consent Withdrawal Blocked State

**Epic:** E14 | **Backlog task:** T14.4 | **AI:** ✅

**Goal:** Frontend blocked state after withdrawal; backend blocking of all CS-05–CS-14 actions.

**Prerequisites:** C18, C55.

**Implementation Prompt:**
```
[Paste master context block]

Implement consent withdrawal frontend blocked state. Reference 10_COMPLIANCE Consent Gate case 2.

1. apps/web/src/lib/api-client.ts (update response interceptor):
   On 403 with code='CONSENT_WITHDRAWN': do NOT redirect to login.
   Redirect to /settings/data?reason=consent_withdrawn.

2. apps/web/src/app/(app)/settings/data/page.tsx (update):
   If reason=consent_withdrawn query param: show persistent non-dismissable banner:
   "Your consent has been withdrawn. Your data is scheduled for deletion.
   You cannot use TaxLens features while deletion is pending."

3. Deletion-in-progress overlay (apps/web/src/app/(app)/layout.tsx update):
   On GET /users/me: if deleted_at is not null: render full-page overlay:
   "Account deletion in progress. You will receive a confirmation email when complete."
   No other navigation available while overlay is shown.

4. apps/api/src/consent/consent.service.ts (update grantConsent):
   Before granting: check admin_pending_actions for pending 'delete_user_data' action for this user.
   If found: throw 409 ConflictException('Account deletion is in progress. New consent cannot be recorded.').
```

**Expected Files:**
```
apps/web/src/lib/api-client.ts (updated interceptor)
apps/web/src/app/(app)/settings/data/page.tsx (updated)
apps/web/src/app/(app)/layout.tsx (updated with deletion overlay)
apps/api/src/consent/consent.service.ts (updated)
```

**Acceptance Criteria:**
- CONSENT_WITHDRAWN 403 response redirects to /settings/data, not login.
- While deletion in progress: overlay shown; normal navigation blocked.
- POST /consent during deletion returns 409.

---

## E15 — Security Hardening

---

### C75 — Rate Limiting, Security Headers, Input Sanitization

**Epic:** E15 | **Backlog tasks:** T15.1, T15.4, T15.5 | **AI:** ✅

**Goal:** NestJS Throttler rate limiting, Helmet security headers, strict ValidationPipe.

**Prerequisites:** C03.

**Implementation Prompt:**
```
[Paste master context block]

Implement security hardening for TaxLens. Reference 10_COMPLIANCE Section 2 (rate limiting values).

1. Rate limiting (@nestjs/throttler) in app.module.ts ThrottlerModule.forRootAsync:
   Default: 100 req/60s per IP.
   Custom @Throttle() overrides:
   Auth routes (login, register, forgot-password): 10/60s per IP.
   POST /documents/upload-url: 20/3600s per user ID.
   POST /computations: 10/3600s per user ID.
   Custom ThrottlerGuard: use user ID (from JWT) for authed routes; IP for public routes.
   On exceeded: 429 TooManyRequestsException { retry_after: seconds } in header.

2. Helmet in main.ts: app.use(helmet()) with:
   contentSecurityPolicy: default-src 'none' (API returns JSON only).
   hsts: maxAge 31536000, includeSubDomains. X-Frame-Options: DENY. X-Content-Type-Options: nosniff.
   In apps/web/next.config.js: add headers() for X-Frame-Options DENY, nosniff, Referrer-Policy,
   Permissions-Policy, CSP for web (script-src 'self'; style-src 'self' 'unsafe-inline').

3. In main.ts: app.useGlobalPipes(new ValidationPipe({
   whitelist: true, forbidNonWhitelisted: true, transform: true,
   transformOptions: { enableImplicitConversion: true } })).
   CORS: allow only NEXT_PUBLIC_WEB_URL origin in production.
```

**Expected Files:**
```
apps/api/src/main.ts (updated with Helmet, ValidationPipe, CORS, Throttler)
apps/api/src/app.module.ts (ThrottlerModule)
apps/api/src/auth/guards/custom-throttler.guard.ts
apps/web/next.config.js (security headers)
```

**Acceptance Criteria:**
- 11 rapid calls to POST /auth/login from same IP: 10 reach business logic, 11th returns 429 with retry_after header.
- `curl -I http://localhost:3001` shows X-Frame-Options, X-Content-Type-Options, HSTS headers.
- Sending `{ "email": "...", "password": "...", "admin": true }` returns 422 'property admin should not exist'.

---

### C76 — KMS Encryption Service (Production)

**Epic:** E15 | **Backlog task:** T15.2 | **AI:** 🔒

**Goal:** Production KMS EncryptionService replacing the stub from C15. Security-critical.

**Prerequisites:** C05 (KMS key), C15 (stub).

**Implementation Prompt:**
```
[Paste master context block]

Implement the production KMS EncryptionService for TaxLens — replacing the stub from C15.
SECURITY-CRITICAL. DO NOT LOG RETURN VALUES OR ARGUMENTS.

apps/api/src/encryption/encryption.service.ts (replace stub):
Use AWS SDK v3: @aws-sdk/client-kms.

encrypt(plaintext: string): Promise<Buffer>:
  // DO NOT LOG RETURN VALUE OR ARGUMENTS
  Call GenerateDataKeyCommand: KeyId=KMS_KEY_ARN, KeySpec='AES_256'.
  → Returns { CiphertextBlob (encrypted DEK), Plaintext (raw AES-256 key) }.
  iv = crypto.randomBytes(12) (96-bit GCM IV).
  cipher = crypto.createCipheriv('aes-256-gcm', plaintextKey, iv).
  encryptedData = cipher.update(plaintext) + cipher.final().
  authTag = cipher.getAuthTag() (16 bytes).
  Store encryptedDEK length as first 4 bytes (uint32 big-endian).
  Final: Buffer.concat([lengthBytes (4), encryptedDEK (variable), iv (12), authTag (16), encryptedData]).
  plaintextKey.fill(0) immediately (zeroise).
  Return assembled buffer.

decrypt(ciphertext: Buffer): Promise<string>:
  // DO NOT LOG RETURN VALUE OR ARGUMENTS
  Read first 4 bytes to get encryptedDEK length.
  Slice encryptedDEK → DecryptCommand → plaintextKey.
  Slice iv and authTag from buffer positions.
  decipher = crypto.createDecipheriv + setAuthTag → decrypt → return string.
  plaintextKey.fill(0) immediately.

maskPan(pan: string): string — unchanged from stub.
Add integration test (with LocalStack KMS or real KMS in test environment).
```

**Expected Files:**
```
apps/api/src/encryption/encryption.service.ts (full KMS implementation)
apps/api/src/encryption/encryption.service.spec.ts
```

**Acceptance Criteria:**
- `encrypt('ABCDE1234F')` returns Buffer with length > 0.
- `decrypt(await encrypt('ABCDE1234F'))` returns exactly `'ABCDE1234F'`.
- Ciphertext differs on each call (random IV).
- Plaintext DEK zeroed: buffer content all zeros after encrypt() returns.

---

### C77 — `v_audit_events_safe` View + Final Integration Check

**Epic:** E15 | **Backlog task:** T15.3 | **AI:** ✅

**Goal:** Apply the safe audit view for operators; run final integration smoke test for v1 release.

**Prerequisites:** C09 (migration already in V003), C76.

**Implementation Prompt:**
```
[Paste master context block]

Implement the v_audit_events_safe view usage enforcement and run the final integration check.

1. apps/api/src/audit/audit.service.ts (update):
   Add getOperatorSafeEvents(filters) method:
   Queries v_audit_events_safe view (not audit_events directly) for operator-role callers.
   Queries audit_events directly for admin-role callers.
   Role check: inject request context to determine caller role.

2. apps/api/src/admin/audit.controller.ts:
   GET /admin/audit: use getOperatorSafeEvents() for operators; raw table for admins.
   GET /admin/audit/raw: admin-only; always queries audit_events directly;
   emits admin.user_data_viewed audit event.

3. Final integration smoke test script (apps/api/test/e2e/smoke.e2e-spec.ts):
   Test the complete salaried ITR-1 flow:
   a. Register + verify email.
   b. Grant consent.
   c. Upload Form 16 (use a test PDF fixture).
   d. Confirm upload; wait for extraction to complete (poll with timeout 60s).
   e. Lock extraction.
   f. Create tax profile; add 80C deduction.
   g. Trigger computation (both regimes).
   h. Verify computation result contains disclaimer field.
   i. Generate report; verify download URL returned.
   j. Check audit_events contains all expected events.
   k. DELETE /users/me; verify W7 completes; verify documents deleted from S3.
   This test validates the full product lifecycle end-to-end.
```

**Expected Files:**
```
apps/api/src/audit/audit.service.ts (updated with role-aware view selection)
apps/api/src/admin/audit.controller.ts (updated)
apps/api/test/e2e/smoke.e2e-spec.ts
```

**Acceptance Criteria:**
- GET /admin/audit with operator role queries v_audit_events_safe (IP masked in response).
- GET /admin/audit with admin role returns raw IP (not masked).
- smoke.e2e-spec.ts passes end-to-end with all steps completing without error.
- Disclaimer field present in computation result at step h.

---

*Artifact status: COMPLETE — all 77 implementation chunks defined.*
*Chunk ordering respects dependency chain: E01 → E02 → E03 → E04 → E05 → E06 → E07 → E08 → E09 → E10 → E11 → E12 → E13 → E14 → E15.*
*Begin at C01. Complete each chunk before starting the next. Save all files from each chunk before opening a new Claude session.*
