# 13 — Delivery Backlog

> **Role:** Technical Program Manager
> **Inputs:** 01–12, STITCH_01 through STITCH_03
> **Complexity:** S = < 4h | M = ~1 day | L = 2–3 days | XL = 1 week+
> **Tags:** `fe` frontend | `be` backend | `py` Python worker | `db` database | `ops` DevOps | `comp` compliance | `prod` product
> **AI-assist:** ✅ Claude-generated first draft suitable | ⚠️ human review required | 🔒 human-led only

---

## Epic List

| # | Epic | Tag(s) | Dependency |
|---|------|--------|-----------|
| E01 | Project Infrastructure & DevOps | `ops`, `db` | None |
| E02 | Auth & Identity | `be`, `fe` | E01 |
| E03 | Consent & Compliance Foundation | `be`, `db`, `comp` | E02 |
| E04 | Document Ingestion | `be`, `ops` | E03 |
| E05 | OCR Extraction Workers | `py`, `be` | E04 |
| E06 | Extraction Review UI | `fe`, `be` | E05 |
| E07 | Tax Profile & Deduction Inputs | `be`, `fe`, `db` | E02 |
| E08 | Tax Engine | `be`, `py`, `db` | E07 |
| E09 | Report Generation | `py`, `be` | E08 |
| E10 | Frontend — Auth & Onboarding | `fe` | E02, E03 |
| E11 | Frontend — Documents & Extraction | `fe` | E04, E05, E06 |
| E12 | Frontend — Estimate & Reports | `fe` | E08, E09 |
| E13 | Admin Console | `fe`, `be` | E02–E09 |
| E14 | Data Deletion & DPDP Controls | `be`, `py`, `comp` | E03, E04 |
| E15 | Security Hardening | `be`, `db`, `comp` | E02 |

---

## Feature Breakdown and Task Table

---

### E01 — Project Infrastructure & DevOps

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T01.1 | Monorepo scaffolding | Initialize monorepo with NestJS (backend), Next.js (frontend), Python workers directories; shared types package | `ops` | M | — | ✅ |
| T01.2 | Docker setup | Dockerfile for NestJS API; Dockerfile for Python extraction worker; Dockerfile for report worker; docker-compose for local dev (API + workers + PG + Redis + Temporal) | `ops` | M | T01.1 | ✅ |
| T01.3 | Database provisioning | PostgreSQL 15 instance (RDS or local); PgBouncer connection pooling config | `db`, `ops` | S | T01.1 | ✅ |
| T01.4 | Redis provisioning | Redis 7 instance; BullMQ config; connection settings | `ops` | S | T01.1 | ✅ |
| T01.5 | Temporal setup | Temporal Cloud account OR self-hosted Temporal server container; worker registration for NestJS and Python workers | `ops` | M | T01.2 | ⚠️ |
| T01.6 | S3 bucket setup | ap-south-1 bucket for documents; bucket for reports; bucket for OCR raw responses; server-side encryption (SSE-S3 or SSE-KMS); lifecycle policies per 05_SCHEMA | `ops` | M | — | ⚠️ |
| T01.7 | KMS setup | AWS KMS key for PAN/phone field encryption; key rotation policy; IAM policies for application access | `ops`, `comp` | M | — | 🔒 |
| T01.8 | CI/CD pipeline | GitHub Actions: lint + type check + test on PR; Docker build + push on merge to main; staging deploy | `ops` | L | T01.2 | ✅ |
| T01.9 | Database migrations framework | Flyway or TypeORM migrations; first migration: all tables from 05_SCHEMA + `admin_pending_actions` (STITCH_03 G11) + `v_audit_events_safe` view (G12) | `db` | L | T01.3 | ✅ |

---

