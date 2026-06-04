# 11 — Frontend Information Architecture

> **Role:** Senior Product Designer & Frontend Architect
> **Stack:** Next.js + TypeScript (SSR/SPA hybrid); Tailwind CSS; Zustand (UI state); React Query (server state)
> **Upload pattern:** Pre-signed S3 URL; client uploads directly; progress tracked via SSE/WebSocket
> **Mobile strategy:** Responsive web only (no native app); see mobile-first vs desktop-first callouts per screen

---

## Sitemap

```
/ (root)
├── /auth
│   ├── /login                        Public
│   ├── /register                     Public
│   ├── /verify-email                 Public
│   ├── /forgot-password              Public
│   └── /reset-password               Public
├── /onboarding                       Auth-required; shown once (no active consent yet)
├── /dashboard                        Auth + consent required
├── /documents
│   ├── /                             Document list (all AYs)
│   └── /[document_id]                Document detail + version history
├── /extraction
│   └── /[extraction_job_id]          Extraction review & correction
├── /tax-profile
│   └── /[assessment_year]            Profile setup + deduction entry (tabbed)
├── /estimate
│   └── /[assessment_year]            Computation result / regime comparison dashboard
├── /reports
│   ├── /                             Report list
│   └── /[report_id]                  Report detail + download
├── /activity                         Audit activity timeline
├── /settings
│   ├── /profile                      Name, email, PAN
│   ├── /security                     Password, 2FA
│   └── /data                         Export, deletion
└── /help                             Help center (static)
```

---

## Screen Specifications

---

### S-01: Login `/auth/login`

**Purpose:** Authenticate returning user.
**Mobile-first:** Yes.

**Components:**
- Logo header (centered)
- Email input
- Password input (show/hide toggle)
- "Forgot password?" link
- Submit button: "Sign In"
- Divider + "Don't have an account? Register" link
- Error alert (inline, dismissable)

**User actions:** Submit credentials; navigate to register; navigate to forgot-password.

**States:**
- Default: empty form
- Loading: button shows spinner; form disabled
- Error: inline error under failed field (e.g., "Invalid credentials") + toast
- Locked: "Too many attempts. Try again in 10 minutes." with disabled form
- Success: redirect to `/dashboard` (or `/onboarding` if no active consent)

---

### S-02: Register `/auth/register`

**Purpose:** Create new account.
**Mobile-first:** Yes.

**Components:**
- Full name input
- Email input
- Password input with strength indicator
- Confirm password input
- "Already have an account? Sign in" link
- Submit button: "Create Account"

**User actions:** Fill form and submit; navigate to login.

**States:**
- Default: empty form
- Validation: per-field inline errors on blur (email format, password requirements)
- Loading: button spinner
- Success: redirect to `/auth/verify-email` + brief toast "Check your email for a verification code"
- Error: "An account with this email already exists"

---

### S-03: Email Verification `/auth/verify-email`

**Purpose:** Confirm email ownership via OTP.
**Mobile-first:** Yes.

**Components:**
- "Enter the 6-digit code sent to [email]" headline
- 6-digit OTP input (auto-advance between digit boxes)
- "Resend code" link (enabled after 60s countdown)
- Submit button: "Verify Email"

**States:**
- Default: empty OTP input
- Loading: spinner
- Error: "Invalid code" / "Code expired — click Resend"
- Success: redirect to `/onboarding`

---

### S-04: Onboarding — Consent Screen `/onboarding`

**Purpose:** Capture explicit user consent before any document processing. Shown once; re-shown if platform T&C version changes.
**Mobile-first:** Yes.

**Components:**
- TaxLens logo + tagline
- Non-ERI disclaimer block (D-01 from Compliance controls)
- Consent statement (D-02)
- Links to Privacy Policy (opens in new tab) and Terms of Service (opens in new tab)
- "I understand TaxLens provides estimates, not official tax filings" checkbox
- "I agree to the Privacy Policy and Terms of Service" checkbox
- "Continue" button (disabled until both checkboxes checked)

**User actions:** Read disclosures; check both boxes; click Continue.

**States:**
- Default: both checkboxes unchecked; Continue disabled
- Active: both checked; Continue enabled
- Loading: spinner on Continue click (API call to `POST /consent`)
- Error: toast "Could not record consent. Please try again."
- Success: redirect to `/dashboard`

