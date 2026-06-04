# 14 — Claude-Sized Coding Chunks

> **Role:** Senior Software Engineer
> **Inputs:** 03_PRD, 04_ARCHITECTURE, 05_SCHEMA, 06_API, 07_WORKFLOWS, 08_EXTRACTION, 09_TAX_ENGINE, 10_COMPLIANCE, 11_FRONTEND, 12_ADMIN, 13_BACKLOG, STITCH_01–03
> **Chunk size:** Each chunk fits in one Claude session (one module, one service, one workflow, or 1–3 closely related files).
> **Dependency rule:** Chunks are ordered; do not skip prerequisites.
> **AI-assist legend:** ✅ Claude first draft | ⚠️ Claude draft + human review | 🔒 human-led

---

## How to Use This Document

1. Start at C01. Complete each chunk fully before moving to the next.
2. Save every output file before opening a new session.
3. Each chunk's **Implementation Prompt** is copy-paste ready. Paste the master context block first (below), then the chunk prompt.
4. For chunks marked ⚠️ or 🔒, treat Claude's output as a first draft and validate against the referenced artifacts before committing.

---

## Master Context Block (Paste at Top of Every Session)

```
Project: TaxLens — India-focused income-tax SaaS for salaried individuals (ITR-1 segment).
AY scope: AY 2025-26 primary. Stack: NestJS + TypeScript (API), Next.js + TypeScript (frontend),
Python + FastAPI/Temporal workers (extraction + reports), PostgreSQL, Redis, S3, Temporal.
Key rules: Modular monolith for v1. Append-only for extractions, computations, audit_events.
Consent gate on all CS-05 to CS-14 actions. Non-ERI: never claim to file returns.
All computation output carries disclaimer. PAN/phone field-level encrypted via KMS.
Coding style: TypeScript strict mode. DTOs with class-validator. snake_case JSON.
UUIDs for all PKs. ISO 8601 UTC timestamps.
```

---

## Chunk Index