### E02 — Auth & Identity

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T02.1 | User registration | `POST /auth/register`; bcrypt hash; unique email; emit verification OTP | `be` | M | T01.9 | ✅ |
| T02.2 | Email verification | OTP generation + send via SES; `POST /auth/verify-email`; expire OTP after 10 min | `be` | M | T02.1 | ✅ |
| T02.3 | JWT auth | `POST /auth/login`; JWT access (15 min) + refresh (7 days) tokens; `POST /auth/refresh`; `POST /auth/logout`; refresh token stored as SHA-256 hash | `be` | M | T02.1 | ⚠️ |
| T02.4 | Password reset | `POST /auth/forgot-password`; `POST /auth/reset-password`; tokenized reset link; 1h expiry | `be` | S | T02.1 | ✅ |
| T02.5 | TOTP 2FA | `POST /auth/2fa/setup`; QR code endpoint; `POST /auth/2fa/verify`; `POST /auth/2fa/disable`; `totp_secret_encrypted` via KMS | `be` | M | T02.3, T01.7 | 🔒 |
| T02.6 | User profile CRUD | `GET /users/me`; `PATCH /users/me`; PAN field: validate format, KMS-encrypt before store, return masked last-4 only | `be` | M | T02.3 | ⚠️ |
| T02.7 | RBAC guard | NestJS `RolesGuard`; `@Roles()` decorator; role extracted from JWT; applied to all protected routes | `be` | S | T02.3 | ✅ |
| T02.8 | Session management | `sessions` table CRUD; revoke-all-for-user method (for W7); background cleanup job for expired sessions | `be`, `db` | S | T02.3 | ✅ |
| T02.9 | Account lockout | Track failed login attempts per email; lock after 10 failures; 10-min lockout; `auth.account_locked` audit event | `be` | S | T02.3 | ✅ |

---

### E03 — Consent & Compliance Foundation

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T03.1 | Consent API | `POST /consent`; `GET /consent/status`; write `consent_records` row; implement Consent Gate middleware (5-case decision from 10_COMPLIANCE Section 2) | `be`, `comp` | M | T02.3 | ⚠️ |
| T03.2 | Audit event service | `AuditEventService.write(event)` method; wraps DB insert; called by all modules via interface (never direct DB writes from business logic) | `be`, `comp` | S | T01.9 | ✅ |
| T03.3 | ComplianceEventWorkflow (W6) | Temporal workflow: durable `WriteAuditEventActivity` with unlimited retry; accepts event payload; used for all critical compliance events | `be`, `py` | M | T01.5, T03.2 | ✅ |
| T03.4 | Audit log API (user-facing) | `GET /audit/me`; paginated; cursor-based; IP + action + resource per event; 50 events per page | `be` | S | T03.2 | ✅ |
| T03.5 | Admin pending actions table | DB migration for `admin_pending_actions` table (STITCH_03 G11); NestJS service for create/approve/reject; 48h expiry job | `be`, `db` | M | T01.9 | ✅ |
| T03.6 | MakerCheckerWorkflow (W9) | Temporal workflow: initiate pending action → wait for approval signal → execute action or expire (STITCH_03 G14) | `be` | M | T01.5, T03.5 | ✅ |
| T03.7 | Admin approvals API | `GET /admin/approvals`; `POST /admin/approvals/{id}/approve`; `POST /admin/approvals/{id}/reject`; self-approval prevention guard | `be`, `comp` | M | T03.5 | ✅ |

---

### E04 — Document Ingestion

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T04.1 | Pre-signed URL endpoint | `POST /documents/upload-url`; validate MIME type, file size, document type, AY format, active consent; issue S3 pre-signed PUT URL (15 min expiry); create `documents` + `document_versions` rows | `be` | M | T03.1, T01.6 | ✅ |
| T04.2 | Upload confirm endpoint | `POST /documents/{version_id}/confirm-upload`; verify S3 object exists (HEAD request); update `document_versions.virus_scan_status = 'pending'`; enqueue `VirusScanJob` via BullMQ | `be` | S | T04.1 | ✅ |
| T04.3 | Virus scan worker | BullMQ consumer; download file from S3 (pre-signed URL); run ClamAV scan; update `virus_scan_status`; on pass: start `DocumentIngestionWorkflow` (W1); on fail: quarantine + notify | `py` or `be` | M | T04.2, T01.4 | ⚠️ |
| T04.4 | DocumentIngestionWorkflow (W1) | Temporal workflow: UpdateDocumentStatus → VirusScanActivity → pass/fail branch → signal ExtractionWorkflow; timeout 5 min (per 07_WORKFLOWS) | `be` | M | T01.5, T04.3 | ✅ |
| T04.5 | Document list + detail API | `GET /documents`; `GET /documents/{id}`; `DELETE /documents/{id}` (soft); query filters: AY, document_type | `be` | S | T04.1 | ✅ |

---

