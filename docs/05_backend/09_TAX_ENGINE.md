# 09 — Tax Rules Engine

> **Role:** Tax Systems Designer for Indian Income-Tax Software
> **Segment:** Salaried individuals, ITR-1 eligible
> **AY scope:** AY 2025-26 (FY 2024-25) — primary; AY 2024-25 (FY 2023-24) — secondary
> **Disclaimer:** This design describes the structure for an estimation engine. All tax rules referenced are based on publicly known Finance Act provisions. This document does not constitute legal or tax advice. Always validate rules against official CBDT notifications before production deployment.

---

## 1. Scope of the Tax Engine (v1)

**In scope:**
- Heads of income: Salary only (for ITR-1 segment)
- Other income (added to total): Savings interest (80TTA/80TTB), FD interest
- Deductions: Standard deduction, 80C (aggregate cap), 80D, HRA, 80TTA/80TTB, 80G (with sub-limits), 80CCD(1B)
- Tax regimes: Old regime and new regime — parallel computation
- Surcharge: Marginal relief computation for incomes near surcharge thresholds
- Cess: Health and education cess (4%)
- Rebate: Section 87A
- TDS reconciliation: Compare engine-computed tax vs TDS credits
- Rule versioning: By (assessment_year, regime, version)

**Not in scope (v1):**
- Capital gains (STCG, LTCG)
- Business / professional income
- House property income (rental or self-occupied loan interest deduction)
- Foreign income / DTAA
- Set-off of losses from previous years
- Advance tax installment computation
- AMT (Alternative Minimum Tax)
- NRI-specific provisions

---

## 2. Rule Objects and Versioning Model

Each rule set is a JSON document stored in the `tax_rules` table, versioned by `(assessment_year, regime, version)`.

### Rule Object Structure

```json
{
  "meta": {
    "assessment_year": "2025-26",
    "regime": "new",
    "version": "v1",
    "effective_from": "2025-04-01",
    "source_reference": "Finance Act 2024 — new regime default"
  },
  "standard_deduction": 75000,
  "basic_exemption_limit": 300000,
  "rebate_87a": {
    "applicable_income_limit": 1200000,
    "max_rebate": 60000
  },
  "tax_slabs": [
    { "from": 0,       "to": 300000,  "rate": 0.00 },
    { "from": 300001,  "to": 700000,  "rate": 0.05 },
    { "from": 700001,  "to": 1000000, "rate": 0.10 },
    { "from": 1000001, "to": 1200000, "rate": 0.15 },
    { "from": 1200001, "to": 1500000, "rate": 0.20 },
    { "from": 1500001, "to": null,    "rate": 0.30 }
  ],
  "surcharge": [
    { "income_above": 5000000,  "income_upto": 10000000, "rate": 0.10 },
    { "income_above": 10000000, "income_upto": 20000000, "rate": 0.15 },
    { "income_above": 20000000, "income_upto": 50000000, "rate": 0.25 },
    { "income_above": 50000000, "income_upto": null,     "rate": 0.25 }
  ],
  "cess_rate": 0.04,
  "permitted_deductions": ["standard_deduction", "nps_employer_contribution_80ccd2"],
  "disallowed_deductions": ["80C", "80D", "HRA", "80TTA", "80G", "80CCD1B"]
}
```

For the old regime, `permitted_deductions` lists all supported deduction codes; `disallowed_deductions` is empty.

**Versioning rule:** Only one rule set is `is_active = true` per `(assessment_year, regime)` at any time. A mid-year amendment creates a new version row with a future `effective_from` date; the engine resolves which version applies based on the computation trigger date.

---

## 3. Input Model (ComputationInput)

The engine receives a normalized input object assembled by `AssembleComputationInputsActivity`:

