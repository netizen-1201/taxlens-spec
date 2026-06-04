# 07 — Workflow Orchestration (Temporal)

> **Role:** Workflow Orchestration Expert
> **Engine:** Temporal (Temporal Cloud for v1; self-hosted option viable)
> **Worker languages:** Python (extraction, report) + NestJS (light event workflows)
> **Queue strategy:** Temporal for durable multi-step workflows; BullMQ/Redis for fire-and-forget tasks (virus scan)

---

## Workflow List

| # | Workflow | Triggered By | Priority |
|---|---------|-------------|---------|
| W1 | `DocumentIngestionWorkflow` | Upload confirmed | High |
| W2 | `ExtractionWorkflow` | Virus scan passed | High |
| W3 | `HumanReviewNotificationWorkflow` | Low confidence detected | Medium |
| W4 | `ComputationWorkflow` | User triggers computation | High |
| W5 | `ReportGenerationWorkflow` | User triggers report | Medium |
| W6 | `ComplianceEventWorkflow` | Any audit-worthy action | Medium |
| W7 | `DataDeletionWorkflow` | Consent withdrawal / DELETE /users/me | High |
| W8 | `ExtractionRetryWorkflow` | Extraction failure | High |

---

## W1 — `DocumentIngestionWorkflow`

**Trigger:** `POST /documents/{version_id}/confirm-upload`
**Purpose:** Orchestrate virus scan → queue extraction → update document status

```
DocumentIngestionWorkflow(document_version_id)
  │
  ├── Activity: UpdateDocumentStatus(version_id, 'PENDING_SCAN')
  ├── Activity: VirusScanActivity(version_id)
  │     ├── [pass]  → UpdateDocumentStatus(version_id, 'PASSED')
  │     │             → Signal ExtractionWorkflow (start)
  │     └── [fail]  → UpdateDocumentStatus(version_id, 'QUARANTINED')
  │                   → NotifyUserActivity(user_id, 'document_quarantined')
  │                   → emit audit_event: document.quarantined
  └── [timeout 5min] → UpdateDocumentStatus(version_id, 'SCAN_TIMEOUT')
                       → Alert ops queue
```

**Activities:**
- `UpdateDocumentStatus(version_id, status)` — write to `document_versions` table.
- `VirusScanActivity(version_id)` — download file from S3 (pre-signed), run ClamAV or vendor API, return `pass`/`fail`.
- `NotifyUserActivity(user_id, event_type)` — send email via SES; non-blocking (failure does not fail workflow).

**Idempotency:** `document_version_id` is the idempotency key. Re-triggering with the same ID is safe (idempotent UpdateDocumentStatus).
**Retry:** `VirusScanActivity` retries 3× with 10s exponential backoff. Timeout after 5 minutes total → alert ops.
**Non-retryable failures:** `QUARANTINED` result from scanner — no retry.

---

## W2 — `ExtractionWorkflow`

**Trigger:** Signal from W1 (virus scan pass)
**Purpose:** Call OCR vendor, map fields, score confidence, store result, update status

```
ExtractionWorkflow(extraction_job_id, document_version_id, document_type)
  │
  ├── Activity: UpdateExtractionJobStatus(job_id, 'PROCESSING')
  ├── Activity: FetchDocumentFromS3Activity(version_id) → raw bytes
  ├── Activity: CallOCRVendorActivity(raw_bytes, document_type)
  │             → ocr_raw_response
  ├── Activity: MapFieldsActivity(ocr_raw_response, document_type)
  │             → extraction_result (fields + confidence_scores)
  ├── Activity: StoreExtractionResultActivity(job_id, extraction_result)
  ├── Activity: EvaluateConfidenceActivity(extraction_result)
  │     ├── [all fields ≥ 0.75] → UpdateExtractionJobStatus(job_id, 'COMPLETED')
  │     │                         → NotifyUser(user_id, 'extraction_ready_for_review')
  │     └── [any field < 0.75]  → UpdateExtractionJobStatus(job_id, 'NEEDS_REVIEW')
  │                               → Signal HumanReviewNotificationWorkflow
  └── emit audit_event: extraction.completed / extraction.needs_review
```

**Activities:**
- `FetchDocumentFromS3Activity` — generates a short-lived pre-signed URL; does not pass raw bytes through Temporal history (passes S3 key only; bytes fetched in-activity).
- `CallOCRVendorActivity` — vendor-specific; result is normalized to internal schema. Retryable.
- `MapFieldsActivity` — pure function; applies `Form16Mapper`, `AISMapper`, etc. Idempotent.
- `StoreExtractionResultActivity` — upsert with version tracking.
- `EvaluateConfidenceActivity` — reads threshold from config (default 0.75); writes `low_confidence_fields[]`.