### E05 — OCR Extraction Workers

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T05.1 | OCR vendor adapter | Abstract `OCRVendorAdapter` interface + concrete implementation (AWS Textract or selected vendor); accepts file bytes or S3 key; returns normalized raw response | `py` | M | T01.6 | ⚠️ |
| T05.2 | Form 16 mapper | `Form16Mapper`: Part A + Part B field extraction; regex patterns for all Must-Have fields (employer name, TAN, employee PAN, gross salary, TDS deducted, taxable income); confidence scoring | `py` | L | T05.1 | ⚠️ |
| T05.3 | AIS mapper | `AISMapper`: salary entries, TDS entries, interest income, advance tax from AIS PDF and CSV formats | `py` | L | T05.1 | ⚠️ |
| T05.4 | Form 26AS mapper | `Form26ASMapper`: Part A TDS entries, Part B non-salary TDS, Part C advance tax | `py` | M | T05.1 | ⚠️ |
| T05.5 | Salary slip mapper | `SalarySlipMapper`: gross, basic, HRA component, PF deduction, net pay; handles multiple payroll vendor formats (Zoho, Keka, generic PDF) | `py` | L | T05.1 | ⚠️ |
| T05.6 | Interest certificate mapper | `InterestCertMapper`: bank name, account type, interest amount, TDS deducted | `py` | M | T05.1 | ✅ |
| T05.7 | Confidence scorer | `compute_field_confidence(vendor_confidence, heuristic_match)` function per 08_EXTRACTION spec; `overall_confidence = min(Must-Have field scores)` | `py` | S | T05.1 | ✅ |
| T05.8 | ExtractionWorkflow (W2) | Temporal workflow: FetchDocument → OCRVendor → MapFields → StoreResult → EvaluateConfidence → notify (per 07_WORKFLOWS); large payload stored to S3 not Temporal history | `py`, `be` | L | T01.5, T05.1–T05.7 | ✅ |
| T05.9 | Extraction API | `GET /extractions/{job_id}`; `GET /extractions/{job_id}/result`; `PATCH /extractions/{result_id}/corrections`; `POST /extractions/{result_id}/lock` | `be` | M | T05.8 | ✅ |
| T05.10 | Cross-document reconciliation hints | After both Form 16 and 26AS present for same AY: compare TDS amounts; flag mismatches as `reconciliation_warnings[]` on ExtractionResult | `py` | M | T05.2, T05.4 | ⚠️ |
| T05.11 | ExtractionRetryWorkflow (W8) | Temporal workflow for ops escalation on W2 exhaustion; create ops alert; notify user | `be` | S | T05.8 | ✅ |

---

### E06 — Extraction Review UI

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T06.1 | Extraction review screen (S-08) | Side-by-side PDF viewer (PDF.js) + field list panel; per-field confidence bars; low-confidence highlighting; source tag chips | `fe` | XL | T05.9 | ⚠️ |
| T06.2 | Inline field editing | Click-to-edit field value; save/cancel; note input; POST to `/extractions/{result_id}/corrections` | `fe` | M | T06.1 | ✅ |
| T06.3 | Correction history drawer | Per-field: show correction log with original value, corrected value, timestamp | `fe` | S | T06.2 | ✅ |
| T06.4 | Lock extraction flow | "Lock Extraction" sticky CTA; disabled state when Must-Have fields unresolved; confirmation modal; POST to `/extractions/{result_id}/lock` | `fe` | S | T06.1 | ✅ |
| T06.5 | TDS reconciliation warning UI | Orange warning banner when `reconciliation_warnings[]` present; field-level callout linking to conflicting document | `fe` | S | T06.1 | ✅ |
| T06.6 | HumanReviewNotificationWorkflow (W3) | Temporal workflow: notify user of review required; 72h timer; send reminder; log abandoned reviews | `be` | M | T05.8 | ✅ |

---

