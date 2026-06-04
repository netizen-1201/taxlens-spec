# 10 — Compliance Controls

> **Role:** Fintech Compliance Product Architect
> **Inputs:** 01_SCOPE through 09_TAX_ENGINE, STITCH_01, STITCH_02
> **Regulatory context:** DPDP Act 2023, IT Act 2000 / SPDI Rules 2011, Income Tax Act 1961
> **Non-ERI boundary:** Product is filing-support only. Not a CBDT-registered ERI. Must never represent itself as one.

---

## 1. Compliance-Sensitive Actions

Every action below must be logged to `audit_events`, validated against consent status, and subject to RBAC before execution.

| # | Action | Actor | Sensitivity | Requires Consent Gate |
|---|--------|-------|-------------|----------------------|
| CS-01 | User registration + PII collection (email, name) | user | Medium | No — consent is captured during registration |
| CS-02 | PAN submission / update | user | **High** — PII | Yes — consent must be active |
| CS-03 | Consent grant | user | High — legal record | N/A — this IS the consent event |
| CS-04 | Consent withdrawal + deletion trigger | user | **Critical** | N/A — user exercising legal right |
| CS-05 | Document upload (financial documents) | user | **High** — PII + financial | Yes |
| CS-06 | Document virus scan result (quarantine) | system | Medium | N/A — system action |
| CS-07 | Extraction result access (contains salary, TDS, income) | user | **High** | Yes |
| CS-08 | Extraction field correction (user modifies financial data) | user | **High** — before/after values | Yes |
| CS-09 | Extraction locking (finalizes data for computation) | user | High | Yes |
| CS-10 | Deduction input (user-entered financial data) | user | High | Yes |
| CS-11 | Tax computation trigger | user | High | Yes |
| CS-12 | Computation result access | user | High | Yes |
| CS-13 | Report generation request | user | High | Yes |
| CS-14 | Report PDF download | user | High — file contains PII summary | Yes |
| CS-15 | Admin: view any user's data or documents | operator/admin | **Critical** | Logged with business justification |
| CS-16 | Admin: trigger extraction re-run | operator | High | Yes (admin action on user data) |
| CS-17 | Admin: create or update tax rules | admin | **Critical** — affects all computations | Requires secondary admin review (maker-checker) |
| CS-18 | Admin: execute user data deletion | admin | **Critical** | Requires confirmation + audit |
| CS-19 | Admin: unlock a locked extraction | admin | **Critical** — reverses user decision | Requires secondary admin approval + reason |

---

## 2. Controls

### Per-Action Control Matrix

| Control Type | How Implemented |
|-------------|----------------|
| **Authentication gate** | All non-public endpoints require valid JWT access token (15-min expiry). |
| **Consent gate** | NestJS guard checks `consent_records` for active consent before allowing CS-05 through CS-14. Returns `403 NO_ACTIVE_CONSENT` if missing. |
| **RBAC guard** | NestJS `RolesGuard` enforces role-level access (see Section 4). |
| **Input validation** | `class-validator` DTOs on all API inputs; malformed requests rejected with 422 before DB touch. |
| **PII field protection** | `pan_encrypted`, `phone_encrypted`, `totp_secret_encrypted` are encrypted/decrypted only in the `UserService`. Never logged, never serialized in responses except last-4 masked versions. |
| **Rate limiting** | Auth endpoints: 10 req/min per IP. Upload URL: 20 req/hour per user. Computation: 10 req/hour per user. |
| **Audit write on every CS action** | `ComplianceEventWorkflow` (W6) fires asynchronously after every CS action. Failure of audit write does not fail the primary action — it retries indefinitely. |
| **Maker-checker for CS-17, CS-18, CS-19** | These actions require two admin role accounts: one initiates, one approves via `POST /admin/approvals/{action_id}/approve`. Pending approval is recorded as a new `admin_pending_actions` record. Action executes only on approval. |
| **Secure file access** | S3 keys for documents and reports are never returned in API responses. All file access uses time-limited pre-signed URLs (15 min for upload, 10 min for download). |
| **Extraction correction immutability** | `extraction_results` rows are append-only. User corrections are recorded in `extraction_corrections` (separate table) and applied at read time. Original extraction is never mutated. |
| **Computation immutability** | Every `POST /computations` creates a new `computation_runs` row. Previous runs are permanently accessible and never overwritten. |

