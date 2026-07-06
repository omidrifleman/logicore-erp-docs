# Research: Shipment Documents

**Feature**: 004-shipment-documents | **Date**: 2026-07-06

**Scope**: گزینه B — backend + frontend full stack

---

## R0: Backend Baseline (pre-change)

**Decision**: ماژول `document/` **پایه محکم** دارد — گسترش می‌شود نه rewrite.

| مورد | وضعیت فعلی |
|------|------------|
| Controller | `document.controller.ts` — list, get, upload, generate-locked-pdf, download, delete |
| Verify | `document-verify.controller.ts` — `GET /api/verify/:documentId` (public) |
| Storage | `StorageService` — local یا S3/MinIO — **بدون تغییر** |
| PDF | `PdfService.generateLockedCommitmentPdf` — pdf-lib + qrcode |
| Prisma `Document` | type, docNumber, filePath, fileHash, watermark, supersedesId — **بدون** direction/language/status/data |
| E2E | `e2e/tests/documents-api.spec.ts` — فقط GET list pagination |

**Gap vs spec**: POST structured create, PATCH, finalize, amend, typed PDF templates.

---

## R1: Prisma Schema Extension

**Decision**: افزودن enumها و فیلدها با default + migration backfill.

```prisma
enum DocumentDirection { IMPORT EXPORT }
enum DocumentLanguage { FA EN BILINGUAL }
enum DocumentStatus { DRAFT FINALIZED AMENDED }
```

| Field | Migration rule |
|-------|----------------|
| `direction` | nullable، default `EXPORT` برای رکورد جدید |
| `language` | default `FA` |
| `status` | `FINALIZED` اگر `isWatermarked = true` OR `watermarkType IS NOT NULL`؛ else `DRAFT` |
| `data` | default `{}` |
| `issuedAt` | `createdAt` برای رکوردهای finalized قدیمی؛ null برای DRAFT |

**Rationale**: رکوردهای `upload` و `QUOTE_COMMITMENT` همچنان filePath دارند؛ `data` خالی مجاز است.

**Alternatives considered**:
- جدول جدا `DocumentDraft` — rejected (duplication، شکستن supersedesId chain)
- Map status فقط از watermark — rejected (کاربر صریحاً status enum خواست)

---

## R2: API Design — Route Ordering (NestJS)

**Decision**: مسیرهای static قبل از `:id`:

```text
GET    /documents
POST   /documents              # NEW — before :id
POST   /documents/upload         # existing
POST   /documents/generate-locked-pdf
GET    /documents/:id
PATCH  /documents/:id            # NEW
POST   /documents/:id/finalize  # NEW
POST   /documents/:id/amend     # NEW
GET    /documents/:id/download
DELETE /documents/:id
```

**Rationale**: جلوگیری از تداخل `upload` با `:id`.

---

## R3: PDF Generation Strategy

**Decision**: **گسترش `PdfService`** با سه متد template + shared `drawQrAndFooter()`.

| Template | Method | Pages |
|----------|--------|-------|
| Bill of Lading | `generateBillOfLadingPdf(params)` | 1–2 |
| Commercial Invoice | `generateCommercialInvoicePdf(params)` | 1 |
| Packing List | `generatePackingListPdf(params)` | 1 |

**Stack**: `pdf-lib` + `qrcode` (همان commitment) — **بدون** Playwright/Handlebars در v1.

**Rationale**:
- وابستگی zero اضافه برای headless browser
- الگوی QR/hash/watermark از `generateLockedCommitmentPdf` قابل استخراج است
- StorageService بدون تغییر — `write(tenantId, fileName, buffer)`

**Alternatives considered**:

| Alternative | Why rejected (v1) |
|-------------|-------------------|
| Playwright + Handlebars | سنگین CI، نیاز Chrome، کندتر؛ مناسب فاز ۲ layout پیچیده |
| HTML → puppeteer | همان مشکلات Playwright |
| فقط upload PDF دستی | گزینه A — کاربر گزینه B را رد کرد |

### R3.1 — Persian / Bilingual text

**Decision**: embed فونت **Vazirmatn Regular** (فایل `.ttf` در `nestjs-backend/assets/fonts/`) برای `language: FA | BILINGUAL`.

**Fallback**: اگر embed شکست خورد — متن فارسی در PDF به انگلیسی transliterate نشود؛ خطای finalize با پیام واضح.

**Alternatives**: Helvetica-only — rejected برای SC فارسی ERP.

---

## R4: docNumber Generation

