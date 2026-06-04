# 06 — API Contracts (v1)

> **Role:** Senior API Designer
> **Stack:** NestJS REST API; async jobs via Temporal/BullMQ
> **Auth:** JWT Bearer token (access + refresh); all endpoints require auth unless marked `[public]`
> **Base URL:** `/api/v1`
> **Conventions:** snake_case JSON; `assessment_year` format: `"2025-26"`; all timestamps in ISO 8601 UTC

---

## Auth

### `POST /auth/register` `[public]`
**Purpose:** Create a new user account.

**Request:**
```json
{
  "email": "priya@example.com",
  "password": "Str0ng!Pass",
  "full_name": "Priya Sharma"
}
```

**Response 201:**
```json
{
  "user_id": "uuid",
  "email": "priya@example.com",
  "message": "Verification email sent."
}
```

**Validation:** Email unique; password ≥ 8 chars, 1 uppercase, 1 digit, 1 special. **Errors:** 409 email exists, 422 validation.

---

### `POST /auth/verify-email` `[public]`
**Purpose:** Verify email with OTP sent at registration.

**Request:** `{ "email": "...", "otp": "123456" }`
**Response 200:** `{ "message": "Email verified." }`
**Errors:** 400 invalid OTP, 410 OTP expired.

---

### `POST /auth/login` `[public]`
**Purpose:** Authenticate and receive JWT pair.

