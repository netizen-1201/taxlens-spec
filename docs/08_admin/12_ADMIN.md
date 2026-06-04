# 12 — Admin and Ops Console

> **Role:** SaaS Operations Architect
> **Inputs:** All prior artifacts (01–11); STITCH_01 and STITCH_02
> **Access:** Internal team only; not exposed to end users
> **Base path:** `/admin` (separate auth session; admin-role JWT required; TOTP enforced)

---

## 1. Admin Roles

Defined fully in `10_COMPLIANCE.md` Section 4; summarized here for admin console context:

| Role | Console Access | Sensitive Capabilities |
|------|---------------|----------------------|
| `operator` | Read-only views; job queue management; extraction re-triggers; user lookup | Cannot delete user data; cannot modify tax rules; cannot approve maker-checker actions |
| `admin` | Full console access | Tax rule management; user data deletion; extraction unlock; maker-checker approvals; RBAC assignment |

**Admin session requirements:**
- Separate login at `/admin/login` (shares user DB but enforces `role IN ('admin', 'operator')`)
- TOTP 2FA required — no admin actions permitted without enrolled and verified 2FA
- Session timeout: 15 minutes inactivity (shorter than end-user 30 min)
- All admin sessions produce `admin.*` audit events (see `10_COMPLIANCE` event taxonomy)

---

## 2. Admin Screens

---

### A-01: Overview Dashboard `/admin`

**Purpose:** At-a-glance operational health for the current day and week.

**Components:**
- **System health banner:** Green/Yellow/Red indicator per service: API, Extraction Workers, Temporal, OCR Vendor, S3, Redis, Database, Email
- **Job queue summary cards:**
  - Extraction jobs today: `[N] completed / [N] needs_review / [N] failed`
  - Computation runs today: `[N] completed / [N] failed`
  - Reports generated today: `[N]`
- **Alerts panel:** Pinned items requiring action:
  - Failed extractions not yet re-triggered
  - Maker-checker actions pending approval (highlighted if > 24h old)
  - OCR vendor error rate spike
  - User deletion requests awaiting admin execution
- **Recent audit events strip:** Last 10 events (all users)
- **User signups (last 7 days):** sparkline

---

### A-02: Extraction Job Queue `/admin/extractions`

**Purpose:** Monitor all extraction jobs; review failed and needs-review queues; re-trigger jobs.

**Components:**
- **Filter bar:** Status (all, `needs_review`, `failed`, `processing`, `completed`), AY, document type, date range
- **Job table:**
  - Columns: Job ID (short), User (email, masked), Document Type, AY, Status, OCR Vendor, Created At, Last Updated, Actions
  - Actions: "View Extraction" | "Re-trigger" (operator) | "View User" (admin)
- **Batch actions:** Select multiple failed jobs → "Re-trigger Selected"
- **Job Detail drawer:** Clicking a row opens a side drawer:
  - Full extraction result (field values + confidence scores)
  - Raw OCR vendor response location (S3 link, admin-only)
  - Failure reason (if failed)
  - Temporal workflow ID (deep link to Temporal UI)
  - User-submitted corrections (if any)
  - Re-trigger button (logs `admin.extraction_rerun` event)

**States:**
- Empty (all clear): "No extraction jobs requiring attention."
- Failed spike: alert panel highlights count
- Re-trigger in progress: row shows "Re-triggering…" status

---

### A-03: User Management `/admin/users`

**Purpose:** Look up users for support; view their documents, extractions, computations.

**Components:**
- **Search bar:** Search by email (partial match)
- **User table:** Email (masked), Name, Role, AY, Last Login, Registration Date, Consent Status, Deletion Status
- **User detail page `/admin/users/[user_id]`:**
  - Profile summary (masked PAN last 4, role, consent status)
  - Compliance banner (D-09: "You are viewing sensitive financial data…")
  - Tabs: Documents | Extractions | Computations | Reports | Activity | Consent History
  - **Documents tab:** List of documents with version count and extraction status
  - **Extractions tab:** All extraction jobs and results for this user
  - **Computations tab:** All computation runs with full result JSON (expandable)
  - **Reports tab:** All generated PDFs
  - **Activity tab:** Full audit log for this user (admin sees raw values; operator sees masked view)
  - **Consent History tab:** All consent_records rows for this user
- **Admin actions on user (right sidebar):**
  - "Re-trigger extraction" (operator)
  - "Re-run computation" (operator + admin)
  - "Initiate data deletion" (admin only; triggers maker-checker flow)
  - "Change role" (admin only; limited to `user` ↔ `operator` via this UI; `admin` assignment done out-of-band)

**Note:** Every page view of a user's sensitive data (`/admin/users/[id]`) emits `admin.user_data_viewed` audit event with the operator/admin actor.

---

### A-04: Computation Management `/admin/computations`

**Purpose:** Monitor computation runs; trigger reruns for rule corrections.

