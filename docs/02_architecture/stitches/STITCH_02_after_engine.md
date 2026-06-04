# Stitching Report — After Artifacts 05–09

> **Role:** Systems Editor
> **Scope:** 05_SCHEMA.md, 06_API.md, 07_WORKFLOWS.md, 08_EXTRACTION.md, 09_TAX_ENGINE.md
> **Checked against:** 01–04 for backward consistency

---

## Contradictions Identified

| # | Location | Issue | Resolution |
|---|----------|-------|-----------|
| C1 | 06_API (`GET /computations/{id}`) — `standard_deduction` shown as `50000` in example | 09_TAX_ENGINE says old regime = ₹50,000, new regime = ₹75,000 for AY 2025-26 | Update API example response to not hardcode a specific amount; note "value per regime rules." |
| C2 | 07_WORKFLOWS (W4 `ComputationWorkflow`) — regime param is `old`/`new` | 06_API allows `regime: "compare"` which triggers two runs | W4 should note that `"compare"` mode triggers W4 twice (once per regime) and returns both `computation_run_ids`. Add this to 07_WORKFLOWS. |
| C3 | 05_SCHEMA (`deduction_inputs.section_code`) — no enum constraint listed | 09_TAX_ENGINE defines permitted deduction codes | Add a DB-level CHECK constraint or enum note to 05_SCHEMA for `section_code` values. |
| C4 | 08_EXTRACTION — `salary_slip` field `tds_deducted` marked as "No" (not Must-Have) | 09_TAX_ENGINE input model uses `tds_credits[]` from 26AS/AIS as the TDS source | Consistent — salary slip TDS is supplementary, not the primary TDS source. ✅ No action needed, but add a note in 08_EXTRACTION. |
| C5 | 09_TAX_ENGINE — mentions `nps_employer_contribution_80ccd2` as permitted in new regime | 06_API `POST /tax-profiles/{ay}/deductions` does not list `80CCD2` as a valid `section_code` | Add `80CCD2` to the API's permitted `section_code` list. |

---

## Missing Dependencies Identified

| # | Gap | Blocking? | Action |
|---|-----|----------|--------|
| G6 | `tax_rules` table seed data for AY 2025-26 (old + new) is not yet defined. Engine references it but no migration/seed exists. | Blocks computation in production | Define seed data in 13_BACKLOG.md as a required task before computation can be tested. |
| G7 | The `NormalizedIncome` object (08_EXTRACTION downstream output) is not a named DB table — it is constructed in-memory by `AssembleComputationInputsActivity`. Its structure must match `ComputationInput` exactly. | Blocks W4 | Add a data contract note to 07_WORKFLOWS W4 confirming that `AssembleComputationInputsActivity` output = `ComputationInput` interface from 09_TAX_ENGINE. |
| G8 | 06_API `POST /computations` validates "at least one locked extraction" but does not specify what to do if only salary slips are locked (no Form 16). | Blocks edge case handling | Add validation rule: Form 16 OR (salary slips for all 12 months) must be present before computation can proceed. |
| G9 | 05_SCHEMA `extraction_results.extracted_fields` is `jsonb` — the field structure varies per document type. There is no typed schema enforced at DB level. | Partially blocking — DB accepts anything | Acceptable for v1. Add a note in 05_SCHEMA: "Field structure is enforced at application layer via DocumentTypeMapper." |
| G10 | 09_TAX_ENGINE mentions marginal relief for surcharge but no test case or rule structure defines marginal relief calculation. | Partially blocks tax engine implementation | Add marginal relief computation pseudocode to 09_TAX_ENGINE or defer to implementation task in 13_BACKLOG. |

---

## Terminology Normalization (Additions)

| Concept | Terms Used | Canonical Term |
|---------|-----------|----------------|
| Old tax regime | "old regime", "old tax regime", "existing regime" | **Old Regime** (capitalize when referring to the tax system) |
| New tax regime | "new regime", "new tax regime", "default regime" | **New Regime** (same convention) |
| Section 87A | "87A rebate", "Section 87A", "tax rebate u/s 87A" | **Section 87A rebate** |
| Must-Have fields (extraction) | "must-have", "required fields", "core fields" | **Must-Have fields** (capitalize) |
| Computation result | "computation result", "tax result", "tax estimate result" | **ComputationResult** (code) / "computation result" (prose) |
| Input snapshot | "input snapshot", "normalized inputs", "computation inputs" | **InputSnapshot** (code) / "input snapshot" (prose) |

---

## Required Updates to Earlier Documents

| Document | Update |
|----------|--------|
| 06_API.md (`GET /computations/{id}` example) | Remove hardcoded `standard_deduction: 50000`; comment it as "per active tax rule." |
| 06_API.md (`POST /tax-profiles/{ay}/deductions`) | Add `80CCD2` to the list of permitted `section_code` values. |
| 07_WORKFLOWS.md (W4) | Clarify: `regime = 'compare'` in the API triggers two separate W4 runs; both `computation_run_id`s are returned to the client. |
| 05_SCHEMA.md | Add note: `section_code` is validated at application layer against a defined enum; add DB comment or CHECK constraint reference. |
| 09_TAX_ENGINE.md | Add marginal relief pseudocode or defer to backlog task. |

---

*Stitching complete — 5 new contradictions (all resolvable), 5 new gaps. G6 (tax rule seed data) and G7 (input contract alignment) are the most critical to address before coding the computation layer.*
*Proceeding to Prompt 10 (Compliance Controls).*
