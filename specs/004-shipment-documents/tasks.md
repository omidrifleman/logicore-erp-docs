# Tasks: Shipment Documents (Document Management)

**Status**: Ready for implementation — B1 complete (T001–T004 ✅)

**Input**: Design documents from `/specs/004-shipment-documents/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, `002-shipments` ✅, `003-shipment-wizard` ✅

**Scope**: گزینه B — backend (`logicore-erp-backend`) + frontend (`logicore-erp-frontend`)

**Tests**: T0 live probe + E2E `documents-api.spec.ts` + manual quickstart + Visual QA

**Organization**: Backend phases B1–B5 → T0 → Frontend F1–F6 → Polish

## Format: `[ID] [P?] [Story] Description`

**Backend paths**: `Shipping-Project-V2/production-ready/...`  
**Frontend paths**: `Shipping/src/...`

---

## Phase B1: Prisma Schema & Migration

**Repo**: Backend | **Blocker for**: B2+

- [x] T001 Add enums `DocumentDirection`, `DocumentLanguage`, `DocumentStatus` to `production-ready/prisma/schema.prisma`
- [x] T002 Add fields `direction`, `language`, `status`, `data` (Json), `issuedAt` to `Document` model — defaults: `EXPORT`, `FA`, `DRAFT`, `{}`, null; `docNumber` remains nullable (null until finalize)
- [x] T003 Create migration `production-ready/prisma/migrations/*_documents_v2/` — backfill: watermarked→`FINALIZED`+`issuedAt`; legacy upload→`DRAFT`; `data={}`
- [x] T004 Run `npx prisma migrate deploy` + `npx prisma generate` in `production-ready/nestjs-backend` — verify compiles

**Checkpoint**: Schema deployed locally without breaking existing Document rows

---

## Phase B2: DTOs & Document Service Lifecycle

**Repo**: Backend | **Depends on**: B1

- [ ] T005 Create `nestjs-backend/src/modules/document/dto/create-document.dto.ts` — `shipmentId`, `type` (BL|INV|PL), `direction?`, `language?`, `data` with nested validators per type
- [ ] T006 Create `nestjs-backend/src/modules/document/dto/update-document.dto.ts` — partial `direction`, `language`, `data` merge
- [ ] T007 Create `nestjs-backend/src/modules/document/dto/amend-document.dto.ts` — optional `data` override (API-only v1)
- [ ] T008 Implement `DocumentService.create()` in `document.service.ts` — `status=DRAFT`, `docNumber=null`, reject if shipment `CLOSED`
- [ ] T009 Implement `DocumentService.update()` — only `status===DRAFT`; shallow-merge `data`; 409 otherwise
- [ ] T010 Implement `DocumentService.amend()` — source `FINALIZED`→`AMENDED`, new `DRAFT` with `supersedesId` (**API-only — no ERP UI in v1**)
- [ ] T011 Extend `tenant.service.ts` `syncSequenceCounter` — branches for `DOCUMENT_BL`/`DOCUMENT_INV`/`DOCUMENT_PL` healing from `Document.docNumber` prefix
- [ ] T012 Add `allocateDocNumber(tx, tenantId, type)` in `document.service.ts` — maps type→prefix (`BL`/`INV`/`PL`) + `nextSequence`; **callable only from finalize**

**Checkpoint**: create/patch/amend unit-testable via service (no PDF yet)

---

## Phase B3: PDF Templates (pdf-lib)

**Repo**: Backend | **Depends on**: B2

- [ ] T013 Add `nestjs-backend/assets/fonts/Vazirmatn-Regular.ttf` for FA/BILINGUAL PDF embed
- [ ] T014 Refactor `pdf.service.ts` — shared `drawHeaderFooterQrWatermark()` extracted from commitment PDF
- [ ] T015 [P] Implement `generateBillOfLadingPdf()` in `pdf.service.ts`
- [ ] T016 [P] Implement `generateCommercialInvoicePdf()` in `pdf.service.ts`
- [ ] T017 [P] Implement `generatePackingListPdf()` in `pdf.service.ts`

**Checkpoint**: Each template returns `{ buffer, hash, watermarkText }` + QR → `/api/verify/{id}`

---

## Phase B4: Finalize, Controller & API Responses

**Repo**: Backend | **Depends on**: B3

- [ ] T018 Implement `DocumentService.finalize()` in `document.service.ts` — validate `data` → `allocateDocNumber()` → PDF → `StorageService.write` → set `fileHash`, `status=FINALIZED`, `isWatermarked`, `issuedAt`; **docNumber assigned ONLY here** (`BL-000001` pattern)
- [ ] T019 Extend `document.controller.ts` — `POST /documents`, `PATCH /:id`, `POST /:id/finalize`, `POST /:id/amend` (static routes before `:id` conflict; preserve upload/generate-locked-pdf)
- [ ] T020 Extend `findAll`/`findOne` in `document.service.ts` — return `direction`, `language`, `status`, `data`; `isLocked` from status
- [ ] T021 Update `prisma/20-verify-function.sql` (if needed) so `app.verify_document` reflects finalized MVP documents
- [ ] T022 Verify `remove()` blocks `FINALIZED`/`AMENDED` and regression on existing upload/download/delete DRAFT

**Checkpoint**: Full backend API ready for T0

---

## Phase B5: Backend E2E

**Repo**: Backend | **Depends on**: B4

- [ ] T023 Extend `e2e/tests/documents-api.spec.ts` — create BL DRAFT → patch → finalize → assert `docNumber` matches `/^BL-\d{6}$/` → verify `valid:true`
- [ ] T024 [P] E2E amend — `POST .../amend` on FINALIZED → new DRAFT + source `AMENDED` (no UI)
- [ ] T025 [P] E2E regression — `POST /documents/upload` + `POST /documents/generate-locked-pdf` still succeed

**Checkpoint**: Playwright documents suite green → proceed T0

---

## Phase 0: Backend Live Verification ⚠️ BLOCKER

**Purpose**: تأیید endpointهای جدید قبل از هر کار Frontend

**⚠️ CRITICAL**: No F* tasks until T0 pass + user sign-off

- [ ] T0 Backend Live Verification — per `quickstart.md` T0 Smoke: migration applied؛ `POST/PATCH/finalize` BL؛ `docNumber` like `BL-000001` only after finalize؛ DRAFT has `docNumber:null`؛ `GET /verify/:id` → `valid:true`؛ amend API smoke (optional); upload/commitment regression

**Checkpoint**: User confirms T0 → Phase F1

---

## Phase F1: Frontend Setup

**Repo**: Frontend | **Depends on**: T0 ✅

- [ ] T026 Create `src/lib/documents.ts` — types, `fetchShipmentDocuments`, `createDocument`, `updateDocument`, `finalizeDocument`, `downloadDocument`, `deleteDocument`, `getVerifyUrl` (port from `Shipping-Project-V2/src/lib/api.ts` + new endpoints)
- [ ] T027 Create `src/hooks/use-shipment-documents.ts` — `useShipmentDocuments`, `useCreateDocument`, `useUpdateDocument`, `useFinalizeDocument`, `useDownloadDocument`; invalidate `['shipment-documents', shipmentId]`
- [ ] T028 Add `qrcode.react` to `package.json` for QR preview (US5)

**Checkpoint**: API client + hooks compile

---

## Phase F2: User Story 1 — لیست اسناد (Priority: P1) 🎯 MVP

**Goal**: جایگزینی placeholder تب اسناد با لیست واقعی

**Independent Test**: باز کردن تب اسناد → لیست/empty/loading/error

- [ ] T029 [US1] Create `src/components/erp/shipment-documents-panel.tsx` — `data-testid=shipment-documents-panel`, glass card shell
- [ ] T030 [US1] Implement `document-list` + `document-row-{id}` rows — type label, `docNumber` (or «—» for DRAFT), status badge, `createdAt`
- [ ] T031 [US1] Wire `shipment-detail.tsx` — replace placeholder with `<ShipmentDocumentsPanel shipmentId={...} shipment={...} />`
- [ ] T032 [US1] Loading (`Loader2` amber), error rose box, empty state (`FileText` + `document-create-btn`)
- [ ] T033 [US1] Status badges — DRAFT slate, FINALIZED emerald, AMENDED yellow per `statusPalettes`

**Checkpoint**: US1 independently testable

---

## Phase F3: User Story 2 — ایجاد/ویرایش سند (Priority: P2)

**Goal**: فرم پویا BL / Invoice / Packing List با prefill

**Independent Test**: create DRAFT → appears in list with `docNumber` null

- [ ] T034 [US2] Document modal — `document-form`, `document-type-select`, `document-create-btn` trigger
- [ ] T035 [US2] `BILL_OF_LADING` fields + prefill from shipment (shipper/consignee, vessel, voyage, ports, cargo)
- [ ] T036 [US2] `COMMERCIAL_INVOICE` fields + prefill (buyer, currency, lineItems)
- [ ] T037 [US2] `PACKING_LIST` fields (packages, weights, volume)
- [ ] T038 [US2] Wire `useCreateDocument` + `useUpdateDocument` — edit existing DRAFT row
- [ ] T039 [US2] Client validation فارسی — required fields per type before submit
- [ ] T040 [US2] Non-blocking warning when duplicate same-type doc exists on shipment

**Checkpoint**: Three document types creatable as DRAFT

---

## Phase F4: User Story 3 — Finalize (Priority: P3)

**Goal**: نهایی‌سازی → `docNumber` + PDF + قفل

**Independent Test**: finalize DRAFT BL → `BL-000001`, status FINALIZED, no edit

- [ ] T041 [US3] `document-finalize-btn` on DRAFT rows + confirm dialog
- [ ] T042 [US3] Wire `useFinalizeDocument` — display allocated `docNumber` after success
- [ ] T043 [US3] Disable form edit + delete on FINALIZED/AMENDED — message «از amend استفاده کنید» (amend UI خارج از scope)
- [ ] T044 [US3] Surface 409 when shipment `CLOSED` on create/finalize

**Checkpoint**: Full create→finalize path works in UI

---

## Phase F5: User Story 4 — دانلود PDF (Priority: P4)

**Goal**: دانلود فایل نهایی

**Independent Test**: download FINALIZED doc → valid PDF file

- [ ] T045 [US4] `document-download-btn` — blob download via `downloadDocument`, filename from `docNumber` or `fileName`
- [ ] T046 [US4] Show rose error on integrity failure (`fileHash` mismatch message)

**Checkpoint**: Download audited via backend access log

---

## Phase F6: User Story 5 — QR / Verify (Priority: P5)

**Goal**: نمایش QR و لینک verify

**Independent Test**: QR scans to `/api/verify/{id}` with `valid:true`

- [ ] T047 [US5] `document-qr-preview` — `qrcode.react` rendering verify URL for FINALIZED docs
- [ ] T048 [US5] Optional `document-detail-drawer` — show `fileHash`, `issuedAt`, verify link (no sensitive data)

**Checkpoint**: US5 complete — full spec happy path

---

## Phase P: Polish & Validation

**Purpose**: Cross-cutting QA before merge

- [ ] T049 [P] Validate all UI contract `data-testid`s from `contracts/documents-api.md`
- [ ] T050 [P] Run `quickstart.md` manual checklist US1–US5 + edge cases (CLOSED, delete DRAFT)
- [ ] T051 [P] Add `shipment-documents` + mobile keys to `design-reference/visual-qa.mjs` + capture `13-shipment-documents-desktop.png`, `14-shipment-documents-mobile.png`
- [ ] T052 [P] Mobile smoke — documents tab usable on Pixel 5 viewport
- [ ] T053 [P] `npm run build` (frontend) + backend `npm run build` — zero errors
- [ ] T054 Update `quickstart.md` Notes — docNumber format, amend API-only, T0 results

---

## Dependencies & Execution Order

```text
B1 (T001–T004)
  → B2 (T005–T012)
    → B3 (T013–T017)
      → B4 (T018–T022)
        → B5 (T023–T025)
          → T0 ⚠️
            → F1 (T026–T028)
              → F2/US1 (T029–T033) 🎯 MVP
              → F3/US2 (T034–T040)
              → F4/US3 (T041–T044)
              → F5/US4 (T045–T046)
              → F6/US5 (T047–T048)
                → P (T049–T054)
```

### User Story Dependencies

```text
US1 (list) → US2 (create) → US3 (finalize) → US4 (download) → US5 (QR)
Amend: backend API only (T010, T024) — no frontend tasks in v1
```

---

## Parallel Opportunities

| Phase | Parallel tasks |
|-------|----------------|
| B3 | T015 + T016 + T017 (separate PDF methods) |
| B5 | T024 + T025 after T023 |
| F1 | T026 + T028 after T0 |
| P | T049–T053 all parallel after F6 |

---

## MVP Scope

**Minimum shippable increment**: `T0` (after B1–B5) + `T026–T028` + `T029–T033` (US1 list only) — *or* continue through US3 for business value.

**Recommended MVP for demo**: B1–B5 → T0 → F1 → US1–US3 (list + create + finalize with `BL-000001`).

---

## Task Count Summary

| Phase | Tasks | IDs |
|-------|-------|-----|
| **B1** Schema | 4 | T001–T004 |
| **B2** Service/DTOs | 8 | T005–T012 |
| **B3** PDF | 5 | T013–T017 |
| **B4** Finalize/Controller | 5 | T018–T022 |
| **B5** E2E | 3 | T023–T025 |
| **T0** Live verification | 1 | T0 |
| **F1** Frontend setup | 3 | T026–T028 |
| **F2** US1 List | 5 | T029–T033 |
| **F3** US2 Create | 7 | T034–T040 |
| **F4** US3 Finalize | 4 | T041–T044 |
| **F5** US4 Download | 2 | T045–T046 |
| **F6** US5 QR | 2 | T047–T048 |
| **P** Polish | 6 | T049–T054 |
| **Total** | **55** | T0 + T001–T054 |

| User Story | Task count |
|------------|------------|
| US1 | 5 |
| US2 | 7 |
| US3 | 4 |
| US4 | 2 |
| US5 | 2 |
| Backend only | 25 + T0 |
| Polish | 6 |

---

## Notes

- **docNumber**: `BL-000001` / `INV-000001` / `PL-000001` — per-tenant per-type via `TenantService.nextSequence`; **only on finalize** (T012, T018)
- **amend**: API in T010/T019/T024 — **no UI button** in v1 (confirmed)
- **Dual-repo**: commit backend first; tag `logicore-erp-backend` then frontend `v0.3.0`
- Backend path alias: `Shipping-Project-V2/production-ready/nestjs-backend`