**Decision**: سرور **فقط در `finalize`** تولید می‌کند — `POST`/`PATCH` هرگز `docNumber` نمی‌پذیرند؛ DRAFT همیشه `null`.

**Format** (الگوی `SHP-000037`):

| type | prefix | example |
|------|--------|---------|
| `BILL_OF_LADING` | `BL` | `BL-000001` |
| `COMMERCIAL_INVOICE` | `INV` | `INV-000001` |
| `PACKING_LIST` | `PL` | `PL-000001` |

- **Sequence scope**: per-tenant, per-type (مستقل — BL و INV شمارنده جدا)
- **Padding**: 6 digits
- **Mechanism**: `TenantService.nextSequence()` با `sequenceType` = `DOCUMENT_BL` | `DOCUMENT_INV` | `DOCUMENT_PL`
- **Heal**: extend `syncSequenceCounter` — `findFirst` on `Document.docNumber` با `startsWith: '${prefix}-'`

**Rationale**: شماره gapless tenant-scoped؛ حذف DRAFT شماره را مصرف نمی‌کند.

**Alternatives considered**:
- per-shipment counter (`BL-SHP-000001-001`) — rejected by user (۱۴۰۵/۰۴/۱۶)
- count query بدون TenantSequence — rejected (race under concurrency)

---

## R5: Amend Flow

**Decision**: `POST /api/documents/:id/amend` روی `FINALIZED` only.

1. Validate source `status === FINALIZED`
2. Create new row: `status=DRAFT`, `data` = copy source `data`, `supersedesId` = source.id, same shipmentId/type/direction/language
3. Update source: `status=AMENDED`
4. Return new draft document

**UI v1**: endpoint + E2E test — **بدون** دکمه amend در ERP (spec out of scope).

---

## R6: isLocked / Delete / Download compatibility

**Decision**:

| Operation | DRAFT | FINALIZED | AMENDED |
|-----------|-------|-----------|---------|
| PATCH | ✅ | ❌ | ❌ |
| DELETE | ✅ (if no file or draft) | ❌ | ❌ |
| DOWNLOAD | ✅ if file exists | ✅ | ✅ |
| finalize | ✅ | ❌ | ❌ |
| amend | ❌ | ✅ | ❌ |

Legacy `isLocked` در JSON response: `status === 'FINALIZED' || status === 'AMENDED' || isWatermarked`.

---

## R7: Frontend Port Reference

**Decision**: پورت از `Shipping-Project-V2/src/lib/api.ts` (`documentApi`) به `Shipping/src/lib/documents.ts` با افزودن create/update/finalize/amend.

**Hooks**: الگوی `use-shipment-legs.ts` — queryKey `['shipment-documents', shipmentId]`.

**QR UI**: dependency `qrcode.react` (سبک، بدون canvas manual).

**Alternatives**: generate QR data URL از backend — rejected (verify URL ثابت است).

---

## R8: CLOSED Shipment Guard

**Decision**: `POST /documents` و `PATCH` و `finalize` روی shipment با `status === CLOSED` → **409 Conflict** با پیام فارسی.

**Rationale**: spec edge case — پیش‌فرض ممنوع.

---

## R9: Verify Endpoint Extension

**Decision**: `app.verify_document` SQL function — بررسی T0 که `status` جدید را در پاسخ `is_valid` لحاظ کند؛ در صورت نیاز migration SQL در `prisma/20-verify-function.sql`.

**Public response** (unchanged shape): `documentId`, `type`, `docNumber`, `sha256`, `issuedAt`, `valid` — بدون customer name.

---

## T0 Probe Checklist (پس از B5 — قبل از Frontend)

1. `npx prisma migrate deploy` — schema جدید بدون خطا
2. `POST /api/documents` — BL draft با `data.shipper` → 201، `status: DRAFT`
3. `PATCH /api/documents/:id` — update `data` → 200
4. `PATCH` روی FINALIZED → 400/409
5. `POST /api/documents/:id/finalize` → `status: FINALIZED`, `docNumber` set, `fileHash` set, PDF downloadable
6. `GET /api/verify/:id` → `valid: true`
7. `POST /api/documents/:id/amend` → new DRAFT + old `AMENDED`
8. `POST /api/documents/upload` — همچنان کار می‌کند (regression)
9. `POST /api/documents/generate-locked-pdf` — regression commitment flow

---

## R10: Visual QA

**Decision**: capture جدید `13-shipment-documents-desktop.png` / mobile در Polish — فعلاً بدون PNG مرجع.

**Rationale**: placeholder فعلی — baseline از detail tab موجود.