### Consent Gate — Decision Logic

```
For every CS-05 to CS-14 action:
  1. Fetch latest consent_records row for user_id, ordered by created_at DESC
  2. If no row exists → 403 NO_ACTIVE_CONSENT
  3. If latest row action = 'withdrawn' → 403 CONSENT_WITHDRAWN
  4. If latest row action = 'granted' AND consent_version matches current platform version → PASS
  5. If consent_version is outdated (platform updated terms) → 403 CONSENT_REFRESH_REQUIRED
     → Frontend redirects user to re-consent screen
```

---

## 3. Audit Log Design

### Event Taxonomy

```
auth
  auth.registered          — new user account created
  auth.email_verified      — email OTP confirmed
  auth.login               — successful login
  auth.login_failed        — invalid credentials (includes IP)
  auth.logout              — explicit logout
  auth.password_reset      — password changed via reset flow
  auth.session_revoked     — refresh token explicitly revoked
  auth.account_locked      — too many failed attempts

consent
  consent.granted          — user accepted terms
  consent.withdrawn        — user withdrew consent
  consent.refresh_required — platform updated terms; user notified
  consent.refreshed        — user re-consented after version change

document
  document.upload_url_issued   — pre-signed URL given to client
  document.upload_confirmed    — client confirmed S3 upload complete
  document.quarantined         — virus scan failed
  document.scan_passed         — virus scan passed
  document.soft_deleted        — user removed document from account

extraction
  extraction.job_created       — extraction queued
  extraction.processing        — OCR vendor call started
  extraction.completed         — all fields above threshold
  extraction.needs_review      — one or more fields below threshold
  extraction.failed            — unrecoverable extraction error
  extraction.field_corrected   — user or admin corrected a field value
  extraction.locked            — user locked extraction for computation

computation
  computation.triggered        — user triggered a computation run
  computation.completed        — tax engine returned result
  computation.failed           — engine validation or rule error

report
  report.generation_requested  — user triggered PDF generation
  report.generated             — PDF stored in S3
  report.download_url_issued   — pre-signed download URL returned to user

tax_profile
  tax_profile.created          — first tax profile for AY
  tax_profile.deduction_added  — deduction input added
  tax_profile.deduction_removed — deduction input removed

user
  user.profile_updated         — name or other profile field changed
  user.pan_updated             — PAN submitted or updated (before: masked, after: masked)
  user.deletion_requested      — user triggered DELETE /users/me
  user.data_deleted            — W7 DataDeletionWorkflow completed

admin
  admin.user_data_viewed       — admin opened user profile or document list
  admin.extraction_rerun       — admin re-triggered extraction
  admin.computation_rerun      — admin re-triggered computation run
  admin.tax_rule_created       — new tax rule version added
  admin.tax_rule_activated     — tax rule version set as active
  admin.user_deletion_executed — admin executed data deletion
  admin.extraction_unlock      — admin unlocked a locked extraction
  admin.approval_initiated     — maker-checker: action initiated
  admin.approval_granted       — maker-checker: second admin approved
  admin.approval_rejected      — maker-checker: second admin rejected
```

### Audit Record Structure

```sql
-- Enforced by application layer; see 05_SCHEMA audit_events table
{
  id             UUID         -- event identifier
  user_id        UUID         -- subject user (nullable for system events)
  org_id         UUID         -- org context
  actor_id       UUID         -- who performed the action (may differ from user_id for admin ops)
  actor_role     text         -- 'user', 'admin', 'operator', 'system'
  action         text         -- from taxonomy above
  resource_type  text         -- 'document', 'extraction_result', 'computation_run', etc.
  resource_id    UUID         -- ID of affected resource
  before_value   jsonb        -- PII masked (see masking rules)
  after_value    jsonb        -- PII masked
  ip_address     inet
  user_agent     text
  created_at     timestamptz  -- UTC; append-only
}
```

### PII Masking Rules for Audit Log Views

Read views (`v_audit_events_safe`) apply these masks. Raw table is accessible only to `admin` role:

| Field | Mask Rule |
|-------|-----------|
| PAN in `before_value`/`after_value` | Replace with `"pan": "*****NNNNA"` (last 5 visible) |
| Phone in any value column | Replace with `"phone": "XXXXXXX####"` (last 4 visible) |
| `password_hash` anywhere | Replace with `"password_hash": "[REDACTED]"` |
| `refresh_token_hash` anywhere | Replace with `"[REDACTED]"` |
| `extracted_fields.employee_pan` | Masked same as PAN rule |
| `extracted_fields` income/TDS amounts | Not masked — financial values are the audit substance |

### Audit Write Guarantee

`ComplianceEventWorkflow` (W6) uses Temporal with **unlimited retries** and exponential backoff up to 24 hours. If still not written after 24 hours, an ops page alert is triggered. Audit records must not be silently dropped. This is a compliance-critical guarantee.

---

## 4. Role-Based Access Control (RBAC)

### Roles

| Role | Description | Assignment |
|------|-------------|-----------|
| `user` | Standard registered user; can only access their own data | Default on registration |
| `operator` | Internal ops team; read-only access to user data for support; can re-trigger jobs | Manually assigned by admin |
| `admin` | Full access including tax rule management, data deletion, maker-checker approvals | Manually assigned; MFA required |

### Permission Matrix

| Resource / Action | `user` | `operator` | `admin` |
|------------------|--------|------------|---------|
| Own profile (read/update) | ✅ | ✅ (read only) | ✅ |
| Other user's profile | ❌ | ✅ read (logged) | ✅ |
| Own documents (upload/list/delete) | ✅ | ❌ | ✅ |
| Other user's documents | ❌ | ✅ read (logged) | ✅ |
| Own extraction results | ✅ | ❌ | ✅ |
| Other user's extractions | ❌ | ✅ read (logged) | ✅ |
| Correct own extraction | ✅ | ❌ | ✅ with maker-checker |
| Lock own extraction | ✅ | ❌ | ✅ |
| Unlock any extraction | ❌ | ❌ | ✅ with maker-checker |
| Own tax profile / deductions | ✅ | ❌ | ✅ |
| Trigger own computation | ✅ | ❌ | ✅ |
| View any computation | ❌ | ✅ read (logged) | ✅ |
| Re-trigger any computation | ❌ | ✅ (logged) | ✅ |
| Own reports (generate/download) | ✅ | ❌ | ✅ |
| View any report | ❌ | ✅ read (logged) | ✅ |
| Own audit log (limited) | ✅ | ❌ | ✅ |
| Full audit log (all users) | ❌ | ✅ read (logged) | ✅ |
| Create / update tax rules | ❌ | ❌ | ✅ with maker-checker |
| Activate tax rule version | ❌ | ❌ | ✅ with maker-checker |
| Initiate user data deletion | ❌ | ❌ | ✅ with maker-checker |
| Admin panel access | ❌ | ✅ (limited screens) | ✅ (all screens) |

### Admin MFA Requirement

All `admin` role users must have TOTP 2FA enrolled before any admin action is accepted. Unenrolled admins are blocked with `403 MFA_REQUIRED`.

---

## 5. Document Retention and Deletion

### Retention Schedule

| Data Type | Primary Storage | Retention Period | Post-Period Action |
|-----------|----------------|-----------------|-------------------|
| Uploaded document files (S3) | `documents-bucket/` | Until user deletes or consent withdrawal | Hard-delete on W7 execution |
| `document_versions` rows | PostgreSQL | 7 years | Soft-delete (deleted_at); archive AY > 7 years to cold partition |
| `extraction_results` rows | PostgreSQL | 7 years | Archive; PII fields in `extracted_fields` JSONB nullified on user deletion |
| `computation_runs` rows | PostgreSQL | 7 years | Archive; `input_snapshot` PII nullified on user deletion |
| Report PDFs (S3) | `reports-bucket/` | 3 years (regeneratable) | S3 lifecycle → auto-expire; on user deletion, immediate delete |
| `audit_events` rows | PostgreSQL | **Minimum 7 years; never hard-deleted** | Monthly partitions → S3 Glacier after 7 years; never dropped |
| `consent_records` | PostgreSQL | **Permanently** — legal record | Never deleted even on user request |
| `sessions` rows | PostgreSQL | Until `expires_at` or explicit revocation | Background cleanup job every 6 hours |
| `tax_rules` rows | PostgreSQL | Permanently — historical rule correctness | Never deleted |

