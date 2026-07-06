# Implementation Plan: Shipment Documents (Document Management)

**Branch**: `004-shipment-documents` | **Date**: 2026-07-06 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/004-shipment-documents/spec.md`

**Scope decision**: **گزینه B** — توسعه کامل بک‌اند + فرانت‌اند در همین فیچر (نه UI-only روی API موجود).

## Summary

جایگزینی placeholder تب «اسناد» در `shipment-detail.tsx` با مدیریت کامل اسناد حمل: ایجاد پیش‌نویس ساخت‌یافته (BL / Invoice / Packing List)، ویرایش در DRAFT، نهایی‌سازی با تولید PDF + hash + QR، دانلود، و نمایش QR/verify. **پیش‌نیاز:** گسترش Prisma `Document`، endpointهای جدید در `nestjs-backend`، و گسترش `PdfService` برای سه قالب MVP — هم‌سطح endpointهای موجود (`upload`, `download`, `generate-locked-pdf`, `verify`).

**Backend repo**: `Shipping-Project-V2/production-ready/nestjs-backend` (+ `production-ready/prisma/schema.prisma`)  
**Frontend repo**: `Shipping/`

---

## Technical Context

**Language/Version**: TypeScript 5.x — NestJS (backend), React 19 + Next.js 15 (frontend)

**Primary Dependencies**:
- Backend: Prisma, `pdf-lib`, `qrcode`, `@aws-sdk/client-s3` (موجود)
- Frontend: TanStack Query 5, framer-motion, shadcn/ui, `qrcode.react` (جدید برای preview UI)

**Storage**: PostgreSQL (Prisma) + `StorageService` (local یا MinIO/S3 — بدون تغییر لایه storage)

**Testing**: T0 live API probe؛ E2E گسترش `e2e/tests/documents-api.spec.ts`؛ manual quickstart؛ Visual QA capture تب documents (Polish)

**Target Platform**: ERP desktop + mobile (تب جزئیات محموله)

**Project Type**: Dual-repo — backend `logicore-erp-backend` + frontend `logicore-erp-frontend`

**Performance Goals**: لیست اسناد <5s؛ finalize PDF <10s برای سند تک‌صفحه

**Constraints**:
- Migration **backward-compatible** — رکوردهای `upload` / `QUOTE_COMMITMENT` موجود نباید بشکنند
- endpointهای موجود حفظ شوند
- Design System + RTL (constitution)
- محموله `CLOSED` → ایجاد/ویرایش سند ممنوع (409)

**Scale/Scope**: 3 نوع سند MVP؛ 1 تب UI؛ ~5 endpoint جدید + 3 قالب PDF

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Tech Stack | ✅ PASS | Next.js + Tailwind + shadcn + framer + Vazirmatn |
| II. RTL Persian | ✅ PASS | تب و فرم RTL؛ فیلدهای LTR (شماره سند، hash) با `dir="ltr"` |
| III. Design System | ✅ PASS | glass panel، erpTokens، status badge برای DRAFT/FINALIZED/AMENDED |
| IV. Component Reuse | ✅ PASS | الگوی `shipment-legs-panel` + hooks مشابه `use-shipment-legs` |
| V. Async States | ✅ PASS | لیست/فرم/finalize — Loader/Error/Empty |
| VI. Simplicity | ✅ PASS | یک پنل `shipment-documents-panel`؛ PdfService extended نه microservice جدید |

**Post-design re-check**: ✅ All gates pass.

**Cross-repo note**: تغییرات بک‌اند در `logicore-erp-backend` commit/tag جدا از فرانت (`v0.3.0` پیشنهادی پس از merge).

---

## Project Structure

### Documentation (this feature)

```text
specs/004-shipment-documents/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── documents-api.md
├── checklists/
│   └── requirements.md
└── tasks.md              # (/speckit-tasks — بعداً)
```

### Source Code — Backend (`Shipping-Project-V2`)

```text
production-ready/
├── prisma/
│   ├── schema.prisma                    # MODIFY — enums + Document fields
│   └── migrations/YYYYMMDD_documents_v2/  # NEW — backward-compatible
└── nestjs-backend/src/modules/document/
    ├── document.controller.ts           # MODIFY — POST, PATCH, finalize, amend
    ├── document.service.ts              # MODIFY — CRUD lifecycle + amend chain
    ├── pdf.service.ts                   # MODIFY — BL / Invoice / PL templates
    ├── storage.service.ts               # unchanged
    ├── document-verify.controller.ts    # unchanged route; verify SQL may extend
    └── dto/
        ├── create-document.dto.ts       # NEW
        ├── update-document.dto.ts       # NEW
        ├── finalize-document.dto.ts     # NEW (optional overrides)
        └── amend-document.dto.ts        # NEW