**Idempotency:** `extraction_job_id` is the workflow ID; Temporal deduplicates re-triggers automatically.
**Retryable failures:** OCR vendor timeout, S3 fetch error — retry 3× with backoff.
**Non-retryable:** OCR vendor authentication error (configuration issue; alert ops immediately).
**Large payload note:** OCR raw response can be large. Store vendor response to S3 and pass S3 key through Temporal activities — do NOT store large blobs in Temporal workflow history.

**State model:**
`QUEUED` → `PROCESSING` → `COMPLETED` | `NEEDS_REVIEW` | `FAILED`

---

## W3 — `HumanReviewNotificationWorkflow`

**Trigger:** Signal from W2 when `status = NEEDS_REVIEW`
**Purpose:** Notify user to review flagged fields; track whether review is completed

```
HumanReviewNotificationWorkflow(extraction_job_id, user_id)
  │
  ├── Activity: NotifyUserActivity(user_id, 'review_required', fields=[...])
  ├── Timer: Wait up to 72 hours for user action
  ├── Signal: ExtractionLockedSignal (user locked the extraction in UI)
  │     └── → Complete workflow: emit audit_event: extraction.review_completed
  └── [timeout 72h] → SendReminderActivity(user_id)
                      → Wait further 48h
                      → [no action] → emit audit_event: extraction.review_abandoned
```

**Note:** This workflow does not block computation — the user can lock even a `NEEDS_REVIEW` extraction after manually verifying fields in the UI. The workflow is informational.

---

## W4 — `ComputationWorkflow`

**Trigger:** `POST /computations` API endpoint
**Purpose:** Validate inputs, run tax engine, store immutable ComputationRun

```
ComputationWorkflow(user_id, assessment_year, regime)
  │
  ├── Activity: ValidateComputationPrerequisitesActivity
  │             (check: locked extractions exist, tax profile exists)
  ├── Activity: AssembleComputationInputsActivity
  │             → collect all locked ExtractionResults + DeductionInputs
  │             → produce normalized InputSnapshot (JSON)
  ├── Activity: LoadTaxRulesActivity(assessment_year, regime)
  │             → fetch active TaxRules record
  ├── Activity: RunTaxEngineActivity(input_snapshot, tax_rules)
  │             → computation_result (JSON with full breakdown)
  ├── Activity: StoreComputationRunActivity(result, input_snapshot, rule_version)
  │             → computation_run_id (immutable)
  └── emit audit_event: computation.completed
```

**Idempotency:** Engine is stateless and deterministic. Re-runs with same inputs produce same result; stored as a new `ComputationRun` row regardless (each run is its own record).
**Retryable:** `LoadTaxRulesActivity` (DB read), `AssembleComputationInputsActivity` (DB read). Retry 3×.
**Non-retryable:** `RunTaxEngineActivity` throwing a validation error (missing required fields) — fail immediately, return error to caller with field-level details.
**Synchronous note:** For v1, computation is fast enough (<3s) to make this nearly synchronous. Temporal adds auditability even for fast workflows.

---

## W5 — `ReportGenerationWorkflow`

**Trigger:** `POST /reports` API endpoint
**Purpose:** Assemble data → render PDF → store → issue download URL

```
ReportGenerationWorkflow(computation_run_ids[], report_type)
  │
  ├── Activity: AssembleReportDataActivity(computation_run_ids)
  │             → fetch ComputationRun results, user profile, document summaries
  ├── Activity: RenderPDFActivity(report_data, report_type, template_version)
  │             → pdf_bytes
  ├── Activity: StorePDFToS3Activity(pdf_bytes, user_id, assessment_year, version)
  │             → s3_key, report_record_id
  ├── Activity: StoreReportRecordActivity(report_record)
  └── emit audit_event: report.generated
```

**Retry:** `RenderPDFActivity` retries 2× (PDF rendering is deterministic; failure is usually transient OOM).
**Non-retryable:** `StorePDFToS3Activity` with a permission error — alert ops.
**Note:** Do not pass `pdf_bytes` through Temporal workflow history. `RenderPDFActivity` stores to a temp S3 path; subsequent activity reads from S3.

---

## W6 — `ComplianceEventWorkflow`

**Trigger:** Any significant compliance-relevant event (consent grant, withdrawal, data access, correction lock)
**Purpose:** Ensure audit_events are written durably even if the primary transaction fails