### E07 — Tax Profile & Deduction Inputs

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T07.1 | Tax profile API | `POST /tax-profiles`; `GET /tax-profiles/{ay}`; validates AY format; unique constraint (user_id, AY) | `be` | S | T02.3 | ✅ |
| T07.2 | Deduction inputs API | `POST /tax-profiles/{ay}/deductions`; `DELETE /tax-profiles/{ay}/deductions/{id}`; permitted section_code enum (80C, 80D, HRA, 80TTA, 80TTB, 80G, 80CCD1B, 80CCD2, professional_tax, LTA) | `be` | M | T07.1 | ✅ |
| T07.3 | Tax rules seed data | Define and seed AY 2025-26 old regime + new regime rule JSON objects (STITCH_02 G6); validate against 09_TAX_ENGINE rule object structure; store via `POST /admin/tax-rules` or migration | `db`, `comp` | L | T01.9 | 🔒 |
| T07.4 | Tax rules admin API | `GET /admin/tax-rules`; `POST /admin/tax-rules` (creates inactive version + pending approval); `PUT /admin/tax-rules/{id}/activate` (maker-checker) | `be` | M | T07.3, T03.7 | ✅ |

---

### E08 — Tax Engine

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T08.1 | AssembleComputationInputs | `AssembleComputationInputsActivity`: collect all locked ExtractionResults for user+AY; merge with DeductionInputs; produce `ComputationInput` typed object; validate required fields | `be` | M | E05, E07 | ✅ |
| T08.2 | Tax slab computation | Implement slab-wise tax calculation from rule JSON; handle both old and new regime slabs; return `slab_breakdown[]` array (per 09_TAX_ENGINE Section 5) | `be` | M | T07.3 | ⚠️ |
| T08.3 | Deduction application (old regime) | Apply 80C cap (₹1,50,000); 80D caps (₹25,000/50,000); 80TTA (₹10,000 cap); 80G (50%/100% sub-limit support); 80CCD(1B) (₹50,000 cap); HRA deduction; professional tax | `be` | L | T08.2 | ⚠️ |
| T08.4 | New regime filter | Accept deduction inputs but mark all non-permitted deductions as `ignored_in_new_regime`; apply only standard_deduction (₹75,000) and 80CCD(2); return notes in output | `be` | S | T08.2 | ✅ |
| T08.5 | Surcharge + marginal relief | Apply surcharge thresholds from rule JSON; compute marginal relief when surcharge > incremental income above threshold (STITCH_02 G10) | `be` | M | T08.2 | ⚠️ |
| T08.6 | Section 87A rebate | Check taxable income ≤ threshold from rule JSON; apply rebate up to max_rebate; return 0 if over threshold | `be` | S | T08.2 | ✅ |
| T08.7 | TDS reconciliation | Sum TDS credits from input; compare to total tax liability; produce `net_payable` / `net_refundable`; generate reconciliation warnings if TDS source has gaps | `be` | M | T08.2 | ✅ |
| T08.8 | Computation engine integration | Assemble full `ComputationResult` object (per 09_TAX_ENGINE Section 4); validate; store as immutable `computation_runs` row; include `input_snapshot` + `rule_version` | `be` | M | T08.1–T08.7 | ✅ |
| T08.9 | ComputationWorkflow (W4) | Temporal workflow: ValidatePrereqs → AssembleInputs → LoadRules → RunEngine → StoreResult → AuditEvent; handle 'compare' mode as two W4 runs (STITCH_02 C2) | `be` | M | T01.5, T08.8 | ✅ |
| T08.10 | Computation API | `POST /computations`; `GET /computations/{id}`; `GET /computations?assessment_year=` | `be` | S | T08.9 | ✅ |
| T08.11 | Tax engine unit tests | Test cases: standard salaried ITR-1 (old + new); 87A rebate edge case; surcharge + marginal relief; deduction cap enforcement; TDS > tax (refund scenario) | `be`, `comp` | L | T08.1–T08.8 | ⚠️ |

---

### E09 — Report Generation

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T09.1 | PDF template design | HTML/CSS template with: income summary, deductions detail, slab breakdown table, TDS reconciliation, regime comparison, disclaimer footer (D-04), version and date | `py`, `prod` | L | — | ✅ |
| T09.2 | PDF rendering worker | WeasyPrint or ReportLab; `RenderPDFActivity`; accepts report data JSON + template; outputs bytes; stores to S3 (not Temporal history) | `py` | M | T09.1, T01.6 | ✅ |
| T09.3 | ReportGenerationWorkflow (W5) | Temporal workflow: AssembleReportData → RenderPDF → StoreToS3 → StoreReportRecord → AuditEvent | `be`, `py` | M | T01.5, T09.2 | ✅ |
| T09.4 | Report API | `POST /reports`; `GET /reports/{id}/download-url` (10-min pre-signed URL); `GET /reports` | `be` | S | T09.3 | ✅ |

---