**Note:** Cannot be skipped. Navigating away without completing consent logs the user out and shows a "Complete account setup to continue" message on next login.

---

### S-05: Dashboard `/dashboard`

**Purpose:** Central hub showing current tax year status, quick actions, and document summary.
**Mobile-first:** Yes (cards stack vertically on mobile).

**Components:**
- Page header: "AY 2025-26 Tax Summary"
- **Status card** (primary CTA): shows current state of user's journey:
  - "Upload your documents to start" (no documents yet)
  - "X documents uploaded — review extraction" (documents uploaded, extraction pending review)
  - "Ready to compute — lock extraction to continue" (extraction complete, not locked)
  - "View your tax estimate" (computation done)
- **Document summary row:** Chip per document type (Form 16 ✓, AIS ✓, 26AS pending, etc.)
- **Quick action buttons:** "Upload Document" / "Review Extraction" / "Compute Tax"
- **Estimate summary card** (visible after computation): Regime comparison mini-view (Old: ₹X | New: ₹Y | Recommended: New)
- **Recent activity strip:** Last 3 audit events (e.g., "Form 16 extraction completed 2h ago")
- **AY selector dropdown:** Switch between AY 2025-26 and AY 2024-25 (if applicable)

**User actions:** Upload document (opens upload modal); navigate to extraction review; trigger computation; download latest report; view full activity log.

**States:**
- Loading: skeleton cards
- Empty (new user): full-page empty state with "Start with Form 16 →" CTA
- Documents uploaded, not yet extracted: progress indicator on status card
- Extraction in progress: "Extraction in progress…" with estimated wait time
- Review needed: yellow warning card "Review required before computing"
- Error: toast for any background job failure

---

### S-06: Document List `/documents`

**Purpose:** View all uploaded documents across all AYs; initiate new uploads.
**Mobile-friendly:** Yes (table collapses to cards on mobile).

**Components:**
- Page header + "Upload New Document" button
- **Filter bar:** AY selector, document type filter, status filter (`all`, `needs_review`, `completed`, `failed`)
- **Document table / card list:**
  - Columns: Document Type, AY, Filename, Upload Date, Extraction Status, Actions
  - Status badge (color-coded): Pending Scan / Extracting / Review Needed / Completed / Failed / Quarantined
  - Actions: "View" / "Review Extraction" / "Delete"
- Empty state: "No documents uploaded yet. Upload your Form 16 to start."

**User actions:** Upload document (modal); view document detail; review extraction; soft-delete document; filter/search list.

**Upload modal (overlay):**
- Document type selector (Form 16, AIS, Form 26AS, Salary Slip, Interest Certificate)
- AY selector (default: 2025-26)
- Drag-and-drop + browse file input (PDF, JPG, PNG, CSV; max 20 MB)
- File size indicator
- "Upload" button → triggers `POST /documents/upload-url` → direct S3 upload
- Upload progress bar (shows real file upload progress via XHR)
- Post-upload: "File uploaded. Extraction starting…" with dismiss

**States:**
- Loading: skeleton rows
- Empty: illustrated empty state
- Upload in progress: progress bar; other actions disabled
- Upload error: "Upload failed. Check file type and size." + retry button
- Quarantine: "This file was flagged during security scan. Please re-upload a clean copy."

---

### S-07: Document Detail `/documents/[document_id]`

**Purpose:** Show document metadata, all versions, linked extraction jobs.
**Desktop-first** (version history table is complex).

**Components:**
- Document header: type badge, AY, upload date
- **Version history table:** Version number, upload date, scan status, extraction status, actions
- **Latest extraction status card:** Link to extraction review
- "Upload New Version" button (triggers same upload modal with document_id prefilled)

**User actions:** View extraction for any version; upload a corrected version; soft-delete document.

---

### S-08: Extraction Review `/extraction/[extraction_job_id]`

**Purpose:** The most critical user interaction. Lets the user review extracted fields, see confidence indicators, and correct errors before locking for computation.
**Desktop-first** (side-by-side document view requires horizontal space). **Mobile:** single-column with toggle between document viewer and field list.

**Layout:**
```
┌─────────────────────────┬──────────────────────────────────┐
│  Document Viewer (PDF   │  Extracted Fields Panel          │
│  embedded / paginated)  │  - Field name                    │
│                         │  - Extracted value (editable)    │
│                         │  - Confidence bar                │
│                         │  - Source tag (Extracted /       │
│                         │    Computed / User-entered)      │
└─────────────────────────┴──────────────────────────────────┘
```

