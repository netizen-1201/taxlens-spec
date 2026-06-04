# 03 — Product Requirements Document (PRD)

> **Role:** Senior Product Manager
> **Segment approved:** Salaried individuals, ITR-1 primary filers
> **Tax scope:** FY 2024-25 / AY 2025-26, old regime + new regime comparison
> **Filing posture:** Filing-support output only; NOT direct e-filing in v1

---

## 1. Product Overview

**Product name (working):** TaxLens (placeholder; rebrand as needed)

TaxLens is a web-based India income-tax SaaS for salaried individuals. Users upload their tax-relevant documents (Form 16, AIS, 26AS, salary slips, interest certificates), the platform extracts structured data, the user reviews and corrects the extraction, and the system computes a verified tax estimate under both old and new regimes. The output is a structured tax summary report suitable for self-review, CA handoff, or manual ITR-1 preparation.

TaxLens is not an e-Return Intermediary (ERI). It does not file returns on behalf of users. All computation outputs carry explicit disclaimers stating they are estimates, not legal filings.

---

## 2. Problem Statement

Salaried individuals in India face a fragmented, error-prone tax preparation workflow:

- **Form 16 confusion:** Most users cannot reconcile Part A (TDS) against Part B (income breakup) manually.
- **AIS/26AS mismatch:** The IT portal's pre-filled data frequently differs from employer-reported TDS, causing unexpected tax notices.
- **Regime decision paralysis:** The old vs new regime comparison requires simultaneous computation across multiple deduction categories; most users guess.
- **CA dependency for basic tasks:** Many users pay a CA ₹500–2,000 to do what a well-designed tool could automate for 80% of ITR-1 cases.
- **Unstructured document pile:** Users have their documents in WhatsApp, email, or physical form; there is no single place to organize and compute from them.

TaxLens solves steps 1–4 of the user's journey: organize → extract → reconcile → estimate. It hands off a clean summary to the user for the final step (filing), rather than replacing that step.

---

## 3. Target Users

### Primary User: Salaried Individual (Self-Filer)
- Age: 25–45
- Income: ₹5L–₹30L per annum
- Employment: Single employer, organized sector (private company or PSU)
- Tax knowledge: Basic to intermediate; knows they need Form 16, unsure about regime choice
- Goal: Understand tax position, avoid notices, save money on CA fees

### Secondary User: Salaried Individual (CA-Assisted Filer)
- Same profile but uses a CA for filing
- Goal: Provide accurate, organized input to their CA faster and with fewer back-and-forth calls

### Tertiary (Future): CA/Tax Preparer
- Not in v1 scope; deferred to post-MVP multi-tenant CA portal

---

## 4. Jobs to Be Done

| Job | Importance | Satisfaction (today) |
|-----|-----------|----------------------|
| "Help me extract all the right numbers from my Form 16" | Critical | Very low (manual, error-prone) |
| "Tell me if my TDS credit matches what I owe" | Critical | Very low (no tool does this easily) |
| "Show me whether I save under old or new regime" | High | Low (Excel or CA) |
| "Give me a document I can hand to my CA or use to file" | High | Medium (CA provides it) |
| "Store all my tax documents in one place" | Medium | Low (WhatsApp/email scattered) |
| "Alert me if something looks wrong in my documents" | Medium | Very low |

---

## 5. User Stories

### Document Upload
- As a user, I can upload my Form 16 (PDF) so the platform extracts my salary and TDS data.
- As a user, I can upload my AIS export (PDF or CSV) so the platform extracts my income and tax credits.
- As a user, I can upload my Form 26AS (PDF) so TDS from all deductors is captured.
- As a user, I can upload multiple salary slips so my monthly income picture is complete.
- As a user, I can upload interest certificates so savings and FD interest is included.
- As a user, I can re-upload a corrected version of a document and see a new extraction without losing the old one.

### Extraction Review
- As a user, I can see every field extracted from my documents side-by-side with the source document.
- As a user, I can correct any extracted value and add a note explaining the correction.
- As a user, I can flag a field as "needs review" if I am unsure.
- As a user, I can lock the extraction once I am satisfied, triggering a computation run.

### Tax Profile Inputs
- As a user, I can input my regime preference (old or new) or ask the platform to compare both.
- As a user, I can enter 80C investments (LIC, PPF, ELSS, EPF, home loan principal) up to ₹1.5L.
- As a user, I can enter 80D health insurance premiums.
- As a user, I can enter HRA eligibility details (rent paid, city type) if not fully captured in Form 16.
- As a user, I can enter 80TTA/80TTB interest income.
- As a user, I can flag other common deductions (NPS 80CCD, donations 80G) with manual values.