### Data Deletion Workflow (W7) — Compliance Steps

When `DELETE /users/me` or admin deletion is executed:

```
Step 1: Record deletion initiation in audit_events (consent.withdrawn or user.deletion_requested)
Step 2: Hard-delete all S3 objects for this user (all document files, all report PDFs)
Step 3: Set deleted_at on all document rows and document_version rows
Step 4: Nullify PII in extracted_fields JSONB: set employee_pan, employer_tan → NULL in all extraction_results rows
Step 5: Nullify input_snapshot PII in computation_runs (PAN, phone references)
Step 6: Set users.pan_encrypted = NULL, phone_encrypted = NULL, full_name = 'DELETED_USER'
Step 7: Revoke all active sessions
Step 8: Set users.deleted_at = NOW()
Step 9: Confirm audit_events retention (DO NOT delete; just log W7 completion)
Step 10: Send final confirmation email to the address on file
Step 11: Emit audit_event: user.data_deleted
```

**Important:** `consent_records` and `audit_events` rows are explicitly excluded from deletion. This is a legal requirement: consent records prove lawful basis for processing and must survive user deletion.

### User Data Export (DPDP Act — Right of Access)

v1 scope: Not implemented. Add to v1.1 backlog.

Design note: When implemented, export should include:
- All `extraction_results.extracted_fields` (in human-readable JSON)
- All `computation_runs.result` summaries
- `consent_records` for the user
- Limited `audit_events` (user-facing subset, PII-safe)

---

## 6. Human Review Checkpoints

| Checkpoint | Trigger | Reviewer | Blocking? | Action if Not Reviewed |
|-----------|---------|---------|----------|----------------------|
| HR-01 | Extraction confidence < 0.75 on any Must-Have field | User | **Yes** — cannot lock until corrected or confirmed | Job stays `NEEDS_REVIEW`; computation blocked |
| HR-02 | Must-Have field confidence < 0.50 | User | **Yes** — must re-upload | Job stays `FAILED`; user prompted to re-upload |
| HR-03 | TDS reconciliation warning (Form 16 vs 26AS mismatch) | User | No — informational | Warning persists on estimate; user can proceed |
| HR-04 | Document type classification mismatch | User | No — informational | User can confirm or re-upload |
| HR-05 | Extraction job `NEEDS_REVIEW` + not actioned in 72h | Ops (W3 timer) | No | Reminder sent; job remains in review queue |
| HR-06 | Extraction job `FAILED` + ops alert triggered | Ops | No | Appears in admin extraction queue for investigation |
| HR-07 | Admin tax rule update (CS-17) | Second admin (maker-checker) | **Yes** — rule not activated until approved | Rule stays in `pending_approval` status |
| HR-08 | Admin user data deletion (CS-18) | Second admin (maker-checker) | **Yes** — deletion not executed until approved | Deletion stays in `pending_approval` status |
| HR-09 | Admin extraction unlock (CS-19) | Second admin (maker-checker) | **Yes** | Unlock stays in `pending_approval` |

---

## 7. Required Disclaimers

### D-01: Onboarding / Registration Screen
> "TaxLens helps you organize your tax documents and compute an income tax estimate. **TaxLens is not a CBDT-registered e-Return Intermediary (ERI) and does not file income tax returns on your behalf.** All outputs are estimates only. Consult a Chartered Accountant before filing your return."

**Placement:** Shown prominently before registration form. Must be visible without scrolling.
**Acceptance:** User must check "I understand TaxLens provides estimates, not official tax filings" to proceed.

### D-02: Consent Screen
> "By continuing, you consent to TaxLens processing the financial documents you upload for the sole purpose of generating an income tax estimate for your personal use. Your data is encrypted and stored on servers in India. You can withdraw consent and delete your data at any time from Account Settings. For details, read our Privacy Policy [link] and Terms of Service [link]."

**Placement:** Full-screen modal before first upload. Cannot be dismissed without explicit "I Agree" button press.
**Logged:** `consent.granted` event with consent_version, IP, timestamp.