### E10 — Frontend — Auth & Onboarding

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T10.1 | Login screen (S-01) | Form validation; JWT storage (memory + httpOnly cookie for refresh); redirect logic | `fe` | M | T02.3 | ✅ |
| T10.2 | Register + verify (S-02, S-03) | Registration form; 6-digit OTP input with auto-advance; resend timer | `fe` | M | T02.1, T02.2 | ✅ |
| T10.3 | Consent screen (S-04) | Full-screen modal; D-01 + D-02 disclaimers; dual checkbox; cannot be skipped; calls `POST /consent` | `fe`, `comp` | M | T03.1 | 🔒 |
| T10.4 | Auth guards (client) | `withAuth` HOC / middleware for Next.js; redirect to login if no token; redirect to `/onboarding` if no consent | `fe` | S | T10.1 | ✅ |

---

### E11 — Frontend — Documents & Extraction

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T11.1 | Document list (S-06) | List with filter bar; status badges; delete action; empty state | `fe` | M | T04.5 | ✅ |
| T11.2 | Upload modal | Document type + AY selector; drag-and-drop + browse; direct S3 XHR upload with progress bar; confirm upload call | `fe` | L | T04.1, T04.2 | ⚠️ |
| T11.3 | Extraction progress polling | WebSocket or SSE connection for job status updates; fallback to polling every 5s; surface status on document list + extraction screen | `fe` | M | T05.9 | ✅ |
| T11.4 | Extraction review screen (S-08) | PDF.js viewer; field list panel; confidence bars; inline edit; correction history drawer; lock CTA; TDS mismatch warning; mobile single-column toggle | `fe` | XL | T06.1–T06.6 | ⚠️ |

---

### E12 — Frontend — Estimate & Reports

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T12.1 | Tax profile form (S-09) | Regime selector; deduction accordion sections; 80C progress bar; form save; pre-fill from extractions | `fe` | L | T07.2 | ✅ |
| T12.2 | Estimate dashboard (S-10) | Regime comparison card; computation breakdown accordion; slab table; TDS reconciliation; D-03 disclaimer banner; recompute button | `fe` | XL | T08.10 | ⚠️ |
| T12.3 | Reports screen (S-11) | Report list; generate report modal; download button (pre-signed URL refresh) | `fe` | M | T09.4 | ✅ |
| T12.4 | Activity timeline (S-12) | Chronological audit event feed; filter bar; infinite scroll | `fe` | M | T03.4 | ✅ |
| T12.5 | Settings screens (S-13, S-14, S-15) | Profile edit; PAN update; password change; 2FA setup; data deletion confirmation modal (D-08) | `fe` | M | T02.6, T02.5, E14 | ✅ |
| T12.6 | Dashboard (S-05) | Status card logic; document summary chips; recent activity strip; AY selector (AY 2024-25 disabled in v1) | `fe` | M | T11.1, T08.10 | ✅ |

---

### E13 — Admin Console

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T13.1 | Admin auth enforcement | TOTP 2FA required for admin role; admin route guards; 15-min session timeout | `be`, `fe` | M | T02.5 | 🔒 |
| T13.2 | Overview dashboard (A-01) | Service health banner; job queue summary cards; alerts panel; recent audit events | `fe`, `be` | L | T13.6, T13.3 | ✅ |
| T13.3 | Extraction job queue (A-02) | Filter + table; job detail drawer with full extraction result; re-trigger action | `fe`, `be` | L | T05.9 | ✅ |
| T13.4 | User management (A-03) | Search by email; user detail page with tabs; compliance banner (D-09) on every view; all admin actions logging | `fe`, `be` | L | T02.6 | ⚠️ |
| T13.5 | Computation management (A-04) | Filter by rule_version; rerun action with reason; rule version impact filter | `fe`, `be` | M | T08.10 | ✅ |
| T13.6 | Tax rules management (A-05) | Rule version table; view rule JSON rendered as tables; create new version form; activate via maker-checker | `fe`, `be` | M | T07.4, T03.7 | ✅ |
| T13.7 | Pending approvals (A-06) | Approval queue; approve/reject buttons; self-approval prevention; 48h expiry display | `fe`, `be` | M | T03.7 | ✅ |
| T13.8 | Audit log inspection (A-07) | Search/filter; expandable event JSON; PII masking toggle; CSV export (admin only) | `fe`, `be` | M | T03.2 | ⚠️ |
| T13.9 | System health (A-08) | Service status grid; worker throughput charts; OCR vendor error log; `GET /admin/health` endpoint (STITCH_03 G13) | `fe`, `be` | M | — | ✅ |
| T13.10 | Security monitoring (A-09) | Login failure summary; locked accounts; active admin sessions table | `fe`, `be` | M | T02.9 | ✅ |