e2e/tests/
└── documents-api.spec.ts                # EXTEND — create/finalize/amend flows
```

### Source Code — Frontend (`Shipping`)

```text
src/
├── components/erp/
│   ├── shipment-detail.tsx              # MODIFY — wire documents tab
│   └── shipment-documents-panel.tsx     # NEW — list + form + finalize + QR
├── hooks/
│   └── use-shipment-documents.ts        # NEW — queries + mutations
└── lib/
    └── documents.ts                     # NEW — types + API client
```

**Structure Decision**: بک‌اند و فرانت در repoهای جدا؛ tasks باید **Backend phases قبل از Frontend** را رعایت کند.

---

## Phase Overview

| Phase | Repo | Scope | Blocker |
|-------|------|--------|---------|
| **B1** | Backend | Prisma schema + migration + data backfill | — |
| **B2** | Backend | DTOs + `DocumentService` lifecycle | B1 |
| **B3** | Backend | `PdfService` templates (BL, Invoice, PL) + QR/hash | B2 |
| **B4** | Backend | Controller routes + extend list/detail responses | B3 |
| **B5** | Backend | E2E tests + `verify` SQL compatibility | B4 |
| **0** | Both | **T0 Live Verification** — probe endpoints جدید | B5 |
| **F1** | Frontend | `documents.ts` + `use-shipment-documents.ts` | T0 ✅ |
| **F2** | Frontend | US1 — لیست اسناد در تب | F1 |
| **F3** | Frontend | US2 — ایجاد/ویرایش پیش‌نویس | F1 |
| **F4** | Frontend | US3 — finalize + confirm dialog | F1 |
| **F5** | Frontend | US4 — download PDF | F1 |
| **F6** | Frontend | US5 — QR preview + verify link | F1 |
| **P** | Both | Polish — testids, visual QA, quickstart sign-off | F2–F6 |

### Task breakdown principle (برای `/speckit-tasks`)

```
Phase B1–B5  → Backend (جدا، با prefix B یا Phase B در tasks.md)
Phase 0      → T0 (پس از B5، قبل از هر F*)
Phase F1–F6  → Frontend (وابسته به T0)
Phase P      → Polish
```

---

## Backend Design Summary

### Schema additions (Prisma)

به مدل `Document` اضافه می‌شود — جزئیات در [data-model.md](./data-model.md):

| Field | Type | Default | Backfill |
|-------|------|---------|----------|
| `direction` | `DocumentDirection?` | `EXPORT` | null OK |
| `language` | `DocumentLanguage` | `FA` | `FA` |
| `status` | `DocumentStatus` | `DRAFT` | `FINALIZED` if `isWatermarked`; else `DRAFT` |
| `data` | `Json` | `{}` | `{}` for legacy file-only rows |
| `issuedAt` | `DateTime?` | null | `createdAt` if was locked |

`isLocked` در API response: `status === FINALIZED || status === AMENDED` (یا `isWatermarked` برای سازگاری عقب‌رو).

### New endpoints (هم‌سطح موجود — نه جایگزین)

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/api/documents` | ایجاد پیش‌نویس با `data` JSON |
| `PATCH` | `/api/documents/:id` | ویرایش فقط `DRAFT` |
| `POST` | `/api/documents/:id/finalize` | قفل + `docNumber` + PDF + hash |
| `POST` | `/api/documents/:id/amend` | نسخه جدید DRAFT + `supersedesId`؛ قبلی → `AMENDED` |

**Existing (unchanged)**:
`GET /api/documents`, `GET /api/documents/:id`, `POST /api/documents/upload`, `POST /api/documents/generate-locked-pdf`, `GET /api/documents/:id/download`, `DELETE /api/documents/:id`, `GET /api/verify/:documentId`

### PDF generation