### D-03: Computation Result Screen
> ⚠️ **ESTIMATE ONLY — NOT A LEGAL TAX FILING.**
> "This computation is based on data extracted from your uploaded documents. It may be inaccurate if documents were incomplete, contained errors, or if you have income sources not entered here. TaxLens is not liable for tax notices or penalties arising from reliance on this estimate. Verify all figures with original documents before filing your ITR."

**Placement:** Yellow banner at top of computation result; must be visible above the fold.
**Cannot be collapsed or dismissed.**

### D-04: Generated PDF Report — Every Page
> Footer text: `ESTIMATE ONLY | TaxLens | AY 2025-26 | Generated: [DATE] | Version [N] | This is NOT an official tax filing. Verify all figures before ITR submission.`

**Implementation:** Embedded in PDF template; cannot be removed by user.

### D-05: Extraction Review Screen
> "The values below were automatically extracted from your document. **Please review each field carefully.** Incorrect values will directly affect your tax estimate. You can correct any field before locking."

**Placement:** Info banner at top of extraction review screen.

### D-06: Deduction Input Form
> "Enter only actual eligible investments and expenses you have made during FY 2024-25. TaxLens applies statutory caps (e.g., 80C limit of ₹1,50,000) automatically but **does not verify the eligibility of individual investments.** You are responsible for maintaining documentation for all claimed deductions."

**Placement:** Help text block above the deduction form.

### D-07: Regime Comparison Screen
> "This comparison uses tax rules for AY 2025-26 as of the last update ([DATE]). Your actual liability may differ due to income sources not entered, employer-verified details, or future CBDT amendments. For personalized tax advice, consult a Chartered Accountant."

**Placement:** Below the old vs new regime comparison table.

### D-08: Data Deletion Confirmation Modal
> "Deleting your account will permanently remove all your uploaded documents and personal data from TaxLens. **Regulatory audit logs will be retained for 7 years** as required by applicable law, but will not contain personally identifiable information after deletion. This action cannot be undone."

**Placement:** Confirmation modal before triggering W7. User must type "DELETE MY ACCOUNT" to confirm.

### D-09: Admin Panel — User Data Access
> Shown to operator/admin on any screen displaying user data:
> "You are viewing sensitive financial data belonging to a TaxLens user. This access is logged. Access this data only as required for legitimate support or compliance purposes."

**Placement:** Sticky banner on all admin screens showing user-specific data.

---

## 8. Compliance Architecture Notes

### DPDP Act 2023 Alignment

| Obligation | Implementation |
|-----------|---------------|
| Consent before processing | `ConsentModule` gate on all CS-05 to CS-14 actions |
| Purpose limitation | Documents used only for tax estimate; no third-party sharing |
| Data minimization | Only document types needed for ITR-1 are accepted in v1 |
| Right to erasure | `DataDeletionWorkflow` (W7) with 7-day SLA |
| Right of access / portability | Deferred to v1.1 (data export feature) |
| Grievance officer | Required for DPDP compliance; designate on launch; surface contact in Privacy Policy |
| Cross-border transfer | No — all data in ap-south-1; S3 bucket must have `bucket-region` policy blocking replication outside India |

### IT Act / SPDI Rules Alignment

| Obligation | Implementation |
|-----------|---------------|
| Reasonable security practices for financial data | AES-256 at rest, TLS 1.2+ in transit, field-level encryption for PAN/phone |
| Disclosure of data practices | Privacy Policy published; link on every consent screen |
| No unlawful disclosure to third parties | OCR vendor receives document bytes only (no PII identifiers); contract terms required |

### Non-ERI Boundary — Hard Rules

1. The platform MUST NOT accept payment or represent itself as offering "ITR filing" as a service.
2. API responses, PDF reports, and all UI text must consistently use "estimate" and "filing-support" — never "filing" or "e-filing."
3. Any ITR XML or JSON export feature (deferred to v2) must include a prominent disclaimer that submission remains the user's responsibility.
4. The platform cannot pre-populate ITR forms with an "auto-submit" button.
5. Periodic legal review of all user-facing text for ERI boundary compliance — recommended every 6 months or before any new tax season.

---

*Artifact status: COMPLIANCE CONTROLS COMPLETE.*
*Next: Prompt 11 (Frontend Information Architecture).*