**Components:**
- **Filter bar:** Status, AY, regime, rule_version, date range
- **Computation table:** User (masked), AY, Regime, Rule Version, Status, Created At, Actions
- **Rerun action:** "Rerun" button on any row → opens rerun modal:
  - Warning: "This creates a new computation run. The user's existing results are not changed."
  - Reason input (required)
  - Confirm → logs `admin.computation_rerun`
- **Rule version filter:** Show all computations that used a specific (outdated) rule version — useful after a rule amendment to understand impact

---

### A-05: Tax Rules Management `/admin/tax-rules`

**Purpose:** View, create, and activate versioned tax rule configurations.

**Components:**
- **Rule version table:** AY, Regime, Version, Is Active, Effective From, Created At, Notes
- **View rule detail:** Full `rules_json` rendered in a readable format (slabs table, surcharge table, deductions permitted/disallowed, rebate threshold)
- **"Create New Rule Version" button:**
  - Form: AY, regime, version tag, effective_from date, notes (required), rules JSON editor (with schema validation)
  - Submit → creates row with `is_active = false`, status = `pending_approval`
  - Emits `admin.approval_initiated` → appears in A-06 pending approvals queue
- **"Activate" button (on inactive rule):**
  - Also requires maker-checker (A-06)
  - On activation: sets `is_active = true` on new version; sets `is_active = false` on previous active version for same (AY, regime) pair

**Immutability rule:** Existing rule rows cannot be edited. Every change creates a new version row. The rules JSON editor is only available on new rule creation.

---

### A-06: Pending Approvals (Maker-Checker) `/admin/approvals`

**Purpose:** Queue for high-risk actions awaiting secondary admin approval.

**Components:**
- **Pending approvals table:**
  - Columns: Action Type, Initiated By, Initiated At, Subject (user/resource), Status, Expires At
  - Action types: `activate_tax_rule` | `delete_user_data` | `unlock_extraction` | `update_user_role`
- **Approve / Reject buttons:**
  - Approve: action executes immediately; `admin.approval_granted` logged
  - Reject: action cancelled; `admin.approval_rejected` logged; initiator notified
- **Self-approval prevention:** The admin who initiated an action cannot approve it. The "Approve" button is replaced with "Cannot approve own action" if the logged-in admin is the initiator.
- **Expiry:** Pending approvals expire after 48 hours. Expired actions are cancelled; initiator notified.

---

### A-07: Audit Log Inspection `/admin/audit`

**Purpose:** Search and inspect the full `audit_events` table for compliance and investigation.

**Components:**
- **Search / filter:**
  - User ID or email, action type (multi-select), resource type, date range, actor role, IP address
- **Event table:** All columns from `audit_events`; `before_value` / `after_value` shown as expandable JSON
- **PII view toggle:** Operators see masked view (safe PAN, masked phone). Admins can toggle "Show raw values" (emits `admin.user_data_viewed` event on each toggle).
- **Export CSV:** Date-range-limited CSV export of audit events (admin only; logged)

**Immutability guarantee:** No edit or delete controls exist anywhere on this screen. The only write operation is the auto-logged `admin.user_data_viewed` event that fires when this screen is accessed.

---

### A-08: System Health `/admin/health`

**Purpose:** Monitor external service availability and worker throughput.

**Components:**
- **Service status grid:**
  | Service | Status | Last Check | Latency (p95) | Notes |
  |---------|--------|-----------|--------------|-------|
  | OCR Vendor API | 🟢 UP | 30s ago | 2.1s | |
  | S3 (ap-south-1) | 🟢 UP | 30s ago | 45ms | |
  | Temporal | 🟢 UP | 30s ago | — | |
  | Redis | 🟢 UP | 30s ago | 3ms | |
  | PostgreSQL | 🟢 UP | 30s ago | 12ms | |
  | Email (SES) | 🟡 DEGRADED | 2m ago | — | Bounce rate elevated |
  | ClamAV | 🟢 UP | 30s ago | — | |

- **Worker throughput charts (last 24h):**
  - Extraction jobs completed per hour
  - Computation runs per hour
  - Reports generated per hour
  - Failed job rate (%)

- **OCR Vendor error log:** Last 20 vendor errors with error code, document type, timestamp

**Implementation:** Health endpoint data comes from `/api/v1/admin/health` which aggregates synthetic checks against each service.

---

### A-09: Security Monitoring `/admin/security`

**Purpose:** Detect abuse, suspicious login patterns, and policy violations.

**Components:**
- **Login failure summary:** Failed logins per IP per hour (threshold: 10+ failures → flagged)
- **Locked accounts:** Users in locked state due to too many failed attempts; "Unlock" button
- **Unusual activity flags:**
  - Users uploading > 50 documents in 24h (abuse signal)
  - Computations triggered > 20 times in 1 hour (could indicate API abuse)
  - Admin actions by inactive admin accounts
- **Active admin sessions:** Table of current admin sessions (IP, user agent, created at); "Revoke" button per row
- **Consent withdrawal spike:** If > 5% withdrawal rate in 24h, highlighted as an alert

---

## 3. High-Risk Actions