### Tax Estimate
- As a user, I can see my computed tax liability under both regimes in a single view.
- As a user, I can see a breakdown of: gross income, total deductions, taxable income, slab-wise tax, surcharge, health and education cess, total tax, TDS already paid, and balance payable or refundable.
- As a user, I can recompute if I change any input value.
- As a user, I see a clear "ESTIMATE — NOT A LEGAL FILING" label on every computation result.

### Report Generation
- As a user, I can download a structured tax summary PDF.
- As a user, the PDF includes: income summary, deduction details, TDS reconciliation table, regime comparison, and final tax estimate.
- As a user, I can see the version number and computation date on the report.

### Account and Consent
- As a user, I must explicitly consent to data processing before any document is stored or extracted.
- As a user, I can withdraw consent and request deletion of all my data.
- As a user, I can view an activity log of every action performed on my account and data.

---

## 6. Functional Requirements

### FR-1: Document Ingestion
- Accept PDF, JPEG, PNG, and CSV file uploads.
- Maximum file size: 20 MB per document.
- Validate MIME type and reject unsupported formats.
- Scan uploaded files for malware before processing (ClamAV or equivalent).
- Store original file in S3-compatible storage with server-side encryption.
- Generate a unique document ID, version number, and upload timestamp.
- Initiate extraction job asynchronously; return job ID to client.

### FR-2: Extraction Pipeline
- Extract structured fields from: Form 16 (Part A and Part B), AIS (all income and TDS sections), Form 26AS (TDS and TCS entries), salary slips (gross, basic, DA, HRA allowance, total deductions, net pay), and interest certificates (account type, interest amount, TDS deducted).
- Return per-field confidence scores.
- Flag fields below confidence threshold (configurable, default: 0.75) for human review.
- Store extracted data as versioned JSON records.
- Never overwrite original extraction; create new extraction version on re-run.

### FR-3: Extraction Review UI
- Display extracted fields alongside the original document (side-by-side or overlay).
- Allow user to edit any field value.
- Record edit history: original extracted value, corrected value, timestamp, user ID.
- Allow user to mark a field as "confirmed" or "needs review."
- Allow user to lock extraction (finalize for computation).

### FR-4: Tax Profile
- Accept manual deduction inputs for: 80C (itemized or aggregate), 80D, HRA (if applicable), 80TTA/80TTB, 80G, 80CCD(1B), LTA (self-declared).
- Support multiple deduction entries per category with source annotation.
- Validate total 80C claim does not exceed ₹1,50,000.
- Support selecting tax year (default: current AY).
- Support selecting computation regime (old, new, or compare both).

### FR-5: Tax Computation Engine
- Compute tax under old regime: apply all inputted deductions and exemptions, apply slab rates for the relevant AY, compute surcharge (if applicable), compute health and education cess (4%), apply Section 87A rebate (if applicable).
- Compute tax under new regime: apply only permitted deductions (standard deduction ₹75,000 for FY 2024-25, NPS employer contribution), apply new regime slabs, compute surcharge and cess, apply 87A rebate.
- Reconcile TDS credits (from Form 26AS / AIS) against computed tax liability.
- Produce: gross income, exemptions, deductions, taxable income, slab-wise tax breakup, surcharge, cess, 87A rebate, total tax, TDS credit, and net payable / refundable.
- Tag every computation run with: version, input source IDs, tax year, regime, timestamp.

### FR-6: Report Generation
- Generate a multi-section PDF report including: income summary, deduction detail, TDS reconciliation, regime comparison, and final tax estimate.
- Embed computation version, tax year, and "ESTIMATE — NOT A LEGAL FILING" disclaimer prominently.
- Store report as a versioned record linked to the computation run that produced it.
- Reports are immutable once generated; re-computation creates a new report version.

### FR-7: Audit Trail
- Record every action: upload, extraction job create/complete, field edit, lock, computation trigger, report download, login, logout, consent action.
- Audit records are append-only; no deletion except via explicit data deletion workflow triggered by user or legal hold policy.
- Each audit record includes: timestamp (UTC), user ID, action type, resource type, resource ID, IP address, and before/after values for edits.