**Components:**
- Compliance banner (D-05): "Review each field carefully before locking."
- PDF viewer: rendered in-browser (PDF.js); page navigation; zoom
- **Extraction field list (right panel):**
  - Section headers per document section (Part A / Part B for Form 16)
  - Per-field row: label, value, confidence bar (green/yellow/red), "Edit" inline
  - Low-confidence fields: highlighted with yellow background + "⚠ Low confidence" chip
  - TDS reconciliation warnings: shown as orange banner if Form 16 vs 26AS mismatch detected
  - "Source" chip per field: `Extracted` / `Computed` / `User-entered`
- **Edit inline mode:** Click field → input appears in place; "Save" / "Cancel"; note input (optional)
- **Correction history drawer:** Per field, shows history of corrections with timestamps
- **Lock extraction CTA:** Bottom-right sticky bar: "Lock Extraction → Enable Computation" button
  - Disabled if any Must-Have field is unconfirmed at low confidence (< 0.75)
  - Tooltip explains what must be resolved

**User actions:** Edit any field value; add a note; mark field as "confirmed" manually; view correction history; lock extraction; go back to document list.

**States:**
- Loading: skeleton fields + PDF loading spinner
- Extraction in progress: "Still extracting... Check back in a moment" + auto-refresh
- Completed (all fields OK): fields displayed; Lock button enabled
- Needs review: yellow banner "X fields require your attention"; low-confidence fields highlighted; Lock button disabled until resolved
- Failed: error card "Extraction failed. Please re-upload your document."
- Locked: all fields read-only; "Extraction locked on [date]" banner; no edit controls visible
- Empty (no result yet): loading or error state

**Mobile adaptation:** Toggle button switches between "View Document" and "Review Fields" panes. Field editing works the same way.

---

### S-09: Tax Profile + Deductions `/tax-profile/[assessment_year]`

**Purpose:** Configure regime preference, residential status, and enter deduction amounts for the selected AY.
**Mobile-friendly:** Yes (form fields stack cleanly on mobile).

**Layout:** Two tabs — "Profile" and "Deductions".

#### Tab 1: Profile
**Components:**
- AY display (read-only heading)
- Regime selector: Radio group — "Old Regime" / "New Regime" / "Compare Both (Recommended)"
- Residential status selector: "Resident" / "NRI" (NRI shows alert: "NRI provisions are not supported in v1")
- Employment type: "Salaried" (only option in v1; shown as read-only chip)
- Save button

#### Tab 2: Deductions
**Components:**
- Disclaimer banner (D-06)
- Section accordion per deduction category:
  - **Section 80C** (cap: ₹1,50,000): Itemized entry rows (sub-item: PPF, LIC, ELSS, EPF, Home Loan Principal, etc.); running total vs cap progress bar
  - **Section 80D**: Self insurance (cap ₹25,000), Parent insurance (cap ₹25,000/50,000 for senior)
  - **HRA**: Rent paid per month, city type (metro/non-metro), employer HRA component (pre-filled if extracted)
  - **Section 80TTA / 80TTB**: Savings interest (pre-filled from AIS if extracted)
  - **Section 80G**: Charity donations with organization name + % eligible (50% or 100%)
  - **Section 80CCD(1B)**: NPS additional contribution (cap ₹50,000)
  - **Professional Tax**: Actual amount (pre-filled if extracted from Form 16)
- Running total summary at bottom: "Total deductions entered: ₹X"
- Save button (auto-saves on each section accordion close)

**User actions:** Select regime; enter each deduction; save; navigate to compute.

**States:**
- Empty: placeholders in each accordion with "Enter amount"
- Pre-filled from extraction: value shown with `Extracted` source chip; editable
- Validation error: red border on field exceeding cap (e.g., "80C cannot exceed ₹1,50,000")
- Saved: green checkmark on each section; "Saved" toast
- New Regime selected: 80C, 80D, 80G, 80TTA fields grayed out with tooltip "Not applicable under New Regime; still saved for Old Regime comparison if you change regimes later"

---

### S-10: Tax Estimate Dashboard `/estimate/[assessment_year]`

**Purpose:** Show computation results — old vs new regime comparison, full breakdowns, TDS reconciliation.
**Desktop-first** for full breakdown view. **Mobile:** collapsible sections, comparison table scrollable.

