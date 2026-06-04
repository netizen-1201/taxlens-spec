# 01 — Scope Clarification

> **Role:** Senior Product Manager & Fintech Systems Analyst
> **Master context applied:** India-focused income-tax SaaS, start narrow, compliance-first.

---

## Confirmed Product Requirements

1. **Document upload** — users can upload tax-relevant documents (PDFs, images, Excel exports) to the platform.
2. **Data extraction** — the platform extracts structured fields from uploaded documents using OCR and/or document AI.
3. **Financial summary generation** — aggregate income, deductions, TDS, and other financial data into a structured summary for the relevant tax year.
4. **Filing-support output** — produce a pre-filled or partially-filled representation of the user's tax position suitable for CA review or self-review; NOT the act of e-filing itself in v1.
5. **Tax estimate** — compute an estimated income tax payable based on extracted and user-supplied data, explicitly labelled as an estimate.
6. **Indian tax regime support** — support computation under both the old tax regime and the new tax regime for the relevant assessment year.
7. **Indian accountancy workflows** — designed to fit how Indian CAs and individual taxpayers work (document-centric, TDS-heavy, AIS/26AS reconciliation is central).
8. **Auditability** — every extraction, correction, computation, and report must be traceable to inputs and versions.
9. **Consent management** — explicit, logged user consent before data processing and document storage.
10. **Versioning** — documents, extraction outputs, tax computations, and generated reports must be versioned, not mutated in place.
11. **Compliance-friendly design** — architecture and data flows must support future ERI or Tax API vendor integrations without a rewrite.

---

## Implied Assumptions

| # | Assumption | Risk if Wrong |
|---|-----------|---------------|
| A1 | The product is B2C or B2B2C (individuals or CAs managing clients), not pure B2B enterprise. | Affects auth model, multi-tenancy depth, and pricing. |
| A2 | The primary tax type is income tax (ITR). GST, TDS compliance for businesses, and payroll tax are out of scope for v1. | If GST is expected, schema and tax engine are significantly different. |
| A3 | Users upload documents themselves; the platform does not pull documents directly from IT portal, DigiLocker, or AA (Account Aggregator) in v1. | Pull-based ingestion is a separate integration problem. |
| A4 | The platform is not an ERI (e-Return Intermediary) in v1; it produces filing-support output only. | If direct e-filing is expected, CBDT licensing, ERI onboarding, and DSC integration are required. |
| A5 | The primary currency is INR; multi-currency income (foreign salary, ESOP abroad) is out of v1 scope. | Foreign asset disclosure (Schedule FA) and DTAA logic are non-trivial. |
| A6 | The MVP covers one tax year at a time, not multi-year consolidated view. | Multi-year carry-forwards (losses, TDS credits) add engine complexity. |
| A7 | OCR/extraction is delegated to a third-party provider (e.g., AWS Textract, Azure Form Recognizer, Nanonets). The platform does not build its own OCR model. | Vendor lock-in risk; accuracy dependency. |
| A8 | The product stores encrypted copies of uploaded documents. It is not a "zero-knowledge" system. | If users expect zero-storage, this is a fundamental architecture change. |
| A9 | Extraction confidence below a threshold triggers a human-review step, not automatic rejection or acceptance. | Defines UX flow for review screens. |
| A10 | The admin/ops team is internal (Anthropic/startup employees), not external CAs. | Affects admin access model and data handling obligations. |

---

## Missing Decisions

| # | Decision Needed | Impact |
|---|----------------|--------|
| D1 | **Target user segment for MVP** — salaried, freelancer/consultant, small business owner, trader, or CA-as-intermediary? | Changes documents supported, tax rules engine scope, and UX flow. |
| D2 | **Tax year in scope for launch** — AY 2025-26 (FY 2024-25) only, or also AY 2024-25 with amended rules? | Rules versioning and retroactive computation. |
| D3 | **Direct e-filing in v1?** — Yes (needs ERI), No (filing-support only), or "export ITR XML/JSON for user to upload manually"? | If XML export, need to conform to ITR schema versions released by CBDT. |
| D4 | **CA/intermediary mode** — can a CA manage multiple client profiles under one org account? | Multi-tenancy depth, RBAC, and data segregation requirements. |
| D5 | **Paid or free tier at launch?** — Freemium, subscription, per-computation pricing? | Doesn't change architecture much, but affects onboarding and limits. |
| D6 | **Document retention period** — How long are uploaded documents stored? Who decides — user, platform policy, or compliance mandate? | Data retention architecture, deletion workflows, cost. |
| D7 | **PII encryption standard** — AES-256 at rest, field-level encryption for specific columns, or envelope encryption via KMS? | Schema and storage design. |
| D8 | **Mobile-first or desktop-first for v1?** — Upload and review flows on mobile may require different UX than desktop. | Frontend IA decisions. |
| D9 | **OCR vendor choice** — Which provider is approved? This affects extraction pipeline design and field-mapping contracts. | Pipeline code is vendor-specific. |
| D10 | **Deployment target** — AWS India region (ap-south-1), Azure India, or self-hosted? | Latency, data residency, and compliance obligations. |

---

## High-Value Clarifying Questions

1. **Who is the primary user at launch?** Salaried individual managing their own taxes, a freelancer with mixed income, or a CA managing 10–100 clients?
2. **Is direct e-filing (ERI route) a requirement for v1, or is generating filing-support output sufficient?** If ERI is v1, that requires CBDT registration and changes the entire compliance posture.
3. **Which tax forms are in scope for v1?** ITR-1 (salaried, single-house, <₹50L), ITR-2 (capital gains, multiple houses), ITR-3 (business income), ITR-4 (presumptive)?
4. **What does "filing-support output" mean in your context?** A PDF summary for CA review? A JSON export? A pre-filled ITR form?
5. **Is AIS/TIS reconciliation (matching extracted data against the IT portal's pre-filled data) a v1 requirement?** This requires users to upload their AIS export, which many users don't know how to do.
6. **Should the platform support both old and new regime computation in v1, or new regime only?** New regime is the default from FY 2024-25; however, many high-deduction users still benefit from the old regime.
7. **Will users correct extracted data themselves, or does a CA or operator review corrections before they are locked?** This determines whether a maker-checker flow is required at the extraction step.
8. **Is there a multi-tenant CA portal requirement?** Can a CA create an account, invite clients, and view their tax positions across clients?
9. **What is the expected volume at launch?** 100 users, 1,000 users, or 10,000+? This affects queue sizing, storage cost estimates, and OCR vendor tier selection.
10. **What is the data residency requirement?** Must user documents and PII stay within Indian data centers? (DPDP Act 2023 considerations.)
11. **Will the platform handle capital gains from equity/mutual funds?** STCG (15% flat), LTCG (10% above ₹1L threshold) are common for salaried users with SIPs, but require broker statement parsing.
12. **Are salary slips from all payroll formats (PDF, Excel, HTML exports from Zoho/Keka/GreytHR) expected to be parseable, or only standard-format PDFs?** Format variability is the hardest OCR problem.
13. **What is the monetization model?** This affects whether user-level computation limits, per-document charges, or subscription gates need to be enforced in the platform.
14. **Is there a white-label or embed requirement?** Some fintechs and banks want to embed tax tools. This changes API design and branding requirements.
15. **What is the handling strategy for partial or incomplete document sets?** Can the system produce an estimate with missing documents flagged, or does it require a complete set before computing?

---

*Artifact status: DRAFT — awaiting answers to clarifying questions before MVP definition.*
*Next step: Run Prompt 2 (MVP Definition) after key decisions D1, D3, D4 are resolved.*
