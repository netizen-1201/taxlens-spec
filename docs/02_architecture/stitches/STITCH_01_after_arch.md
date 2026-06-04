# Stitching Report — After Artifacts 01–04

> **Role:** Systems Editor
> **Scope:** 01_SCOPE.md, 02_MVP.md, 03_PRD.md, 04_ARCHITECTURE.md
> **Purpose:** Contradictions, missing dependencies, terminology normalization, required updates

---

## Contradictions Identified

| # | Location | Issue | Resolution |
|---|----------|-------|-----------|
| C1 | 01_SCOPE (A3) vs 04_ARCH (Document Ingestion) | Scope assumption says "no DigiLocker pull in v1" — Architecture confirms manual upload only. ✅ Consistent. | No action needed. |
| C2 | 02_MVP (Should Have: mobile-responsive) vs 04_ARCH (FE: SSR Next.js) | MVP says "mobile-responsive" but architecture says "responsive web" not native app. ✅ Consistent. | Confirm in 11_FRONTEND that this means responsive web, not PWA or native. |
| C3 | 03_PRD (FR-2: confidence threshold 0.75 default) vs 04_ARCH (extraction worker: confidence < 0.75 → NEEDS_REVIEW) | Same threshold value. ✅ Consistent. | None. |
| C4 | 03_PRD (FR-1: file size 20 MB) vs 04_ARCH (pre-signed URL) | Max file size not mentioned in architecture upload flow. ⚠️ Gap. | Add 20 MB limit to API contract and S3 upload policy in 06_API.md. |
| C5 | 02_MVP (Should Have: email notification) vs 04_ARCH (Email provider listed as external dependency) | Consistent but email notification is marked "non-blocking" in architecture. ✅ Acceptable. | Document explicitly in workflows that notification failure does not fail extraction workflow. |
| C6 | 03_PRD (Out-of-scope: ITR XML) vs 04_ARCH (no mention of ITR XML at all) | ✅ Consistent omission. | None. |

---

## Missing Dependencies Identified

| # | Gap | Blocking? | Action |
|---|-----|----------|--------|
| G1 | OCR vendor is not chosen — architecture says "specialized OCR/doc AI provider" but no vendor named. Extraction mapper code is vendor-specific. | Blocks 08_EXTRACTION | Decide OCR vendor before coding extraction workers. Mark as assumption in 08_EXTRACTION. |
| G2 | KMS strategy mentioned in architecture but not in PRD NFRs (field-level encryption). | Blocks 05_SCHEMA | Add KMS/field encryption notes to 05_SCHEMA when designing PAN/Aadhaar columns. |
| G3 | Temporal server (managed vs self-hosted) not decided. Affects DevOps setup. | Partially blocks 07_WORKFLOWS | Note decision as open in 07_WORKFLOWS; default to Temporal Cloud for v1. |
| G4 | Audit retention period (7 years) in PRD — architecture does not mention database archival/partition strategy for audit table. | Blocks 05_SCHEMA | Add partitioning/archival strategy to 05_SCHEMA for `audit_events`. |
| G5 | Pre-signed URL expiry (15 min) mentioned in architecture but not in PRD FRs. | Blocks 06_API | Document in 06_API under document upload endpoint. |

---

## Terminology Normalization

| Concept | Terms Used Across Documents | Canonical Term |
|---------|---------------------------|----------------|
| Tax year | "tax year", "AY", "FY", "assessment year" | **Assessment Year (AY)** — use "AY 2025-26" format throughout. |
| User's tax position | "tax position", "tax estimate", "tax liability" | **Tax Estimate** — "liability" implies finality; use "estimate" unless context is final filing. |
| Extraction output | "structured data", "extracted data", "extraction result", "normalized output" | **ExtractionResult** (code) / "extraction output" (prose). |
| Document version | "version", "document version", "re-upload" | **DocumentVersion** (code) / "document version" (prose). |
| Computation | "computation run", "tax computation", "estimate run" | **ComputationRun** (code) / "computation run" (prose). |
| Confidence | "confidence score", "confidence threshold", "OCR confidence" | **extraction confidence score** (prose) / `confidence_score` (code). |
| Admin | "admin", "ops", "internal team", "operator" | **Operator** (prose for internal team) / **Admin** (role name in RBAC). |

---

## Required Updates to Earlier Documents

| Document | Update Required |
|----------|----------------|
| 03_PRD.md (FR-1) | Add: "Maximum file size enforced at S3 pre-signed URL policy level (20 MB)." |
| 03_PRD.md (NFR table) | Add: "Pre-signed URL expiry: 15 minutes from issuance." |
| 04_ARCHITECTURE.md | Add note: "OCR vendor selection is a pre-coding dependency; pipeline is designed to be vendor-agnostic at the interface level." |
| All future docs | Use canonical terms from the table above. |

---

*Stitching complete — no blocking contradictions. 5 gaps identified; all are forward-resolvable in later artifacts.*
*Proceeding to Prompt 5 (Database Schema).*