```typescript
interface ComputationInput {
  user_id: string;
  assessment_year: string;           // e.g., "2025-26"
  regime: 'old' | 'new';
  residential_status: 'resident' | 'nri';  // NRI = v2 scope; must be 'resident' in v1

  // --- Income ---
  gross_salary: number;              // from Form 16 Part B / salary slips
  standard_deduction_claimed: number; // always from rules (50000 old / 75000 new)
  taxable_salary: number;            // gross_salary - standard_deduction - exempt_allowances

  salary_other_allowances_exempt: number; // HRA, LTA, etc. already exempt in Form 16

  other_income: {
    savings_interest: number;        // 80TTA/TTB source
    fd_interest: number;
    // v2: dividend, capital gains, rental
  };

  // --- Deductions (old regime only; ignored for new) ---
  deductions: {
    section_80c: number;             // capped at 150000 in engine
    section_80d_self: number;        // capped at 25000 (non-senior) / 50000 (senior)
    section_80d_parents: number;     // capped at 25000/50000
    section_80tta: number;           // capped at 10000 (savings interest)
    section_80ttb: number;           // capped at 50000 (senior citizen FD/savings)
    section_80g: number;             // partial support; 50% or 100% depending on org
    section_80ccd_1b: number;        // NPS additional; capped at 50000
    hra_deduction: number;           // computed separately if not in Form 16
    professional_tax: number;        // actual; no cap
  };

  // --- TDS Credits ---
  tds_credits: Array<{
    deductor_tan: string;
    section: string;                  // e.g., "192" for salary
    amount: number;
  }>;

  advance_tax_paid: number;
  self_assessment_tax_paid: number;

  // --- Meta ---
  rule_version: string;              // resolved by engine before running
  computation_run_id: string;        // for traceability
  locked_extraction_ids: string[];   // audit trail
}
```

**Validation before engine run:**
- `gross_salary` must be > 0
- `regime` must be 'old' or 'new'
- `residential_status` must be 'resident' for v1
- `assessment_year` must have a corresponding active rule set
- All deduction amounts must be ≥ 0
- If regime = 'new', deduction inputs (80C, 80D, etc.) are accepted as input but marked as `ignored_in_new_regime` in the output — they are NOT applied to computation.

---

## 4. Output Model (ComputationResult)

```typescript
interface ComputationResult {
  computation_run_id: string;
  assessment_year: string;
  regime: 'old' | 'new';
  rule_version: string;
  disclaimer: string;  // hardcoded: "ESTIMATE ONLY. NOT A LEGAL FILING."

  income_summary: {
    gross_salary: number;
    standard_deduction: number;
    taxable_salary: number;
    savings_interest: number;
    fd_interest: number;
    total_gross_income: number;      // sum of all income heads
  };

  deductions_applied: {
    section_80c: number;             // 0 if new regime
    section_80d: number;             // 0 if new regime
    section_80tta_or_ttb: number;
    section_80g: number;
    section_80ccd_1b: number;
    hra_deduction: number;
    professional_tax: number;
    total_deductions: number;
    notes: string[];                 // e.g., "80C capped at ₹1,50,000", "Ignored: 80D (new regime)"
  };

  taxable_income: number;            // total_gross_income - total_deductions

  slab_breakdown: Array<{
    slab_label: string;              // e.g., "₹7,00,001 – ₹10,00,000 @ 10%"
    taxable_in_slab: number;
    rate: number;
    tax: number;
  }>;

  tax_before_rebate: number;
  rebate_87a_applied: number;        // 0 if income > threshold
  tax_after_rebate: number;
  surcharge: number;
  surcharge_marginal_relief: number; // marginal relief amount if applicable
  cess: number;
  total_tax_liability: number;       // tax_after_rebate + surcharge + cess

  tds_reconciliation: {
    total_tds_credit: number;
    total_advance_tax: number;
    total_self_assessment_tax: number;
    total_taxes_paid: number;
    net_payable: number;             // positive = payable
    net_refundable: number;          // positive = refund expected
    reconciliation_warnings: Array<{
      warning_type: string;
      message: string;
    }>;
  };

  regime_comparison?: {              // populated if computation triggered as 'compare'
    old_regime_tax: number;
    new_regime_tax: number;
    recommended_regime: 'old' | 'new';
    savings: number;
  };

  computed_at: string;               // ISO 8601 UTC
  input_snapshot_id: string;         // reference to stored input snapshot
}
```

