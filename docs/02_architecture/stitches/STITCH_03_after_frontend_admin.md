# Stitching Report — After Artifacts 10–12

> **Role:** Systems Editor
> **Scope:** 10_COMPLIANCE.md, 11_FRONTEND.md, 12_ADMIN.md
> **Checked against:** 01–09, STITCH_01, STITCH_02

---

## Contradictions Identified

| # | Location | Issue | Resolution |
|---|----------|-------|-----------|
| C6 | 10_COMPLIANCE (Consent Gate — step 5) — introduces `403 CONSENT_REFRESH_REQUIRED` | 06_API does not list this response on any endpoint | Add `403 CONSENT_REFRESH_REQUIRED` as a documented error on `POST /documents/upload-url` and all CS-05 to CS-14 endpoints. Add to 06_API in next pass. |
| C7 | 11_FRONTEND (S-09, Deductions Tab) — shows 80CCD(1B) but not 80CCD(2) | STITCH_02 C5 required adding 80CCD2 to permitted section codes; 09_TAX_ENGINE confirms it is permitted in new regime | Add 80CCD(2) field to deduction UI in S-09. Note it is employer NPS contribution (source: `extracted` from Form 16 Part B); user does not enter it manually. |
| C8 | 12_ADMIN (A-05) — tax rule activation uses `pending_approval` record | 06_API has no approval endpoints; 07_WORKFLOWS has no maker-checker workflow | New: document `POST/GET /admin/approvals` and `POST /admin/approvals/{id}/approve` + `reject` endpoints in 06_API. Add `MakerCheckerWorkflow` as W9 in 07_WORKFLOWS. Mark as forward-dependency for 13_BACKLOG. |
| C9 | 12_ADMIN — references `/admin/login` as a separate path | 06_API only documents `/auth/login` for all users | Decision: Admin login reuses `/auth/login` endpoint; the frontend routes to `/admin` if `role IN ('admin', 'operator')` post-login. No separate endpoint needed. Note in 06_API. |
| C10 | 11_FRONTEND (S-05, AY selector) — mentions AY 2024-25 as selectable | 03_PRD scopes v1 to AY 2025-26 only; AY 2024-25 is secondary | Mark AY 2024-25 selector as a "coming soon" disabled option in v1. Update S-05 to note this. |

---

## Missing Dependencies Identified

| # | Gap | Blocking? | Action |
|---|-----|----------|--------|
| G11 | `admin_pending_actions` table (for maker-checker) is referenced in 10_COMPLIANCE and 12_ADMIN but not defined in 05_SCHEMA | Blocks maker-checker feature | Add table definition to 05_SCHEMA in next schema pass; include in 13_BACKLOG as a migration task. Columns: id, action_type, initiated_by, target_resource_type, target_resource_id, payload_json, status (`pending`/`approved`/`rejected`/`expired`), initiated_at, expires_at, resolved_by, resolved_at. |
| G12 | `v_audit_events_safe` DB view (PII-masked audit view for operators) defined in 10_COMPLIANCE but not in 05_SCHEMA | Blocks operator audit screen | Add as a view definition note to 05_SCHEMA; implement as a named view in DB migration. Include in 13_BACKLOG. |
| G13 | `GET /admin/health` endpoint (A-08 in 12_ADMIN) is not in 06_API | Blocks admin health screen | Add health check endpoint to 06_API admin section. Response shape: `{services: [{name, status, latency_ms, checked_at}], overall: 'healthy'|'degraded'|'down'}` |
| G14 | `MakerCheckerWorkflow` (W9) is needed for CS-17, CS-18, CS-19 actions from 10_COMPLIANCE but not defined in 07_WORKFLOWS | Blocks all maker-checker flows | Add W9 definition to 07_WORKFLOWS. Pattern: Initiate → store pending_action → signal second admin → Approve/Reject → execute or cancel. |
| G15 | Gaps G6 (tax rule seed data) and G10 (marginal relief pseudocode) from STITCH_02 remain open | G6 blocks computation in any deployed environment | Both must appear in 13_BACKLOG as concrete tasks with acceptance criteria. |

---

## Terminology Normalization (Additions)

| Concept | Terms Used | Canonical Term |
|---------|-----------|----------------|
| Pending admin review action | "pending_approval", "maker-checker action", "approval queue" | **PendingAdminAction** (code) / "pending approval" (prose) |
| Admin approval flow | "maker-checker", "two-admin approval", "secondary approval" | **Maker-checker** (prose) / `MakerCheckerWorkflow` (code) |
| Consent version outdated response | "CONSENT_REFRESH_REQUIRED", "re-consent required" | `403 CONSENT_REFRESH_REQUIRED` (API error code) / "consent refresh required" (prose) |
| IT Act / SPDI | "IT Act 2000 / Intermediary Rules", "IT Act 2000 / SPDI Rules 2011" | **IT Act 2000 / SPDI Rules 2011** (full cite in compliance docs) |
| Admin viewing user data | "admin viewed user data", "operator data access" | `admin.user_data_viewed` (audit event code) |

---

## Required Updates to Earlier Documents

| Document | Update Required |
|----------|----------------|
| 05_SCHEMA.md | Add `admin_pending_actions` table definition (G11). Add `v_audit_events_safe` view note (G12). |
| 06_API.md | Add `403 CONSENT_REFRESH_REQUIRED` to CS-05 through CS-14 endpoint error cases (C6). Add `/admin/approvals` endpoint group (C8 + G14). Add `GET /admin/health` (G13). Note admin login reuses `/auth/login` (C9). |
| 07_WORKFLOWS.md | Add W9 `MakerCheckerWorkflow` definition (C8 + G14). |
| 11_FRONTEND.md (S-05) | Mark AY 2024-25 in AY selector as disabled in v1 (C10). |
| 11_FRONTEND.md (S-09) | Add 80CCD(2) as a read-only extracted field under deductions; not user-entered (C7). |
| 09_TAX_ENGINE.md | Add marginal relief pseudocode (STITCH_02 G10 — still open; add to 13_BACKLOG). |

---

## Open Items Carried into Backlog (13_BACKLOG.md)

The following are unresolved and must appear as concrete tasks in the backlog:

1. **G6** — Tax rules seed data for AY 2025-26 (old + new regime) must be defined and seeded in the DB.
2. **G10** — Marginal relief pseudocode in tax engine implementation.
3. **G11** — `admin_pending_actions` table migration.
4. **G12** — `v_audit_events_safe` view migration.
5. **G13** — `GET /admin/health` endpoint implementation.
6. **G14** — `MakerCheckerWorkflow` (W9) Temporal workflow implementation.
7. **C8** — `/admin/approvals` API endpoints.
8. **STITCH_02 C5** — `80CCD2` added to `section_code` permitted enum.

---

*Stitching complete — 5 new contradictions (all resolvable), 5 new gaps. G11 (pending_actions table) and G14 (MakerCheckerWorkflow) are the most structurally new additions — both are v1 features that were implied by compliance but not yet modeled.*
*Proceeding to Prompt 13 (Delivery Backlog).*
