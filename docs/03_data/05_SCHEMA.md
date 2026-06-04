# 05 — Database Schema

> **Role:** Staff Backend Architect
> **Database:** PostgreSQL (multi-tenant SaaS)
> **Inputs:** 03_PRD.md, 04_ARCHITECTURE.md, STITCH_01 (gaps G2, G4 addressed here)

---

## Schema Overview

The schema is organized into eight logical domains:

1. **Identity & Tenancy** — users, organizations, sessions
2. **Consent** — consent records
3. **Documents** — uploads, versions, storage references
4. **Extraction** — extraction jobs, results, corrections
5. **Tax Profile** — tax years, deduction inputs
6. **Computation** — computation runs, slab results, TDS reconciliation
7. **Reports** — generated report records
8. **Audit** — append-only event log

**Mutability principle:**
- `documents`, `extraction_results`, `computation_runs`, `reports`, `audit_events` are **immutable once created**. New versions are new rows, never updates.
- `users`, `organizations`, `tax_profiles`, `deduction_inputs` are **mutable** (normal UPDATE allowed, but changes are audit-logged).

---

## Table-by-Table Design

---

### `organizations`
**Purpose:** Top-level tenant container. In v1, each user is their own org unless CA multi-client is enabled later.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `name` | `text NOT NULL` | Org/user display name |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `updated_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `deleted_at` | `timestamptz` | Soft-delete |

**Indexes:** `deleted_at` (partial index for active orgs)

---

### `users`
**Purpose:** Individual user accounts; belongs to an org.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `org_id` | `uuid` FK → `organizations.id NOT NULL` | |
| `email` | `text UNIQUE NOT NULL` | |
| `password_hash` | `text NOT NULL` | bcrypt; never store plaintext |
| `full_name` | `text` | |
| `pan_encrypted` | `bytea` | AES-256-GCM, KMS-wrapped key; nullable until user provides |
| `phone_encrypted` | `bytea` | Encrypted mobile number |
| `role` | `text NOT NULL DEFAULT 'user'` | `user`, `admin`, `operator` |
| `totp_secret_encrypted` | `bytea` | For 2FA; nullable if not enrolled |
| `is_email_verified` | `boolean NOT NULL DEFAULT false` | |
| `last_login_at` | `timestamptz` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `updated_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `deleted_at` | `timestamptz` | Soft-delete on consent withdrawal |

**Indexes:** `email` (unique), `org_id`, `role`, `deleted_at` partial

**Security:** `pan_encrypted`, `phone_encrypted`, `totp_secret_encrypted` must never appear in application logs.

---

### `sessions`
**Purpose:** Track active refresh token sessions for revocation.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `refresh_token_hash` | `text NOT NULL` | SHA-256 of issued refresh token |
| `ip_address` | `inet` | |
| `user_agent` | `text` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `expires_at` | `timestamptz NOT NULL` | |
| `revoked_at` | `timestamptz` | Null = active |

**Indexes:** `user_id`, `refresh_token_hash` (unique), `expires_at` (for cleanup job)

---

### `consent_records`
**Purpose:** Immutable record of each consent event (capture or withdrawal).

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `consent_version` | `text NOT NULL` | e.g., `v1.2` — version of T&C/privacy policy |
| `action` | `text NOT NULL` | `granted`, `withdrawn` |
| `ip_address` | `inet` | |
| `user_agent` | `text` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `user_id`, `created_at`
**Immutable:** No UPDATE or DELETE on this table.

---

### `documents`
**Purpose:** Master record for each uploaded document (parent; one per logical upload).

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `org_id` | `uuid` FK → `organizations.id NOT NULL` | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `document_type` | `text NOT NULL` | `form_16`, `ais`, `form_26as`, `salary_slip`, `interest_cert`, `other` |
| `assessment_year` | `text NOT NULL` | e.g., `2025-26` |
| `display_name` | `text` | User-provided label |
| `latest_version_id` | `uuid` FK → `document_versions.id` | Denormalized pointer |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `deleted_at` | `timestamptz` | Soft-delete only |

**Indexes:** `user_id`, `document_type`, `assessment_year`

---

### `document_versions`
**Purpose:** Immutable record of each uploaded file version. A re-upload creates a new version row.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `document_id` | `uuid` FK → `documents.id NOT NULL` | |
| `version_number` | `integer NOT NULL` | Increments per document |
| `s3_bucket` | `text NOT NULL` | |
| `s3_key` | `text NOT NULL` | Full object key; server-side encrypted |
| `file_name_original` | `text NOT NULL` | Original filename from upload |
| `mime_type` | `text NOT NULL` | |
| `file_size_bytes` | `bigint NOT NULL` | |
| `checksum_sha256` | `text NOT NULL` | Integrity check |
| `virus_scan_status` | `text NOT NULL DEFAULT 'pending'` | `pending`, `passed`, `quarantined` |
| `virus_scan_at` | `timestamptz` | |
| `upload_ip` | `inet` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `document_id`, `version_number` (unique per document_id)
**Immutable:** No UPDATE. Use partial index on `virus_scan_status = 'passed'` for extraction queries.