---

### E14 — Data Deletion & DPDP Controls

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T14.1 | DataDeletionWorkflow (W7) | Temporal workflow: all 11 steps from 07_WORKFLOWS; PII nullification; S3 hard-delete; session revocation; audit retention; final email | `py`, `be`, `comp` | XL | T01.5, T03.3 | 🔒 |
| T14.2 | User deletion endpoint | `DELETE /users/me` — triggers W7; confirmation string required; returns 202 + deletion_job_id | `be`, `comp` | M | T14.1 | ⚠️ |
| T14.3 | Admin deletion endpoint | `DELETE /admin/users/{id}/data` — requires maker-checker approval first; then triggers W7 | `be`, `comp` | M | T14.1, T03.7 | ⚠️ |
| T14.4 | Consent withdrawal support | On `consent.withdrawn` event: block all subsequent CS-05 to CS-14 actions; show "account pending deletion" state to user | `be`, `fe` | M | T03.1 | ✅ |

---

### E15 — Security Hardening

| Task ID | Feature | Task | Tag | Size | Depends On | AI-Assist |
|---------|---------|------|-----|------|-----------|----------|
| T15.1 | Rate limiting | NestJS `@Throttler`: auth 10/min per IP; upload URL 20/hr per user; computation 10/hr per user | `be` | S | T02.3 | ✅ |
| T15.2 | KMS field encryption service | Shared `EncryptionService.encrypt(plaintext)` / `decrypt(ciphertext)`; wraps AWS KMS; used for PAN, phone, TOTP secret | `be` | M | T01.7 | 🔒 |
| T15.3 | `v_audit_events_safe` view | DB view applying PII masking rules (10_COMPLIANCE Section 3); used by operator audit screen | `db`, `comp` | S | T01.9 | ✅ |
| T15.4 | Security headers | NestJS Helmet: HSTS, CSP, X-Frame-Options, X-Content-Type-Options; Next.js `next.config.js` headers | `be`, `fe` | S | T01.1 | ✅ |
| T15.5 | Input sanitization | NestJS `ValidationPipe` with `whitelist: true, forbidNonWhitelisted: true` on all DTOs; strip unknown properties | `be` | S | T02.1 | ✅ |

---

## Dependencies Summary

```
E01 → E02 → E03 → E04 → E05 → E06
                  E04        ↘
              E07 ──────────── E08 → E09
E02 ─────────────────────────────────────→ E10
E04, E05, E06 ───────────────────────────→ E11
E08, E09 ────────────────────────────────→ E12
E02–E09 ─────────────────────────────────→ E13
E03, E04 ────────────────────────────────→ E14
E02 ─────────────────────────────────────→ E15
```

---

## Suggested Sprint Order (2-week sprints)

| Sprint | Epics | Goal |
|--------|-------|------|
| S1 | E01, E02 | Infrastructure up; auth working end-to-end |
| S2 | E03, E15 | Consent gate working; security baseline; audit writes |
| S3 | E04, E05.1–T05.4 | Upload flow working; Form 16 + AIS extraction |
| S4 | E05.5–T05.11, E06 | All mappers; extraction review UI; lock flow |
| S5 | E07, E08.1–T08.8 | Tax profile; tax engine (old + new regime) |
| S6 | E08.9–T08.11, E09 | Computation workflow; report PDF |
| S7 | E10, E11 | Frontend: auth, upload, extraction review |
| S8 | E12 | Frontend: estimate dashboard, reports |
| S9 | E13 | Admin console |
| S10 | E14, integration testing | Data deletion; end-to-end test of full ITR-1 flow |

---

*Artifact status: BACKLOG COMPLETE.*
*Open items from STITCH_03 all assigned to tasks above (G6→T07.3, G10→T08.5, G11→T01.9/T03.5, G12→T15.3, G13→T13.9, G14→T03.6, C8→T03.7).*
*Next: Prompt 14 (Claude-sized Coding Chunks).*