```
ComplianceEventWorkflow(event_type, user_id, resource_id, metadata)
  │
  └── Activity: WriteAuditEventActivity(event)
        └── Retry indefinitely with backoff until written
            (audit records must not be lost)
```

**Retry policy:** Unlimited retries with exponential backoff up to 24h. If not written after 24h, page ops.
**Rationale:** Audit events are compliance artifacts. Better to write late than lose them.

---

## W7 — `DataDeletionWorkflow`

**Trigger:** `DELETE /users/me` or admin `DELETE /admin/users/{id}/data`
**Purpose:** Orderly, auditable deletion of all user PII and documents

```
DataDeletionWorkflow(user_id, deletion_reason)
  │
  ├── Activity: RecordDeletionInitiatedAuditEvent(user_id)
  ├── Activity: DeleteS3DocumentsActivity(user_id)  [delete all S3 objects]
  ├── Activity: SoftDeleteDocumentRecordsActivity(user_id)  [set deleted_at]
  ├── Activity: NullifyPIIFieldsActivity(user_id)
  │             [set pan_encrypted, phone_encrypted = NULL; name = 'DELETED_USER']
  ├── Activity: RevokeAllSessionsActivity(user_id)
  ├── Activity: SoftDeleteUserActivity(user_id)  [set deleted_at]
  ├── Activity: RetainAuditEventsActivity(user_id)  [MUST NOT delete; just confirm]
  ├── Activity: NotifyUserDeletionCompleteActivity(user_id, email)  [send final email]
  └── emit audit_event: user.data_deleted
```

**Important:** `audit_events` rows are NEVER deleted (regulatory retention). Workflow explicitly confirms retention.
**Idempotency:** Deletion activities are idempotent (soft-delete is a no-op if already deleted; S3 delete of missing object succeeds).
**Retry:** Each activity retries 5× with backoff. If any non-PII step fails, continue; if PII nullification fails, halt and alert ops (data breach risk).

---

## W8 — `ExtractionRetryWorkflow`

**Trigger:** W2 failure after exhausting retries
**Purpose:** Escalate to human review queue; allow ops to manually re-trigger

```
ExtractionRetryWorkflow(extraction_job_id, failure_reason)
  │
  ├── Activity: UpdateExtractionJobStatus(job_id, 'FAILED')
  ├── Activity: CreateOpsAlertActivity(job_id, failure_reason)
  ├── Activity: NotifyUserActivity(user_id, 'extraction_failed_retry_later')
  └── emit audit_event: extraction.failed
```

**Manual re-trigger:** Ops can call admin API `POST /admin/extraction-jobs/{id}/retry` to re-run W2.

---

## Retry & Idempotency Summary

| Activity | Retryable | Max Retries | Backoff | Idempotency Key |
|----------|----------|------------|--------|----------------|
| VirusScanActivity | Yes | 3 | 10s exp | `document_version_id` |
| CallOCRVendorActivity | Yes | 3 | 30s exp | `extraction_job_id` |
| MapFieldsActivity | Yes | 3 | 5s linear | `extraction_job_id` |
| StoreExtractionResultActivity | Yes | 5 | 5s exp | `extraction_job_id` |
| RunTaxEngineActivity | No (validation) / Yes (transient) | 2 | 5s | `computation_run_id` |
| RenderPDFActivity | Yes | 2 | 10s | `computation_run_id` |
| WriteAuditEventActivity | Yes | Unlimited | Exp, max 1h | `event_id` |
| NullifyPIIFieldsActivity | Yes | 5 | 10s exp | `user_id` (idempotent null set) |
| DeleteS3DocumentsActivity | Yes | 5 | 15s exp | `user_id` (S3 delete idempotent) |

---

## Synchronous vs Asynchronous

| Operation | Mode | Reason |
|-----------|------|--------|
| Auth, user profile CRUD, deduction input | **Synchronous** | Fast DB operations; no queue needed |
| Pre-signed URL generation | **Synchronous** | Client needs URL immediately |
| Virus scan | **Async (BullMQ)** | External call; can be slow |
| OCR extraction | **Async (Temporal)** | Slow; must survive worker restarts |
| Tax computation | **Near-synchronous (Temporal)** | Fast but needs audit trail |
| Report generation | **Async (Temporal)** | Slow PDF render |
| Data deletion | **Async (Temporal)** | Multi-step; must be durable |
| Audit writes | **Async (Temporal, fire-and-forget signal)** | Must not fail the primary transaction |
| Email notifications | **Async (BullMQ)** | Non-blocking; retry on failure |

---

*Artifact status: WORKFLOWS COMPLETE.*
*Next: Prompt 8 (Document Ingestion & Extraction).*