### FR-8: Consent
- Present a clear consent screen before any document is uploaded or processed.
- Record: consent version, timestamp, user ID, IP address, user agent.
- Provide mechanism to withdraw consent; trigger data deletion workflow on withdrawal.
- Consent must be re-captured if consent terms are materially updated.

---

## 7. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Availability | 99.5% uptime (excluding planned maintenance) |
| Extraction job latency | P95 < 60 seconds for a single Form 16 PDF |
| Computation latency | < 3 seconds for a single tax computation run |
| PDF report generation | < 10 seconds |
| File upload size | Up to 20 MB per file |
| Concurrent users (v1) | Support 500 concurrent users without degradation |
| Data at rest | AES-256 encryption |
| Data in transit | TLS 1.2+ for all connections |
| PII fields | Field-level encryption for PAN, Aadhaar (if collected), bank account numbers |
| Session timeout | 30 minutes of inactivity |
| Password policy | Minimum 8 chars, 1 uppercase, 1 digit, 1 special char |
| Dependency monitoring | Health check endpoints for all external services |

---

## 8. Compliance and Audit Requirements

- **DPDP Act 2023 (India):** Consent-first data collection, data minimization, right to deletion, purpose limitation.
- **IT Act 2000 / Intermediary Rules:** Reasonable security practices for sensitive personal data (financial information).
- **AIS/26AS Data:** Not to be transmitted to third parties; used only for user's own computation.
- **ERI Non-Compliance Boundary:** Platform must not represent itself as an authorized ERI or claim to file on the user's behalf.
- **Audit Retention:** Audit logs retained for minimum 7 years (aligned with IT Act dispute window).
- **Report Disclaimer:** Every generated report must carry an explicit legal disclaimer that it is an estimate, not a tax filing, and that the user is responsible for verifying accuracy before use.
- **TDS Reconciliation Accuracy:** Any mismatch between extracted TDS and computed liability must be surfaced to the user, not silently resolved.

---

## 9. In-Scope vs Out-of-Scope

### In-Scope (v1)
- Salaried individuals, single employer, ITR-1 eligible
- Documents: Form 16 (Part A, Part B), AIS, Form 26AS, salary slips, bank interest certificates
- Tax: Old regime + new regime comparison, ITR-1 applicable heads of income only
- Output: Tax estimate, TDS reconciliation, downloadable PDF report
- Auth, consent, audit trail, versioning
- Internal admin console (job monitoring, support actions)

### Out-of-Scope (v1)
- Direct e-filing, ERI integration, ITR XML generation
- Capital gains (equity, MF, property)
- Business/professional income
- F&O / speculative income
- Foreign assets, DTAA
- Multi-employer scenarios (two Form 16s for the same year)
- CA multi-client portal
- DigiLocker / Account Aggregator pull
- GST, payroll tax, advance tax computation
- Notifications/reminders (advance tax deadlines, ITR due dates)
- Mobile apps (web-responsive only)

---

## 10. Success Metrics

| Metric | Target (90 days post-launch) |
|--------|-------------------------------|
| Registered users | 500 |
| Documents uploaded per user (avg) | ≥ 2 |
| Extraction accuracy (confirmed without correction) | ≥ 80% of Form 16 fields |
| Tax computation completed (% of users who uploaded ≥ 1 doc) | ≥ 60% |
| PDF report downloaded | ≥ 50% of users who completed computation |
| User-reported trust score ("I trust this estimate") | ≥ 4.0 / 5.0 (in-app survey) |
| Support tickets due to extraction error | < 10% of sessions |
| Consent withdrawal rate | < 2% |

---

## 11. Risks and Dependencies

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| OCR vendor accuracy on non-standard Form 16 PDFs | High | High | Manual review UX; confidence scoring; vendor benchmarking |
| AIS format changes by CBDT mid-season | Medium | High | Version parser; fallback CSV upload |
| Tax rule changes for AY 2025-26 post-budget | Low | High | Rule versioning in engine; CBDT notification tracking |
| Users uploading incorrect documents | High | Medium | Document type classifier; upload-time validation warnings |
| Computation disclaimer not understood as non-filing | Medium | High | Prominent UI language; legal review of disclaimer copy |
| DPDP Act compliance requirements evolving | Medium | Medium | Consent-first design from day one; modular consent layer |
| S3 storage cost growing faster than revenue | Low | Low | Lifecycle policies; document compression; retention limits |

---

*Artifact status: PRD COMPLETE — approved for architecture design.*
*Next: Run Prompt 4 (System Architecture).*