**Components:**
- **Estimate disclaimer banner** (D-03): Yellow warning, sticky or near top, cannot be collapsed
- **Regime comparison card** (top, prominent):
  ```
  Old Regime      New Regime      Recommended
  ₹1,17,000       ₹93,600         ✅ New Regime
                  Save ₹23,400
  ```
  - "Regime comparison note" (D-07) as collapsible tooltip
- **Detailed breakdown accordion** (per regime):
  - Gross Income
  - Standard Deduction
  - Other Income (interest, FD)
  - Total Deductions (itemized list)
  - Taxable Income
  - Slab-wise Tax Breakdown (table: slab | rate | tax)
  - Section 87A Rebate Applied / Not Applied
  - Surcharge (if applicable)
  - Health & Education Cess (4%)
  - **Total Tax Liability**
  - TDS Credit (from 26AS/AIS)
  - Net Payable / Refundable (colored: red = payable, green = refund)
- **TDS Reconciliation section:**
  - Table: Deductor | TAN | TDS Amount | Section
  - Any reconciliation warnings shown as orange callout boxes
- **Computation metadata footer:** "Computed on [date] | Rule version: AY2025-26-v1 | Based on [N] locked documents"
- **Action bar:** "Download Report" button | "Recompute" button (with warning: "This will run a fresh computation using your current inputs")

**User actions:** Switch between Old / New regime breakdown; expand/collapse sections; download report; recompute; navigate to deductions (edit inputs).

**States:**
- No computation yet: CTA card "Compute your tax estimate"
- Computation in progress: progress indicator + "Computing…"
- Completed: full breakdown visible
- Error: "Computation failed — [reason]. Please check your inputs and retry."
- Regime comparison unavailable (user chose one regime only): single breakdown shown

---

### S-11: Reports `/reports`

**Purpose:** List all generated report PDFs; generate new report; download.
**Mobile-friendly:** Yes.

**Components:**
- Page header + "Generate Report" button
- **Report list:**
  - Columns: Report Type, AY, Version, Generated At, Computation Run, Download
  - "Tax Summary — AY 2025-26 — v3 — Generated 14 Apr 2025 — [Download PDF]"
- Empty state: "No reports generated yet. Compute your tax estimate to generate a report."

**Generate Report modal:**
- AY selector
- Computation run selector (shows date + old/new regime)
- "Include regime comparison" toggle (if both regimes computed)
- "Generate PDF" button

**States:**
- Loading: skeleton rows
- Generation in progress: "Generating PDF…" with spinner; "Download" button disabled
- Download URL issued: "Download PDF" button active (pre-signed URL; 10-min expiry; new URL on each click)
- Error generating: "PDF generation failed. Retry." with retry button

---

### S-12: Activity Timeline `/activity`

**Purpose:** Show user their full audit activity log — uploads, extractions, computations, downloads.
**Mobile-friendly:** Yes (timeline cards).

**Components:**
- Page header: "Your Account Activity"
- **Timeline feed** (reverse chronological):
  - Event row: icon | action label | resource name | timestamp | IP address
  - E.g.: 📄 "Form 16 uploaded" — Form16_FY2425.pdf — 14 Apr 2025, 10:32 AM — 103.x.x.x
  - 🔒 "Extraction locked" — Form 16 — 14 Apr 2025, 11:00 AM
  - 📊 "Tax estimate computed" — AY 2025-26 (Both Regimes) — 14 Apr 2025, 11:05 AM
  - 📥 "Report downloaded" — v1 — 14 Apr 2025, 11:10 AM
- **Filters:** Date range picker; event type multi-select (uploads, extractions, computations, reports, auth)
- **Pagination / infinite scroll**

**States:**
- Loading: skeleton timeline
- Empty (new user): "No activity yet. Upload your first document to start."
- Load more: "Load earlier activity" button

---

### S-13: Settings — Profile `/settings/profile`

**Purpose:** View and update name, email, PAN.
**Mobile-friendly:** Yes.

**Components:**
- Full name input (editable)
- Email display (read-only; "Change email" deferred to v2)
- PAN input: shows masked value (ABCDE****F); "Update PAN" button; note: "PAN is encrypted and stored securely"
- Save button

---

### S-14: Settings — Security `/settings/security`

