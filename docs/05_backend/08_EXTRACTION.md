# 08 — Document Ingestion & Extraction

> **Role:** Document Intelligence Architect for Fintech
> **Assumption:** OCR vendor to be selected from AWS Textract, Azure Form Recognizer, or Nanonets. Pipeline is vendor-agnostic at the interface level. All vendor-specific logic is isolated in provider adapters.

---

## Document Taxonomy (v1 Scope)

| Code | Document Name | Source | Format | Priority |
|------|--------------|--------|--------|---------|
| `form_16` | Form 16 (Part A + Part B) | Employer | PDF (digitally generated or scanned) | P0 — Must |
| `ais` | Annual Information Statement | IT Portal export | PDF or CSV | P0 — Must |
| `form_26as` | Form 26AS | IT Portal / TRACES | PDF or CSV | P0 — Must |
| `salary_slip` | Monthly Salary Slip | Employer payroll system | PDF (varies by payroll vendor) | P1 — Should |
| `interest_cert` | Bank Interest Certificate | Bank | PDF | P1 — Should |
| `other` | Generic supporting document | User | PDF / image | P2 — Could |

**Out of v1 scope (explicitly deferred):**
- Capital gains statement (broker-specific formats: Zerodha P&L, CAMS/CDSL consolidated)
- Rent receipts
- Donation receipts (80G)
- Home loan statements
- Insurance policy documents

---

## Upload Validation Rules

Applied at API layer (`POST /documents/upload-url`) before any storage:

| Rule | Value | Error Code |
|------|-------|-----------|
| Allowed MIME types | `application/pdf`, `image/jpeg`, `image/png`, `text/csv` | `UNSUPPORTED_MIME_TYPE` |
| Maximum file size | 20 MB (20,971,520 bytes) | `FILE_TOO_LARGE` |
| File name length | ≤ 255 characters | `INVALID_FILE_NAME` |
| Document type | Must be from `DocumentType` enum | `INVALID_DOCUMENT_TYPE` |
| Assessment year | Must be `YYYY-YY` format, e.g., `2025-26` | `INVALID_ASSESSMENT_YEAR` |
| Active consent | User must have a current consent record | `NO_ACTIVE_CONSENT` |
| Duplicate upload | Warn if same document_type + AY already has a locked extraction | `DUPLICATE_WARNING` (non-blocking) |

Applied at virus scan stage (post-upload):

| Rule | Action |
|------|--------|
| Virus/malware detected | Quarantine file; notify user; mark version `QUARANTINED`; do not extract |
| ClamAV signature timeout | Retry 3×; fail with ops alert if all retries exhaust |

Applied at extraction stage:

| Rule | Action |
|------|--------|
| PDF is password-protected | Mark job `FAILED`; return error `PASSWORD_PROTECTED_PDF`; prompt user to upload unlocked version |
| PDF has zero text layer (scanned image PDF) | Route to image-based OCR path (rasterize → OCR); note in extraction metadata |
| CSV does not match expected AIS/26AS column schema | Mark `NEEDS_REVIEW`; flag columns that don't match |
| File corrupt / unreadable | Mark `FAILED`; return error `FILE_UNREADABLE` |

---

## OCR / Extraction Pipeline Steps

```
Step 1: Pre-processing
  └── Detect PDF type: text-based vs scanned-image
      ├── Text-based PDF  → pdfminer / pypdf text extraction first
      └── Scanned PDF     → Rasterize pages (pdf2image → PIL images)
                            → Send images to OCR vendor

Step 2: OCR Vendor Call
  └── Send document (pages as images or PDF bytes) to vendor API
  └── Receive raw vendor response (bounding boxes, text, confidence)
  └── Store raw response to S3 (never in Temporal history)

Step 3: Document Classification (if document_type = 'other' or unverified)
  └── Classify document type from OCR text patterns
  └── Override extraction mapper selection if classification differs from user-declared type
  └── Notify user if mismatch detected

Step 4: Field Mapping (per document type)
  └── Apply DocumentTypeMapper (see field schemas below)
  └── Extract named fields using positional heuristics + regex
  └── Produce ExtractionResult with field values + confidence scores

Step 5: Normalization
  └── Normalize monetary values (strip ₹, commas, spaces → float)
  └── Normalize PAN to uppercase AAAAA9999A format
  └── Normalize TAN to AAAA99999A format
  └── Normalize dates to ISO 8601 (YYYY-MM-DD)
  └── Normalize assessment year to YYYY-YY string

Step 6: Confidence Scoring
  └── Per-field confidence = vendor confidence × heuristic match score
  └── Flag fields below 0.75 as low_confidence_fields[]
  └── Compute overall_confidence = min(per-field scores for Must-Have fields)

Step 7: Cross-Document Reconciliation Hints (informational only)
  └── If Form 16 and 26AS both present for same AY:
      → Flag TDS amount mismatches as reconciliation_warnings[]
  └── If AIS and Form 16 both present:
      → Flag income discrepancies as reconciliation_warnings[]
  └── Store as metadata on ExtractionResult; user sees these in review UI
```