| Action | Protection Level | Mechanism |
|--------|----------------|-----------|
| Activate a new tax rule version | **Critical** | Maker-checker (A-06); different admin must approve; activation logged |
| Execute user data deletion | **Critical** | Maker-checker; reason required; 48h expiry; logged as `admin.user_deletion_executed` |
| Unlock a locked extraction | **Critical** | Maker-checker; reason required; logged as `admin.extraction_unlock` |
| Change a user's role | **High** | Maker-checker (operator → admin upgrades only by existing admin) |
| Export audit log CSV | **High** | Admin-only; date range limited to 90 days per export; logged |
| View raw PII in audit log | **High** | Admin-only toggle; fires `admin.user_data_viewed` per toggle |
| Trigger mass computation rerun (> 10 users) | **High** | Admin-only; requires written reason; logs list of affected user IDs |
| Create new tax rule draft | **Medium** | Admin-only; requires notes field; creates `pending_approval` record |
| Re-trigger individual extraction job | **Low** | Operator and admin; logged as `admin.extraction_rerun` |
| Re-trigger individual computation run | **Low** | Operator and admin; logged as `admin.computation_rerun` |

---

## 4. Immutable Logs vs Editable Metadata

### Immutable (no edit UI exists; enforced at DB and application layer)

| Resource | Reason |
|----------|--------|
| `audit_events` rows | Legal compliance artifact; append-only |
| `consent_records` rows | Proof of consent / withdrawal; legal record |
| `extraction_results` rows | Original extraction result must be preserved |
| `computation_runs` rows | Tax calculation history must be immutable for audit |
| `report_records` rows | Report generation record (PDF may expire but record stays) |
| `document_versions` rows | File upload record; immutable once created |
| `extraction_locks` rows | User decision to finalize; cannot be altered without maker-checker |

### Editable (with mandatory audit logging on every change)

| Resource | Editable Fields | Who Can Edit | Audit Event |
|----------|----------------|-------------|------------|
| `users` | `full_name`, `role` | Admin | `user.profile_updated` / `admin.user_role_changed` |
| `tax_profiles` | `regime_preference`, `residential_status` | User (own); Admin (any) | `tax_profile.updated` |
| `deduction_inputs` | All fields (via delete + re-create) | User (own); Admin (any) | `tax_profile.deduction_added` / `tax_profile.deduction_removed` |
| `extraction_jobs` | `status` (admin resets to `queued` for rerun only) | Operator/Admin | `admin.extraction_rerun` |
| `tax_rules` | `is_active` (activate/deactivate only; rules_json never edited) | Admin (maker-checker) | `admin.tax_rule_activated` |

---

## 5. Escalation Workflows

### Tier 1: Operator — Standard Support Issues

**Handles:**
- Extraction stuck in processing for > 30 minutes
- User reports incorrect extracted value (operator re-triggers extraction)
- User cannot download report (operator inspects job and re-triggers)
- Login issues unrelated to security

**Response SLA:** Same day

**Escalate to Tier 2 if:**
- Issue involves user's PAN or other PII that operator cannot view
- Issue requires data deletion or role change
- OCR vendor is returning systematic errors for a document type

---

### Tier 2: Admin — Compliance and Data Issues

**Handles:**
- User requests data deletion (DPDP Act right to erasure)
- Tax rule amendment required (post-Budget CBDT notification)
- User dispute about computation result accuracy
- Security incident: compromised account
- Consent-related issues: user claims they did not consent

**Response SLA:** 24 hours for deletion requests (DPDP Act compliance target); 48 hours for other compliance requests

**Escalate to Tier 3 if:**
- Regulator (IT Department / CERT-In) inquiry
- DPDP Data Protection Board complaint
- Systemic computation error affecting many users
- Potential data breach suspected

---

### Tier 3: Legal / CTO — Regulatory and Systemic Issues

**Handles:**
- CBDT notice or inquiry about platform outputs
- DPDP Data Protection Board proceedings
- Systemic tax computation error affecting > 100 users (mandatory recomputation batch)
- Security breach requiring user notification under DPDP Act

**Response:** Legal counsel engaged; incident response procedure activated; all actions documented with legal oversight

---

### Support Ticket to Deletion Request Flow

```
User submits deletion request (DELETE /users/me or support@taxlens.in email)
  ↓
Tier 1 operator creates deletion ticket; escalates to Admin
  ↓
Admin logs into /admin/users/[id]; clicks "Initiate Data Deletion"
  ↓
Pending approval created (A-06); different Admin must approve within 48h
  ↓
[Second Admin approves]
  ↓
DataDeletionWorkflow (W7) executes
  ↓
User receives confirmation email
  ↓
Audit trail: user.data_deleted event logged permanently
```

**SLA:** 7 days from request to completion (conservative target; DPDP Act does not specify exact SLA for erasure currently but best practice is ≤ 30 days).

---

*Artifact status: ADMIN CONSOLE COMPLETE.*
*Next: Stitching check (Prompts 10–12), then Prompt 13 (Delivery Backlog).*