| Chunk | Epic | Task(s) | Size | AI |
|-------|------|---------|------|----|
| C01 | E01 | T01.1 | M | ✅ |
| C02 | E01 | T01.2 | M | ✅ |
| C03 | E01 | T01.3, T01.4 | S | ✅ |
| C04 | E01 | T01.5 | M | ✅ |
| C05 | E01 | T01.6, T01.7 | M | ⚠️ |
| C06 | E01 | T01.8 | L | ✅ |
| C07 | E01 | T01.9 part 1 | L | ✅ |
| C08 | E01 | T01.9 part 2 | L | ✅ |
| C09 | E01 | T01.9 part 3 | M | ✅ |
| C10 | E02 | T02.1 | M | ✅ |
| C11 | E02 | T02.2 | M | ✅ |
| C12 | E02 | T02.3 | M | ⚠️ |
| C13 | E02 | T02.4 | S | ✅ |
| C14 | E02 | T02.5 | M | 🔒 |
| C15 | E02 | T02.6 | M | ⚠️ |
| C16 | E02 | T02.7, T02.8 | S | ✅ |
| C17 | E02 | T02.9 | S | ✅ |
| C18 | E03 | T03.1 | M | ⚠️ |
| C19 | E03 | T03.2, T03.4 | M | ✅ |
| C20 | E03 | T03.3 | M | ✅ |
| C21 | E03 | T03.5 | M | ✅ |
| C22 | E03 | T03.6 | M | ✅ |
| C23 | E03 | T03.7 | M | ✅ |
| C24 | E04 | T04.1 | M | ✅ |
| C25 | E04 | T04.2, T04.3 | M | ⚠️ |
| C26 | E04 | T04.4 | M | ✅ |
| C27 | E04 | T04.5 | S | ✅ |
| C28 | E05 | T05.1 | M | ⚠️ |
| C29 | E05 | T05.2 part A | L | ⚠️ |
| C30 | E05 | T05.2 part B | L | ⚠️ |
| C31 | E05 | T05.3 PDF | L | ⚠️ |
| C32 | E05 | T05.3 CSV | M | ⚠️ |
| C33 | E05 | T05.4 | M | ⚠️ |
| C34 | E05 | T05.5 | L | ⚠️ |
| C35 | E05 | T05.6 | M | ✅ |
| C36 | E05 | T05.7, T05.8 | L | ✅ |
| C37 | E05 | T05.9–T05.11 | M | ✅ |
| C38 | E05/E06 | T06.1–T06.4 | M | ⚠️ |
| C39 | E06 | T06.5, T06.6 | M | ✅ |
| C40 | E07 | T07.1, T07.2 | M | ✅ |
| C41 | E07 | T07.3 | L | 🔒 |
| C42 | E07 | T07.4 | S | ✅ |
| C43 | E08 | T08.1, T08.2 | M | ⚠️ |
| C44 | E08 | T08.3, T08.4 | M | ⚠️ |
| C45 | E08 | T08.5 | M | ⚠️ |
| C46 | E08 | T08.6, T08.7 | M | ✅ |
| C47 | E08 | T08.8 | M | ✅ |
| C48 | E08 | T08.9 | M | ✅ |
| C49 | E08 | T08.10 | S | ✅ |
| C50 | E08 | T08.11 | L | ⚠️ |
| C51 | E09 | T09.1 | L | ✅ |
| C52 | E09 | T09.2 | M | ✅ |
| C53 | E09 | T09.3 | M | ✅ |
| C54 | E09 | T09.4 | S | ✅ |
| C55 | E10 | T10.1 | M | ✅ |
| C56 | E10 | T10.2 | M | ✅ |
| C57 | E10 | T10.3 | M | 🔒 |
| C58 | E10 | T10.4 | S | ✅ |
| C59 | E11 | T11.1 | M | ✅ |
| C60 | E11 | T11.2 | L | ⚠️ |
| C61 | E11 | T11.3 | M | ✅ |
| C62 | E11 | T11.4 part 1 | L | ⚠️ |
| C63 | E11 | T11.4 part 2 | L | ⚠️ |
| C64 | E12 | T12.1 | L | ✅ |
| C65 | E12 | T12.2 part 1 | L | ⚠️ |
| C66 | E12 | T12.2 part 2 | L | ⚠️ |
| C67 | E12 | T12.3 | M | ✅ |
| C68 | E12 | T12.4 | M | ✅ |
| C69 | E12 | T12.5, T12.6 | M | ✅ |
| C70 | E13 | T13.1 | M | 🔒 |
| C71 | E13 | T13.2 | L | ✅ |
| C72 | E13 | T13.3 | L | ✅ |
| C73 | E13 | T13.4 | L | ⚠️ |
| C74 | E13 | T13.5, T13.6 | M | ✅ |
| C75 | E13 | T13.7, T13.8 | M | ⚠️ |
| C76 | E13 | T13.9, T13.10 | M | ✅ |
| C77 | E14 | T14.1 part 1 | XL | 🔒 |
| C78 | E14 | T14.1 part 2 | XL | 🔒 |
| C79 | E14 | T14.2, T14.3 | M | ⚠️ |
| C80 | E14 | T14.4 | M | ✅ |
| C81 | E15 | T15.1, T15.4, T15.5 | S | ✅ |
| C82 | E15 | T15.2 | M | 🔒 |
| C83 | E15 | T15.3 | S | ✅ |

---


---

## Implementation Detail Files

The full implementation prompts for each chunk are organized by epic:

| File | Epics Covered |
|------|--------------|
| [`14b_epics_01_05.md`](14b_epics_01_05.md) | E01–E05: Infrastructure, Auth, Consent, Documents, OCR Workers |
| [`14c_epics_06_10.md`](14c_epics_06_10.md) | E06–E10: Extraction UI, Tax Profile, Tax Engine, Reports, Frontend Auth |
| [`14d_epics_11_15.md`](14d_epics_11_15.md) | E11–E15: Frontend Docs, Frontend Estimates, Admin, Data Deletion, Security |

---
*Chunk ordering rule: Start at C01. Complete each chunk fully before moving to the next. Save every output file before opening a new session.*