**Request:** `{ "email": "...", "password": "..." }`
**Response 200:**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "expires_in": 900
}
```

**Errors:** 401 invalid credentials, 403 email not verified, 423 account locked.

---

### `POST /auth/refresh` `[public]`
**Purpose:** Exchange refresh token for new access token.

**Request:** `{ "refresh_token": "eyJ..." }`
**Response 200:** `{ "access_token": "...", "expires_in": 900 }`
**Errors:** 401 invalid/expired refresh token.

---

### `POST /auth/logout`
**Purpose:** Revoke current session (invalidate refresh token).

**Request:** `{ "refresh_token": "eyJ..." }`
**Response 200:** `{ "message": "Logged out." }`

---

### `POST /auth/forgot-password` `[public]`
**Request:** `{ "email": "..." }`
**Response 200:** `{ "message": "Reset link sent if account exists." }` (always 200 to prevent enumeration)

---

### `POST /auth/reset-password` `[public]`
**Request:** `{ "token": "...", "new_password": "..." }`
**Response 200:** `{ "message": "Password updated." }`
**Errors:** 400 invalid token, 410 token expired.

---

## Users & Profile

### `GET /users/me`
**Purpose:** Fetch current user profile.

**Response 200:**
```json
{
  "user_id": "uuid",
  "email": "priya@example.com",
  "full_name": "Priya Sharma",
  "role": "user",
  "pan_last4": "1234",
  "is_email_verified": true,
  "created_at": "2025-01-01T00:00:00Z"
}
```
Note: `pan_encrypted` is never returned; only masked last 4.

---

### `PATCH /users/me`
**Purpose:** Update profile fields.

**Request:** `{ "full_name": "Priya S.", "pan": "ABCDE1234F" }` *(PAN encrypted server-side)*
**Response 200:** Updated user profile (same shape as GET /users/me)
**Validation:** PAN format: 5 alpha + 4 digit + 1 alpha. **Errors:** 422 invalid PAN.

---

### `DELETE /users/me`
**Purpose:** Consent withdrawal + account deletion. Triggers `DataDeletionWorkflow`.

**Request:** `{ "confirmation": "DELETE MY ACCOUNT" }`
**Response 202:** `{ "message": "Deletion scheduled. You will be notified on completion.", "deletion_job_id": "uuid" }`

---

## Consent

### `POST /consent`
**Purpose:** Record user consent acceptance.

**Request:** `{ "consent_version": "v1.2" }`
**Response 201:** `{ "consent_id": "uuid", "action": "granted", "created_at": "..." }`
**Note:** Must be called before any document upload is permitted.

---

### `GET /consent/status`
**Purpose:** Check if active consent is on record for this user.

**Response 200:** `{ "has_active_consent": true, "consent_version": "v1.2", "consented_at": "..." }`

---

## Documents

### `POST /documents/upload-url`
**Purpose:** Issue a pre-signed S3 URL for direct client upload.

**Request:**
```json
{
  "document_type": "form_16",
  "assessment_year": "2025-26",
  "file_name": "Form16_FY2425.pdf",
  "mime_type": "application/pdf",
  "file_size_bytes": 524288
}
```

**Response 200:**
```json
{
  "document_id": "uuid",
  "document_version_id": "uuid",
  "upload_url": "https://s3.amazonaws.com/...?X-Amz-Signature=...",
  "upload_url_expires_at": "2025-04-01T10:15:00Z",
  "fields": {}
}
```

**Validation:** `document_type` must be one of allowed types; `file_size_bytes` ≤ 20971520 (20 MB); `mime_type` in `[application/pdf, image/jpeg, image/png, text/csv]`. **Errors:** 400 unsupported type, 403 no active consent, 413 file too large.

---

### `POST /documents/{document_version_id}/confirm-upload`
**Purpose:** Notify backend that S3 upload is complete; triggers virus scan + extraction.

**Response 202:** `{ "extraction_job_id": "uuid", "status": "queued" }`
**Errors:** 404 version not found, 409 already confirmed.

---

### `GET /documents`
**Purpose:** List all documents for the current user.

**Query params:** `?assessment_year=2025-26&document_type=form_16`

**Response 200:**
```json
{
  "documents": [
    {
      "document_id": "uuid",
      "document_type": "form_16",
      "assessment_year": "2025-26",
      "display_name": "Form16_FY2425.pdf",
      "latest_version_number": 1,
      "latest_version_id": "uuid",
      "extraction_status": "completed",
      "created_at": "..."
    }
  ]
}
```

---

### `GET /documents/{document_id}`
**Purpose:** Fetch document metadata and all versions.

**Response 200:** Document record with `versions[]` array.

---

### `DELETE /documents/{document_id}`
**Purpose:** Soft-delete a document (user removes it from their account).

**Response 200:** `{ "message": "Document removed." }`
**Note:** Underlying S3 file retained for audit; logical delete only in DB.

---

## Extraction Jobs

### `GET /extractions/{extraction_job_id}`
**Purpose:** Poll extraction job status.

**Response 200:**
```json
{
  "extraction_job_id": "uuid",
  "document_version_id": "uuid",
  "status": "completed",
  "started_at": "...",
  "completed_at": "...",
  "result_id": "uuid"
}
```

**Statuses:** `queued`, `processing`, `completed`, `needs_review`, `failed`

---

### `GET /extractions/{extraction_job_id}/result`
**Purpose:** Fetch the structured extraction output with per-field confidence.

**Response 200:**
```json
{
  "result_id": "uuid",
  "document_type": "form_16",
  "assessment_year": "2025-26",
  "extracted_fields": {
    "employer_name": "Acme Corp Pvt Ltd",
    "employer_tan": "DELA12345A",
    "employee_pan": "ABCDE1234F",
    "gross_salary": 1200000,
    "total_tds_deducted": 95000
  },
  "confidence_scores": {
    "employer_name": 0.98,
    "employer_tan": 0.95,
    "gross_salary": 0.91,
    "total_tds_deducted": 0.88
  },
  "low_confidence_fields": [],
  "overall_confidence": 0.93,
  "is_locked": false,
  "corrections": []
}
```

---

### `PATCH /extractions/{result_id}/corrections`
**Purpose:** Submit one or more field corrections by the user.

**Request:**
```json
{
  "corrections": [
    {
      "field_name": "gross_salary",
      "corrected_value": "1250000",
      "note": "Part B shows different figure than Part A"
    }
  ]
}
```

**Response 200:** `{ "corrections_applied": 1 }`
**Errors:** 409 result is already locked.

---

### `POST /extractions/{result_id}/lock`
**Purpose:** User finalizes extraction; enables computation.

**Response 200:** `{ "locked_at": "...", "message": "Extraction locked. Ready for computation." }`
**Errors:** 409 already locked, 422 required fields still flagged as low-confidence.

---

## Tax Profile

### `POST /tax-profiles`
**Purpose:** Create or replace the user's tax profile for a given AY.

**Request:**
```json
{
  "assessment_year": "2025-26",
  "regime_preference": "compare",
  "residential_status": "resident"
}
```

**Response 201:** Tax profile object.

---

### `GET /tax-profiles/{assessment_year}`
**Purpose:** Fetch tax profile for a specific AY.

**Response 200:** Tax profile with associated `deduction_inputs[]`.

---

### `POST /tax-profiles/{assessment_year}/deductions`
**Purpose:** Add or update a deduction input.

**Request:**
```json
{
  "section_code": "80C",
  "sub_item": "PPF",
  "amount": 150000,
  "source": "user_input"
}
```

**Response 201:** Deduction input object.
**Validation:** `amount` > 0; `section_code` must be from permitted list. 80C aggregate validated against ₹1,50,000 cap on computation trigger (not at entry time).

---

### `DELETE /tax-profiles/{assessment_year}/deductions/{deduction_id}`
**Response 200:** `{ "message": "Deduction removed." }`

---

## Tax Computation

### `POST /computations`
**Purpose:** Trigger a tax estimate computation run.

**Request:**
```json
{
  "assessment_year": "2025-26",
  "regime": "compare"
}
```

**Response 202:**
```json
{
  "computation_run_ids": {
    "old": "uuid-old",
    "new": "uuid-new"
  },
  "status": "processing"
}
```

**Pre-conditions:** At least one extraction must be locked for the given AY. Active tax profile must exist. **Errors:** 400 no locked extractions, 400 missing tax profile.

---

### `GET /computations/{computation_run_id}`
**Purpose:** Fetch computation result.

**Response 200:**
```json
{
  "computation_run_id": "uuid",
  "assessment_year": "2025-26",
  "regime": "old",
  "status": "completed",
  "result": {
    "gross_income": 1250000,
    "standard_deduction": 50000,
    "total_deductions": 250000,
    "taxable_income": 1000000,
    "slab_breakdown": [
      { "slab": "0-250000", "rate": 0, "tax": 0 },
      { "slab": "250001-500000", "rate": 0.05, "tax": 12500 },
      { "slab": "500001-1000000", "rate": 0.20, "tax": 100000 }
    ],
    "tax_before_rebate": 112500,
    "rebate_87a": 0,
    "surcharge": 0,
    "cess": 4500,
    "total_tax": 117000,
    "tds_credit": 95000,
    "net_payable": 22000,
    "net_refundable": 0
  },
  "rule_version": "AY2025-26-v1",
  "disclaimer": "This is an estimate. Not a legal tax filing. Verify all figures before submission.",
  "created_at": "..."
}
```

---

### `GET /computations?assessment_year=2025-26`
**Purpose:** List computation history for a user.

**Response 200:** `{ "computation_runs": [...] }` (summary, no full result JSON)

---

## Report Generation

### `POST /reports`
**Purpose:** Trigger PDF report generation for a computation run.

**Request:** `{ "computation_run_id": "uuid" }` (or `["old-uuid", "new-uuid"]` for comparison report)
**Response 202:** `{ "report_job_id": "uuid", "status": "queued" }`

---

### `GET /reports/{report_id}/download-url`
**Purpose:** Get a time-limited pre-signed URL to download the generated PDF.

**Response 200:** `{ "download_url": "https://...", "expires_at": "...", "version": 1 }`
**Errors:** 404 report not generated yet, 202 if still processing.

---

### `GET /reports?assessment_year=2025-26`
**Purpose:** List all generated reports for the user.

**Response 200:** `{ "reports": [...] }` with version, type, computation_run_id, generated_at.

---

## Audit Logs

### `GET /audit/me`
**Purpose:** User-facing activity timeline.

**Query params:** `?from=2025-01-01&to=2025-12-31&limit=50&cursor=uuid`

**Response 200:**
```json
{
  "events": [
    {
      "id": "uuid",
      "action": "document.uploaded",
      "resource_type": "document",
      "resource_id": "uuid",
      "created_at": "...",
      "ip_address": "103.x.x.x"
    }
  ],
  "next_cursor": "uuid"
}
```

---

## Admin / Compliance Operations

*All routes under `/admin/*` require `role: admin` or `role: operator`.*

### `GET /admin/extraction-jobs?status=needs_review`
**Purpose:** List extraction jobs needing operator review.

### `GET /admin/users/{user_id}`
**Purpose:** View full user profile and document list.

### `POST /admin/computations/{computation_run_id}/rerun`
**Purpose:** Re-trigger a computation (e.g., after rule fix).

### `GET /admin/audit?user_id={uuid}&from=...&to=...`
**Purpose:** Full audit inspection with raw before/after values.

### `GET /admin/tax-rules`
**Purpose:** List all active tax rule versions.

### `POST /admin/tax-rules`
**Purpose:** Create a new tax rule version.
**Request:** `{ "assessment_year": "2025-26", "regime": "new", "version": "v2", "rules_json": {...}, "effective_from": "2025-02-01", "notes": "Budget amendment" }`

### `DELETE /admin/users/{user_id}/data`
**Purpose:** Execute data deletion for a consented withdrawal request.
**Requires:** `{ "confirmation": "CONFIRMED", "reason": "user_request" }`
**Response 202:** Deletion job ID.

---

*Artifact status: API CONTRACTS COMPLETE — v1 surface defined.*
*Next: Prompt 7 (Workflow Orchestration).*
