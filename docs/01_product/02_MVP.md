# 02 — MVP Definition

> **Role:** SaaS Product Strategist
> **Master context applied:** India income-tax SaaS, start narrow, compliance-first.
> **Assumption:** Scope Q1 answered — target segment selected as per analysis below.

---

## Recommended Initial Segment: Salaried Individuals (ITR-1 filers)

### Reasoning

| Factor | Salaried Individual | Freelancer/Consultant | Small Business | Trader | CA as Intermediary |
|--------|-------------------|----------------------|---------------|--------|-------------------|
| Document structure | Highly standardized (Form 16, payslips, AIS) | Variable (invoices, TDS certs, bank stmts) | Complex (P&L, GST, balance sheet) | Very complex (broker statements, P&L) | Depends on client mix |
| Tax computation complexity | Low–Medium (ITR-1 / ITR-2) | Medium (ITR-3, presumptive) | High (ITR-3, full books) | Very High (speculative, F&O logic) | All of the above |
| Volume of target users | Largest (100M+ salaried in India) | Large but fragmented | Medium | Niche | Niche but high-value |
| Pain felt acutely | Yes — Form 16 confusion, AIS mismatches, 26AS reconciliation | Yes — but dispersed pain | Yes — but needs CA | Yes — but needs expert | Yes — but they're the expert |
| OCR accuracy requirements | Medium-high (Form 16 is semi-standard) | Hard (diverse document formats) | Very hard | Very hard (broker statement formats) | Depends |
| ERI requirement in v1 | No — ITR-1 can be filed by user directly | No | Sometimes | Sometimes | Yes |
| Time-to-MVP | Fastest | Moderate | Slow | Very slow | Moderate |

**Winner: Salaried Individuals (ITR-1 filers).**

Primary job: "Help me understand my tax position from my Form 16, tell me if TDS matches my actual liability, and give me something I can file or hand to my CA."

Secondary job: "Tell me if I save money under old vs new regime."

---

## MoSCoW Prioritization

### Must Have (MVP blocker if absent)
| Feature | Why |
|---------|-----|
| Secure document upload (PDF, image) | Core action; without it there's no product |
| Form 16 Part A + Part B extraction | Primary source of salaried income and TDS data |
| AIS/TIS CSV/PDF upload and parsing | Government's own pre-filled data; critical for reconciliation |
| Form 26AS upload and TDS extraction | TDS credit reconciliation is the #1 pain point |
| Tax estimate computation (old + new regime) | Core output; must show side-by-side comparison |
| Standard deduction, Section 87A rebate, basic surcharge/cess | Without these, estimates are wrong |
| Extraction review/correction UI | Users must be able to fix OCR errors before computing |
| Downloadable tax summary PDF | Takeable artifact; CA-ready output |
| Versioned computation runs | User must be able to recompute after edits |
| Consent capture before processing | Legal requirement; cannot ship without this |
| Audit log per user action | Compliance baseline; non-negotiable |
| Auth (email/password + OTP, 2FA optional) | Secure access to sensitive financial data |
| Data encrypted at rest | Non-negotiable for PII + financial documents |

### Should Have (strong MVP value)
| Feature | Why |
|---------|-----|
| Salary slip extraction (basic fields: gross, basic, deductions) | Supplements Form 16 for in-year estimates |
| Interest certificate (savings, FD) upload and extraction | Section 80TTA/80TTB deductions; common for salaried users |
| HRA computation assistant | Very high-demand; requires rent inputs, metro/non-metro flag |
| Section 80C deduction input | LIC, ELSS, PPF, EPF — user-supplied, not extracted |
| Regime comparison screen with deduction optimizer | High perceived value |
| Email notification on job completion | UX quality improvement |
| Mobile-responsive upload and review | Large share of Indian internet users are mobile-first |

