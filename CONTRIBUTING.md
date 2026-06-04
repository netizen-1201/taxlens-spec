# Contributing to TaxLens

> **Note:** This repository currently contains the **complete v1 product specification**.  
> Implementation is tracked in [`docs/09_delivery/`](docs/09_delivery/) as 83 ordered chunks (C01–C83).

---

## How to Read This Repo

1. **Start with [`README.md`](README.md)** — project overview and folder map
2. **Scan [`ARCHITECTURE.md`](ARCHITECTURE.md)** — 2-minute system summary with diagram
3. **Deep-dive by role:**
   - *Product / BA:* [`docs/01_product/03_prd.md`](docs/01_product/03_prd.md)
   - *Backend / DevOps:* [`docs/02_architecture/04_architecture.md`](docs/02_architecture/04_architecture.md) + [`docs/03_data/05_schema.md`](docs/03_data/05_schema.md)
   - *Frontend:* [`docs/07_frontend/11_frontend.md`](docs/07_frontend/11_frontend.md)
   - *Compliance / Legal:* [`docs/06_compliance/10_compliance.md`](docs/06_compliance/10_compliance.md)
   - *Engineering lead:* [`docs/09_delivery/14a_index.md`](docs/09_delivery/14a_index.md)

---

## Specification Change Guidelines

TaxLens is a **compliance-regulated fintech design**. Any change to the following artifacts requires explicit justification:

- [`docs/06_compliance/10_compliance.md`](docs/06_compliance/10_compliance.md) — consent gate logic, audit taxonomy, DPDP/IT Act controls
- [`docs/05_backend/09_tax_engine.md`](docs/05_backend/09_tax_engine.md) — tax slab rules, rebate thresholds, surcharge logic
- [`docs/03_data/05_schema.md`](docs/03_data/05_schema.md) — PII field encryption strategy, audit_events immutability

**Process:** Open an Issue describing the contradiction or gap → reference the original artifact and stitching report → propose resolution → maintainers review before PR merge.

---

## Implementation Contribution Rules

If you are implementing chunks from the delivery backlog:

1. **Follow chunk order.** C01 must complete before C02. Dependencies are strict.
2. **Save every output file** before opening a new session or chunk.
3. **Do not skip ⚠️ or 🔒 chunks.** These require human review:
   - `⚠️` — Claude draft + human validation required
   - `🔒` — Human-led only (tax rules, KMS encryption, admin MFA, data deletion workflow)
4. **Preserve the Master Context Block** at the top of every implementation session (see `14a_index.md`).
5. **Never invent candidate information** — the spec forbids hallucinating metrics, employers, or credentials.

---

## Code Style

| Layer | Style |
|-------|-------|
| NestJS (API) | TypeScript strict mode, snake_case JSON, class-validator DTOs, UUID PKs, ISO 8601 UTC timestamps |
| Next.js (Web) | App Router, Tailwind CSS, React Query (server state), Zustand (UI state) |
| Python (Workers) | Pydantic models, Temporal activity stubs, boto3 for AWS, SQLAlchemy for DB |
| Database | PostgreSQL migrations (TypeORM or Flyway), partial indexes, RLS where noted |

---

## Issue Labels

| Label | Use |
|-------|-----|
| `spec-contradiction` | Two artifacts conflict; needs stitching |
| `compliance-risk` | Affects DPDP, IT Act, or non-ERI boundary |
| `chunk-implementation` | Active coding task from 14a–14d |
| `frontend-screen` | UI/UX change to 11_frontend.md |
| `tax-rule-update` | CBDT amendment requiring new rule version |
| `security` | Auth, encryption, or audit trail concern |

---

## Questions?

Open a Discussion or reach out: **sahilzafar12001@gmail.com**

---

*This contributing guide is itself versioned. Last updated: 2025-06.*