---

### `extraction_jobs`
**Purpose:** Track the status of each extraction workflow run against a document version.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `document_version_id` | `uuid` FK → `document_versions.id NOT NULL` | |
| `temporal_workflow_id` | `text` | For Temporal lookup |
| `status` | `text NOT NULL DEFAULT 'queued'` | `queued`, `processing`, `completed`, `needs_review`, `failed` |
| `ocr_vendor` | `text` | e.g., `aws_textract` |
| `ocr_vendor_job_id` | `text` | Vendor's reference ID |
| `started_at` | `timestamptz` | |
| `completed_at` | `timestamptz` | |
| `failure_reason` | `text` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `document_version_id`, `status`, `created_at`

---

### `extraction_results`
**Purpose:** Immutable structured output of an extraction job. Each re-run creates a new row.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `extraction_job_id` | `uuid` FK → `extraction_jobs.id NOT NULL` | |
| `document_type` | `text NOT NULL` | Mirrors parent document type |
| `assessment_year` | `text NOT NULL` | |
| `extracted_fields` | `jsonb NOT NULL` | All extracted field values |
| `confidence_scores` | `jsonb NOT NULL` | Per-field confidence (0.0–1.0) |
| `low_confidence_fields` | `text[]` | Field names below threshold |
| `overall_confidence` | `numeric(5,4)` | Average or minimum |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `extraction_job_id`, `document_type`, `assessment_year`
**Immutable:** append-only.

---

### `extraction_corrections`
**Purpose:** Mutable record of user or operator corrections applied to an extraction result.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `extraction_result_id` | `uuid` FK → `extraction_results.id NOT NULL` | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | Who made the correction |
| `field_name` | `text NOT NULL` | |
| `original_value` | `text` | Value before correction |
| `corrected_value` | `text NOT NULL` | |
| `note` | `text` | Optional user note |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `extraction_result_id`, `user_id`

---

### `extraction_locks`
**Purpose:** Record when a user locks an extraction result as final for computation.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `extraction_result_id` | `uuid` FK → `extraction_results.id NOT NULL` | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `locked_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Immutable:** one row per locked extraction.

---

### `tax_profiles`
**Purpose:** User's tax year–specific configuration: deduction inputs, regime, HRA details.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `assessment_year` | `text NOT NULL` | |
| `regime_preference` | `text NOT NULL DEFAULT 'compare'` | `old`, `new`, `compare` |
| `residential_status` | `text NOT NULL DEFAULT 'resident'` | `resident`, `nri` |
| `employment_type` | `text NOT NULL DEFAULT 'salaried'` | `salaried` only in v1 |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `updated_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Unique constraint:** `(user_id, assessment_year)`

---