### Could Have (post-MVP, pre-Series A)
| Feature | Why deferred |
|---------|-------------|
| Capital gains statement parsing (equity/MF) | Broker statement format variability; hard OCR problem |
| ITR XML export | Requires conformance to CBDT ITR schema versions |
| CA portal / multi-client view | Multi-tenancy RBAC complexity |
| House property income computation | Loan interest, rent income — medium complexity |
| NPS deduction (80CCD 1B) | User-supplied; easy to add but not blocking |
| In-product chat support | Nice to have, not core |
| Advance tax installment calculator | Useful but secondary |

### Won't Have in MVP
| Feature | Why excluded |
|---------|-------------|
| Direct e-filing via ERI | Requires CBDT ERI registration, DSC integration, legal compliance |
| GST filing support | Entirely different domain, different user |
| Business/professional income (ITR-3, ITR-4) | Out of salaried segment scope |
| F&O / speculative trading (ITR-3 Schedule) | Very complex, different user segment |
| Foreign asset disclosure (Schedule FA) | NRI / expat complexity; different user profile |
| Multi-language (Hindi, regional) | Deferred; English-first for v1 |
| White-label / embedded mode | API-first product refinement comes after core product works |
| Offline / desktop app | Web-first only |
| DigiLocker / AA-based document pull | Integration complexity; manual upload is sufficient for v1 |

---

## Smallest Lovable Product

**One sentence:** Upload your Form 16 and AIS, get a clear old-regime vs new-regime tax estimate with a reconciled TDS summary, downloadable as a PDF you can share with your CA or use to self-file ITR-1.

**Minimum flow:**
1. User signs up, consents to data processing.
2. User uploads Form 16 (PDF) and AIS export (PDF or CSV).
3. System extracts: employer name, gross salary, tax-exempt allowances, TDS deducted (Part A), and income breakup (Part B).
4. System extracts: TDS entries and interest/other income from AIS.
5. User reviews extraction on-screen and corrects any errors.
6. User inputs 80C, 80D, HRA (if not in Form 16) via simple form.
7. System computes tax estimate under both regimes with standard deduction, 87A rebate, surcharge, and cess.
8. System generates a structured tax summary PDF.
9. User downloads PDF.

**Success for user:** "I now know whether to choose old or new regime, and I have a document I trust to hand to my CA."

---

## Top 10 Risks of Overbuilding

1. **Building ITR XML export before core extraction accuracy is validated** — shipping a perfectly formatted XML from wrong OCR data is worse than no XML.
2. **Supporting all income heads in v1** — capital gains, business income, and foreign assets each require separate sub-engines; adding them before Form 16 extraction is solid creates a fragile base.
3. **Building the CA portal before individual user flow is proven** — multi-tenancy RBAC complexity can be deferred; CAs will use the individual flow initially.
4. **Implementing ERI integration before product-market fit** — CBDT ERI onboarding is a legal/compliance overhead; direct e-filing is only valuable if users trust the computation first.
5. **Building mobile-native apps before web is stable** — React Native or Flutter adds a second surface to maintain; responsive web covers 80% of mobile use cases.
6. **Over-engineering the tax engine for all deduction categories on day one** — 80C, 80D, HRA, and standard deduction cover ~90% of salaried users; the remaining 10% of exotic deductions can be added iteratively.
7. **Building real-time document pull (DigiLocker/AA) before upload-based flow is working** — pull-based integration requires additional vendor agreements and security audits.
8. **Designing for 100,000 users before validating with 100** — premature horizontal scaling burns runway on infrastructure, not product.
9. **Adding AI-generated tax advice before extraction accuracy is measured** — advice on top of unreliable data creates liability and erodes trust.
10. **Building multi-language support in parallel with core product** — Indian tax forms are in English; English-first is acceptable for the early adopter segment.

---

*Artifact status: APPROVED FOR PRD.*
*Approved segment: Salaried Individuals, ITR-1 primary.*
*Deferred segments: Freelancers, small business, traders, CA portal — all post-v1.*