---

## Extraction Fields by Document Type

### Form 16 — Part A

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `employer_name` | string | Yes | |
| `employer_tan` | string | Yes | Validate: AAAA99999A |
| `employer_pan` | string | No | May not appear on all Form 16s |
| `employee_name` | string | Yes | |
| `employee_pan` | string | Yes | Validate: AAAAA9999A |
| `assessment_year` | string | Yes | e.g., `2025-26` |
| `period_from` | date | Yes | Employment period start |
| `period_to` | date | Yes | Employment period end |
| `tds_q1` | float | No | TDS deposited Q1 |
| `tds_q2` | float | No | TDS deposited Q2 |
| `tds_q3` | float | No | TDS deposited Q3 |
| `tds_q4` | float | No | TDS deposited Q4 |
| `total_tds_deducted` | float | Yes | Sum of quarterly TDS |
| `challan_details` | array | No | BSR codes, deposit dates |

### Form 16 — Part B

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `gross_salary` | float | Yes | As per Section 17(1) |
| `value_of_perquisites` | float | No | Section 17(2) |
| `profits_in_lieu_of_salary` | float | No | Section 17(3) |
| `total_income_salary_head` | float | Yes | After exempt allowances |
| `hra_exemption` | float | No | If employer computed |
| `lta_exemption` | float | No | |
| `standard_deduction` | float | Yes | ₹50,000 (old) / ₹75,000 (new) |
| `entertainment_allowance` | float | No | For govt employees |
| `professional_tax` | float | No | |
| `net_salary_after_deductions` | float | Yes | |
| `section_80c_total` | float | No | If employer-reported |
| `section_80d` | float | No | If employer-reported |
| `total_deductions` | float | Yes | Aggregate |
| `taxable_income` | float | Yes | Net after all deductions |
| `tax_at_normal_rates` | float | Yes | Slab tax |
| `surcharge` | float | No | |
| `cess` | float | Yes | 4% health & education |
| `relief_u_s_89` | float | No | Arrear relief |
| `total_tax_payable` | float | Yes | |
| `less_tds` | float | Yes | TDS as per Part A |
| `balance_tax_payable` | float | Yes | May be 0 or refundable |

### AIS (Annual Information Statement)

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `pan` | string | Yes | |
| `assessment_year` | string | Yes | |
| `salary_income_entries` | array | Yes | [{source, amount, tds}] |
| `interest_income_savings` | float | No | 80TTA source |
| `interest_income_fd` | float | No | |
| `dividend_income` | float | No | |
| `tds_entries` | array | Yes | [{tan, deductor_name, amount, section}] |
| `advance_tax_paid` | float | No | |
| `self_assessment_tax_paid` | float | No | |
| `refund_issued` | float | No | |

### Form 26AS

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `pan` | string | Yes | |
| `assessment_year` | string | Yes | |
| `tds_entries_part_a` | array | Yes | Salary TDS: [{tan, name, amount, date}] |
| `tds_entries_part_b` | array | No | Non-salary TDS |
| `tcs_entries` | array | No | Tax collected at source |
| `advance_tax_part_c` | array | No | Self-paid advance tax |
| `tds_refunds_part_d` | array | No | |

### Salary Slip

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `month` | string | Yes | e.g., `2025-03` |
| `employer_name` | string | Yes | |
| `employee_name` | string | Yes | |
| `gross_salary` | float | Yes | |
| `basic_salary` | float | No | |
| `da` | float | No | Dearness allowance |
| `hra_component` | float | No | HRA received |
| `total_allowances` | float | No | |
| `pf_deduction` | float | No | Employee PF contribution |
| `professional_tax_deduction` | float | No | |
| `tds_deducted` | float | No | Monthly TDS |
| `net_pay` | float | Yes | |