### `deduction_inputs`
**Purpose:** User-entered deduction amounts per category for a given tax profile.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `tax_profile_id` | `uuid` FK → `tax_profiles.id NOT NULL` | |
| `section_code` | `text NOT NULL` | e.g., `80C`, `80D`, `HRA`, `80TTA` |
| `sub_item` | `text` | e.g., `LIC`, `PPF`, `ELSS` |
| `amount` | `numeric(15,2) NOT NULL` | INR |
| `source` | `text` | `user_input`, `extracted` |
| `notes` | `text` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |
| `updated_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `tax_profile_id`, `section_code`

---

### `computation_runs`
**Purpose:** Immutable record of one tax estimate computation.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `tax_profile_id` | `uuid` FK → `tax_profiles.id NOT NULL` | |
| `assessment_year` | `text NOT NULL` | |
| `regime` | `text NOT NULL` | `old`, `new` |
| `input_snapshot` | `jsonb NOT NULL` | Full snapshot of all inputs at time of computation |
| `result` | `jsonb NOT NULL` | Full computation output (see Tax Engine for structure) |
| `rule_version` | `text NOT NULL` | e.g., `AY2025-26-v1` |
| `status` | `text NOT NULL DEFAULT 'completed'` | `completed`, `failed` |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `user_id`, `assessment_year`, `created_at`
**Immutable:** Never updated. New computation = new row.

---

### `tax_rules`
**Purpose:** Versioned tax rule configuration objects (slabs, surcharge thresholds, rebates) used by the computation engine.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `assessment_year` | `text NOT NULL` | |
| `regime` | `text NOT NULL` | `old`, `new` |
| `version` | `text NOT NULL` | e.g., `v1`, `v2` (amended mid-year) |
| `is_active` | `boolean NOT NULL DEFAULT true` | Only one active per (AY, regime) |
| `rules_json` | `jsonb NOT NULL` | Full rule configuration |
| `effective_from` | `date NOT NULL` | |
| `notes` | `text` | Changelog note |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Unique constraint:** `(assessment_year, regime, version)`

---

### `report_records`
**Purpose:** Immutable record of each generated report file.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` FK → `users.id NOT NULL` | |
| `computation_run_id` | `uuid` FK → `computation_runs.id NOT NULL` | |
| `report_type` | `text NOT NULL DEFAULT 'tax_summary'` | |
| `s3_bucket` | `text NOT NULL` | |
| `s3_key` | `text NOT NULL` | |
| `version_number` | `integer NOT NULL` | Per user, per AY |
| `generated_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `user_id`, `computation_run_id`
**Immutable.**

---

### `audit_events`
**Purpose:** Append-only log of every domain action.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PK | |
| `user_id` | `uuid` | Nullable (system events) |
| `org_id` | `uuid` | |
| `actor_role` | `text` | `user`, `admin`, `system`, `operator` |
| `action` | `text NOT NULL` | e.g., `document.uploaded`, `extraction.locked`, `computation.run` |
| `resource_type` | `text` | e.g., `document`, `computation_run` |
| `resource_id` | `uuid` | |
| `before_value` | `jsonb` | For edit events |
| `after_value` | `jsonb` | For edit events |
| `ip_address` | `inet` | |
| `user_agent` | `text` | |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` | |

**Indexes:** `user_id`, `resource_type, resource_id`, `action`, `created_at`
**Immutable.** No DELETE on this table. Partition by `created_at` (monthly) for archival.

---

## Relationships Summary

```
organizations
  └── users (many per org)
        └── consent_records (many per user)
        └── sessions (many per user)
        └── documents (many per user)
              └── document_versions (many per document)
                    └── extraction_jobs (many per version)
                          └── extraction_results (one per job)
                                └── extraction_corrections (many)
                                └── extraction_locks (one)
        └── tax_profiles (one per user per AY)
              └── deduction_inputs (many per profile)
        └── computation_runs (many per user)
              └── report_records (many per run)
audit_events (references any resource)
tax_rules (standalone; referenced by computation_runs.rule_version)
```

---

## Indexing Strategy

- All FK columns indexed by default.
- `audit_events.created_at` — range scans for time-window queries; partition by month.
- `document_versions.virus_scan_status` — partial index on `'passed'` for extraction queue.
- `extraction_results.document_type, assessment_year` — composite for report assembly.
- `computation_runs.user_id, assessment_year` — composite for history queries.
- `tax_rules.assessment_year, regime, is_active` — composite for engine lookup.

---

## Security-Sensitive Fields

| Table | Column | Sensitivity | Protection |
|-------|--------|-------------|-----------|
| `users` | `pan_encrypted` | PAN number — PII | AES-256-GCM, KMS-managed key |
| `users` | `phone_encrypted` | Mobile — PII | AES-256-GCM, KMS-managed key |
| `users` | `password_hash` | Auth secret | bcrypt; never log |
| `users` | `totp_secret_encrypted` | 2FA seed | AES-256-GCM, KMS-managed key |
| `document_versions` | `s3_key` | File path to financial document | Never expose in client responses |
| `sessions` | `refresh_token_hash` | Auth credential | SHA-256 hash only; never store raw |
| `extraction_results` | `extracted_fields` | Contains income, TDS, bank amounts | Column-level access control; masked in logs |
| `audit_events` | `before_value`, `after_value` | May contain PII deltas | Mask PAN/phone in log query views |

---

## Partitioning and Archival Strategy

- **`audit_events`:** Partition by `created_at` (monthly). Partitions older than 7 years are moved to cold storage (S3 Glacier) and dropped from active DB. Never hard-deleted (regulatory retention).
- **`document_versions`:** No partitioning in v1. S3 lifecycle policy moves original files to Glacier after 2 years if user has not re-accessed.
- **`computation_runs`:** No partitioning in v1. Archive rows older than 7 years to cold partition.
- **`report_records`:** S3 lifecycle policy deletes report PDFs after 3 years (regeneratable from computation_runs snapshot).

---

*Artifact status: SCHEMA COMPLETE.*
*Next: Prompt 6 (API Contracts).*