**Purpose:** Change password; manage 2FA.
**Mobile-friendly:** Yes.

**Components:**
- **Change password form:** Current password, new password, confirm new password
- **Two-Factor Authentication section:**
  - Status: "2FA not enabled" / "2FA enabled (TOTP)"
  - "Enable 2FA" button → opens QR code + manual key modal
  - "Disable 2FA" button (requires password confirmation)

---

### S-15: Settings — Data `/settings/data`

**Purpose:** Data export request and account deletion.
**Mobile-friendly:** Yes.

**Components:**
- **Data export section:** "Download a copy of your data" (deferred to v1.1; shown as "Coming Soon")
- **Account deletion section:**
  - Warning text (D-08 disclaimer)
  - "Delete My Account" button → opens confirmation modal
  - Confirmation modal: text input ("Type DELETE MY ACCOUNT to confirm") + confirm button

---

### S-16: Help Center `/help`

**Purpose:** FAQs, guidance on uploading documents, understanding the estimate.
**Mobile-friendly:** Yes (accordion FAQs).

**Components:**
- Search bar (static text search)
- FAQ accordion sections:
  - "What documents do I need?"
  - "What does the tax estimate include?"
  - "How do I correct an extraction error?"
  - "What is the difference between old and new regime?"
  - "Is TaxLens an official tax filing service?" (prominent, links to D-01)
  - "How is my data protected?"
  - "How do I delete my account?"
- "Contact Support" link (email or form — ops-handled in v1)

---

## Global Navigation

```
Top navigation bar (authenticated):
  [TaxLens logo] | Documents | Estimate | Reports | Activity | [User avatar ▾]
                                                              └── Profile Settings
                                                                  Security
                                                                  Data & Privacy
                                                                  Sign Out

AY selector (persistent, shown on document/estimate/report screens):
  [AY 2025-26 ▾]  ← always visible when AY-specific content is shown
```

**Mobile nav:** Hamburger menu collapses to icon; bottom tab bar for: Documents | Estimate | Reports | Settings.

---

## Navigation Flow

```
Register → Email Verify → Onboarding (Consent)
                              ↓
                         Dashboard
                         ├── Upload Document → Document List → Extraction Review
                         │                                          ↓
                         │                               Tax Profile / Deductions
                         │                                          ↓
                         └──────────────────────────── Estimate Dashboard → Reports
```

---

## State Summary

| Screen | Loading | Error | Empty | Review-Needed |
|--------|---------|-------|-------|---------------|
| Dashboard | Skeleton cards | Toast + error card | "Upload your first document" CTA | Yellow warning card for pending review |
| Document List | Skeleton rows | Toast | Illustrated empty state | Status badge on row |
| Extraction Review | Field skeleton | Error card + re-upload CTA | N/A | Yellow field highlight + Lock disabled |
| Tax Profile | Spinner | Toast | Pre-filled accordion open | N/A |
| Estimate Dashboard | Spinner | Error card + retry | "Compute first" CTA | Warning for reconciliation mismatch |
| Reports | Skeleton rows | Retry button | "Generate report" CTA | N/A |
| Activity | Skeleton timeline | Toast | "No activity yet" | N/A |

---

## Mobile-First vs Desktop-First

| Screen | Strategy | Rationale |
|--------|----------|-----------|
| Auth screens (S-01 to S-03) | Mobile-first | Users will register on phone |
| Onboarding / Consent (S-04) | Mobile-first | Critical; must work on all devices |
| Dashboard (S-05) | Mobile-first | Quick status check on mobile |
| Document List (S-06) | Mobile-first | Upload can happen from phone |
| Document Detail (S-07) | Desktop-first | Version history table; complex |
| Extraction Review (S-08) | Desktop-first | Side-by-side PDF + fields requires width |
| Tax Profile / Deductions (S-09) | Mobile-friendly | Stacked accordion works on mobile |
| Estimate Dashboard (S-10) | Desktop-first | Slab breakdown tables benefit from width |
| Reports (S-11) | Mobile-friendly | Simple list + download button |
| Activity Timeline (S-12) | Mobile-friendly | Card-based timeline |
| Settings (S-13 to S-15) | Mobile-friendly | Simple forms |
| Help Center (S-16) | Mobile-first | FAQ lookup on phone |

---

*Artifact status: FRONTEND IA COMPLETE.*
*Next: Prompt 12 (Admin and Ops Console).*