### Interest Certificate

| Field | Type | Must-Have | Notes |
|-------|------|----------|-------|
| `bank_name` | string | Yes | |
| `account_type` | string | Yes | `savings`, `fd`, `rd` |
| `interest_amount` | float | Yes | |
| `tds_deducted` | float | No | TDS on interest (if any) |
| `period_from` | date | No | |
| `period_to` | date | No | |
| `pan_linked` | boolean | No | Affects TDS rate |

---

## Confidence Scoring Logic

```python
def compute_field_confidence(vendor_confidence: float, heuristic_match: bool) -> float:
    """
    vendor_confidence: 0.0–1.0 from OCR vendor
    heuristic_match: True if field value matches expected pattern (regex, range check)
    """
    base = vendor_confidence
    if heuristic_match:
        # Pattern matched; boost confidence slightly
        return min(1.0, base * 1.10)
    else:
        # Pattern failed; heavy penalty
        return base * 0.50

CONFIDENCE_THRESHOLD = 0.75  # Below this → NEEDS_REVIEW
```

**Must-Have field failure:** If any Must-Have field (see tables above) has confidence < 0.5, mark job `FAILED` (not just `NEEDS_REVIEW`) and prompt re-upload.

**Overall confidence:**
- `overall_confidence = min(confidence scores for all Must-Have fields)`
- Not an average — the weakest Must-Have field defines the run quality.

---

## Human Review Thresholds

| Condition | Action |
|-----------|--------|
| Any Must-Have field confidence < 0.50 | Job = `FAILED`; user prompted to re-upload |
| Any Must-Have field confidence 0.50–0.75 | Job = `NEEDS_REVIEW`; user prompted to correct |
| Any non-Must-Have field confidence < 0.75 | Field flagged in UI; user can ignore or correct |
| All Must-Have fields ≥ 0.75 | Job = `COMPLETED`; user can review or skip |
| TDS reconciliation warning present | Warning shown in review UI; not a blocker |
| Document type classification mismatch | Warning shown; user confirms or re-uploads |

---

## Downstream Normalized Output

After user locks extraction, the system produces a **NormalizedIncome** object consumed by the Tax Engine:

```json
{
  "user_id": "uuid",
  "assessment_year": "2025-26",
  "income_sources": [
    {
      "type": "salary",
      "source_document": "form_16",
      "document_version_id": "uuid",
      "gross_salary": 1250000,
      "total_income_salary_head": 1200000,
      "standard_deduction": 50000,
      "taxable_salary": 1150000
    }
  ],
  "other_income": [
    { "type": "interest_savings", "amount": 15000, "source_document": "ais" },
    { "type": "interest_fd", "amount": 25000, "source_document": "interest_cert" }
  ],
  "tds_credits": [
    {
      "deductor_tan": "DELA12345A",
      "deductor_name": "Acme Corp",
      "section": "192",
      "amount": 95000,
      "source_document": "form_26as"
    }
  ],
  "advance_tax_paid": 0,
  "self_assessment_tax_paid": 0,
  "extraction_result_ids": ["uuid1", "uuid2"],
  "locked_at": "2025-04-01T10:00:00Z"
}
```

This object is stored as the `input_snapshot` in `computation_runs`.

---

## Exact Extraction vs Inferred Classification

| Category | Description | Examples |
|----------|-------------|---------|
| **Exact extraction** | Field value copied verbatim from document | Gross salary figure, TAN, TDS amount |
| **Inferred/computed** | Derived from multiple fields or by the mapper logic | `taxable_salary = gross_salary - standard_deduction - hra_exemption` |
| **Classified** | Document type identified from content patterns | Detecting "FORM NO.16" string to confirm document is Form 16 |
| **User-confirmed** | User corrects or enters a value manually | Corrected TDS amount after mismatch |

The extraction result metadata must tag every field with its source category so the review UI can display: "Extracted from document" vs "Computed from extracted fields" vs "Entered by user."

---

*Artifact status: EXTRACTION DESIGN COMPLETE.*
*Next: Prompt 9 (Tax Rules Engine).*