---

## 5. Calculation Traceability

Every step of the computation is recorded in the result JSON. The frontend and PDF report render the full slab breakdown, not just the final number. This is critical for user trust and CA review.

**Slab computation example (new regime, ₹12,50,000 taxable income):**

```
₹0        – ₹3,00,000    @  0%  = ₹0
₹3,00,001 – ₹7,00,000    @  5%  = ₹20,000
₹7,00,001 – ₹10,00,000   @ 10%  = ₹30,000
₹10,00,001 – ₹12,00,000  @ 15%  = ₹30,000
₹12,00,001 – ₹12,50,000  @ 20%  = ₹10,000
                           Total = ₹90,000

Section 87A rebate check: taxable income ₹12,50,000 > ₹12,00,000 threshold → rebate = ₹0
Surcharge: income < ₹50,00,000 → surcharge = ₹0
Cess: ₹90,000 × 4% = ₹3,600
Total tax = ₹93,600
```

Each `slab_breakdown` row is stored as a JSON array — never computed on the fly in the frontend.

**Marginal relief note:** For incomes just above a surcharge threshold (e.g., ₹50,10,000 when surcharge kicks in at ₹50,00,000), surcharge may exceed the incremental income. The engine computes marginal relief and records it explicitly in `surcharge_marginal_relief`.

---

## 6. Auditability Requirements

| Requirement | Implementation |
|-------------|---------------|
| Every computation must be traceable to exact inputs | `input_snapshot` stored as JSONB in `computation_runs` |
| Rule version used must be recorded | `rule_version` column in `computation_runs` |
| Deductions ignored (new regime) must be disclosed | `notes[]` in `deductions_applied` |
| 87A rebate applied or not must be stated | `rebate_87a_applied` always present (0 if not applied) |
| Computation is immutable | `computation_runs` is append-only; re-run creates new row |
| All computation events are audit-logged | `ComplianceEventWorkflow` writes `computation.completed` audit event |
| Disclaimers are baked into the result object | `disclaimer` field hardcoded; cannot be removed from output |

---

## 7. Manual Review Boundary (v1)

The following items require user confirmation or manual entry; the engine does NOT auto-compute them:

| Item | Why Manual |
|------|-----------|
| HRA deduction | Requires rent paid, city type, employer HRA component — not reliably extractable from Form 16 in all cases |
| 80G deductions | Percentage (50% vs 100%) depends on specific organization; not reliably classifiable from certificate |
| Relief u/s 89 (arrear relief) | Complex multi-year calculation; out of engine scope in v1 |
| Professional tax for states other than Maharashtra/standard cases | State-specific rules; user should enter actual amount |
| Double taxation relief (DTAA) | NRI / foreign income; out of v1 scope entirely |
| Exempt allowances beyond those in Form 16 | LTA actuals, meal vouchers, transport — user self-declares |
| Section 80C itemization | User enters aggregate or itemized; engine applies cap; it does not validate individual investment eligibility |

These items are collected via the `DeductionInput` UI and passed as-is to the engine with `source: 'user_input'`. The engine trusts user inputs and applies caps. It does not verify eligibility of individual investments.

---

## 8. Error Handling in Engine

| Error | Code | Behavior |
|-------|------|---------|
| No active tax rule for requested AY + regime | `NO_RULE_FOUND` | Fail computation; return error |
| `residential_status` = 'nri' | `NRI_NOT_SUPPORTED` | Fail computation; return error with message |
| `gross_salary` = 0 or negative | `INVALID_INCOME` | Fail computation |
| Deduction amount negative | `INVALID_DEDUCTION` | Reject specific field; continue with 0 |
| 80C > 150000 input | Auto-cap to 150000; add note in `deductions_applied.notes[]` | |
| 80D claim > statutory limit | Auto-cap; add note | |
| TDS credit sum < 0 | `INVALID_TDS` | Fail computation |

---

*Artifact status: TAX ENGINE DESIGN COMPLETE.*
*Next: Stitching check (Prompts 5–9), then Prompt 10 (Compliance Controls).*