**Decision**: گسترش `PdfService` فعلی (`pdf-lib` + `qrcode`) — یک متد per type + shared helper برای header/footer/QR/watermark.

**Not in scope**: Playwright + Handlebars (فاز بعدی اگر layout پیچیده‌تر شد).

هر قالب MVP: QR → `{NEXTAUTH_URL}/api/verify/{documentId}`؛ `fileHash` SHA-256 پس از serialize.

### docNumber on finalize

**Decision**: تخصیص **فقط در `finalize`** — DRAFT همیشه `docNumber: null` (اسناد حذف‌شده شماره هدر نمی‌برند).

**Format** (مشابه `SHP-000037`):

| type | prefix | نمونه |
|------|--------|-------|
| `BILL_OF_LADING` | `BL` | `BL-000001` |
| `COMMERCIAL_INVOICE` | `INV` | `INV-000001` |
| `PACKING_LIST` | `PL` | `PL-000001` |

- **Scope شمارنده**: per-tenant + per-type — هر نوع sequence مستقل (`BL-000001`, `BL-000002` جدا از `INV-000001`)
- **Padding**: ۶ رقم (`000001`)
- **Implementation**: `TenantService.nextSequence(tx, tenantId, sequenceType, prefix)` — `sequenceType`: `DOCUMENT_BL` | `DOCUMENT_INV` | `DOCUMENT_PL`؛ گسترش `syncSequenceCounter` برای heal از `Document.docNumber`

---

## Frontend Design Summary

### Component: `shipment-documents-panel.tsx`

| بخش | رفتار |
|-----|--------|
| لیست | `document-list` — نوع، شماره، status badge، تاریخ، actions |
| خالی | `FileText` + `document-create-btn` |
| فرم | مودال `document-form` — `document-type-select` + فیلدهای پویا از `data` schema |
| Prefill | از `useShipmentDetail` — customer, vessel, voyage, ports, cargo |
| Finalize | `document-finalize-btn` + confirm dialog — فقط DRAFT |
| Download | `document-download-btn` — blob download |
| QR | `document-qr-preview` — `qrcode.react` + لینک verify |

### Hooks (`use-shipment-documents.ts`)

الگوی `use-shipment-legs.ts`:
- `useShipmentDocuments(shipmentId)`
- `useCreateDocument(shipmentId)`
- `useUpdateDocument(shipmentId)`
- `useFinalizeDocument(shipmentId)`
- `useDownloadDocument()` — mutation/blob helper

Invalidate: `['shipment-documents', shipmentId]`, `['shipment', shipmentId]`, `['shipments']`.

### Amend UI

**v1 frontend**: دکمه amend **خارج از scope UI** (spec) — endpoint بک‌اند + contract برای آینده؛ optional task در Polish اگر زمان بود.

---

## Document Lifecycle (state machine)

```text
POST /documents ──► DRAFT ──PATCH──► DRAFT
                        │
                        ▼ finalize
                   FINALIZED ──amend──► new DRAFT (supersedesId)
                        │                    │
                        ▼                    ▼ finalize
                    AMENDED ◄── status      FINALIZED
                    (previous)
```

---

## Routing

بدون route جدید — همه UI داخل `/shipments/[id]` تب `documents`.

`erp-shell.tsx` لینک `/documents` global (nav) — **خارج از scope v1**؛ فقط تب shipment detail.

---

## Complexity Tracking

| Item | Justification |
|------|----------------|
| Dual-repo delivery | بک‌اند API ناقص بود — گزینه B تأیید کاربر |
| PdfService 3 templates | MVP BL/Invoice/PL — بدون Playwright |
| `qrcode.react` dependency | QR preview در UI (US5) |
| Backend amend endpoint بدون UI | contract آینده + integrity chain |

No constitution violations.

---

## Dependencies

- ✅ `002-shipments` — `shipment-detail.tsx` + detail API
- ✅ `003-shipment-wizard` — محموله‌های DRAFT برای تست
- ⏳ Backend B1–B5 قبل از Frontend F1
- ⏳ T0 پس از deploy migration محلی

---

## Next Steps

1. User reviews plan + design artifacts
2. `/speckit-tasks` پس از تأیید
3. اجرا: **B1→B5 → T0 → F1→F6 → Polish**
4. Tag پیشنهادی: backend `v0.2.0` (یا patch) + frontend `v0.3.0` پس از merge
