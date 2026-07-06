# Data Model: Shipment Documents

**Feature**: 004-shipment-documents | **Date**: 2026-07-06

**Extends**: Prisma `Document` در `Shipping-Project-V2/production-ready/prisma/schema.prisma`  
**Related**: [002-shipments data-model](../002-shipments/data-model.md) — `Shipment` relation

---

## Prisma Enums (NEW)

### DocumentDirection

```
IMPORT | EXPORT
```

### DocumentLanguage

```
FA | EN | BILINGUAL
```

### DocumentStatus

```
DRAFT | FINALIZED | AMENDED
```

### DocumentType (existing — MVP subset)

```
BILL_OF_LADING | COMMERCIAL_INVOICE | PACKING_LIST
```

> سایر enum values (`QUOTE_COMMITMENT`, `AWB`, …) بدون تغییر برای سازگاری.

---

## Document (extended model)

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| id | string (cuid) | yes | auto | |
| tenantId | string | yes | | RLS |
| shipmentId | string? | yes* | | *required for MVP create |
| type | DocumentType | yes | | BL / INV / PL |
| docNumber | string? | no | null | set on **finalize** only |
| direction | DocumentDirection? | no | EXPORT | IMPORT/EXPORT |
| language | DocumentLanguage | yes | FA | affects PDF labels |
| status | DocumentStatus | yes | DRAFT | lifecycle |
| data | Json | yes | `{}` | structured form fields |
| filePath | string | conditional | | empty until finalize or upload |
| fileName | string | conditional | | |
| mimeType | string | yes | application/pdf | |
| fileSize | int? | no | | bytes |
| fileHash | string? | no | | SHA-256 after finalize |
| watermarkType | WatermarkType? | no | | set on finalize (ISSUED) |
| watermarkText | string? | no | | user/ip/timestamp |
| isWatermarked | boolean | yes | false | true after finalize |
| supersedesId | string? | no | | amend chain |
| issuedAt | DateTime? | no | | set on finalize |
| createdAt | DateTime | yes | now | |
| updatedAt | DateTime | yes | | |

### API computed fields (not DB)

| Field | Derivation |
|-------|------------|
| isLocked | `status === FINALIZED \|\| status === AMENDED \|\| isWatermarked` |

---

## Migration Backfill Rules

| Legacy condition | `status` | `data` | `issuedAt` |
|------------------|----------|--------|------------|
| `isWatermarked = true` | FINALIZED | `{}` | `createdAt` |
| `watermarkType IS NOT NULL` | FINALIZED | `{}` | `createdAt` |
| filePath set, not watermarked | DRAFT | `{}` | null |
| no filePath | DRAFT | `{}` | null |

`QUOTE_COMMITMENT` rows: unchanged behavior؛ `status=FINALIZED` if locked.

---

## JSON `data` Schemas (by type)

### Common fields (all types)

| Key | Type | Required | Notes |
|-----|------|----------|-------|
| shipper | string | BL: yes | company name + address text |
| consignee | string | BL: yes | |
| notifyParty | string? | no | BL |
| cargoDescription | string | yes | |
| remarks | string? | no | |

### BILL_OF_LADING (`data`)

| Key | Type | Required | Prefill from Shipment |
|-----|------|----------|----------------------|
| vesselName | string | yes | `shipment.vesselName` |
| voyageNo | string? | no | `shipment.voyageNo` |
| portOfLoading | string | yes | `originPort` \|\| `originCity` |
| portOfDischarge | string | yes | `destinationPort` \|\| `destinationCity` |
| freightTerms | string? | no | e.g. PREPAID/COLLECT |
| numberOfOriginals | number? | no | 1–3 |

### COMMERCIAL_INVOICE (`data`)

| Key | Type | Required | Prefill |
|-----|------|----------|---------|
| buyerName | string | yes | `customer.companyName` |
| sellerName | string? | no | tenant legal name |
| currency | string | yes | e.g. USD, IRR |
| totalAmount | number | yes | |
| lineItems | array | yes | `{ description, qty, unitPrice, total }[]` |
| paymentTerms | string? | no | |

### PACKING_LIST (`data`)

| Key | Type | Required | Prefill |
|-----|------|----------|---------|
| packages | number | yes | |
| grossWeightKg | number? | no | |
| netWeightKg | number? | no | |
| volumeCbm | number? | no | |
| packageType | string? | no | e.g. CARTON, PALLET |
| marksAndNumbers | string? | no | |

---

## CreateDocumentPayload (POST /api/documents)

| Field | Type | Required |
|-------|------|----------|
| shipmentId | string | yes |
| type | DocumentType (MVP subset) | yes |
| direction | DocumentDirection? | no (default EXPORT) |
| language | DocumentLanguage? | no (default FA) |
| data | object | yes — validated per type |

**Response**: `DocumentItem` with `status: DRAFT`, `docNumber: null`, no file yet (or placeholder fileName).

---

## UpdateDocumentPayload (PATCH /api/documents/:id)

| Field | Type | Rule |
|-------|------|------|
| direction | DocumentDirection? | DRAFT only |
| language | DocumentLanguage? | DRAFT only |
| data | object | DRAFT only — partial merge |

**Errors**: 409 if `status !== DRAFT`

---

## FinalizeDocumentPayload (POST /api/documents/:id/finalize)

| Field | Type | Required |
|-------|------|----------|
| (body) | empty or `{}` | optional confirm |

**Server actions**:
1. Validate required `data` fields per type
2. **Allocate `docNumber`** via `TenantService.nextSequence` — **only here**, never on create:
   - `BILL_OF_LADING` → `BL-000001` (per-tenant BL sequence)
   - `COMMERCIAL_INVOICE` → `INV-000001`
   - `PACKING_LIST` → `PL-000001`
3. Render PDF via `PdfService`
4. `StorageService.write`
5. Set `fileHash`, `status=FINALIZED`, `isWatermarked=true`, `issuedAt=now`

**docNumber rules**:
- DRAFT rows: `docNumber` MUST remain `null`
- Deleted DRAFTs do not consume sequence numbers
- Sequence: independent per tenant per document type (6-digit suffix)

**Response**: updated `DocumentItem` + optional `downloadUrl` hint

---

## AmendDocumentPayload (POST /api/documents/:id/amend)

| Field | Type | Required |
|-------|------|----------|
| data | object? | no — copy source if omitted |

**Server actions**:
1. Source must be `FINALIZED`
2. Create new `DRAFT` with `supersedesId`
3. Set source `status=AMENDED`

**Response**: new draft `DocumentItem`

---

## DocumentItem (API response shape)

| Field | Type |
|-------|------|
| id | string |
| shipmentId | string |
| type | string |
| docNumber | string \| null |
| direction | string \| null |
| language | string |
| status | DocumentStatus |
| data | object |
| fileName | string \| null |
| mimeType | string |
| fileSize | number \| null |
| fileHash | string \| null |
| isWatermarked | boolean |
| isLocked | boolean |
| supersedesId | string \| null |
| issuedAt | string \| null (ISO) |
| createdAt | string (ISO) |
| updatedAt | string (ISO) |
| _count | `{ accessLogs: number }` |

---

## State Transitions

```text
DRAFT ──finalize──► FINALIZED
DRAFT ──delete──► (removed)
FINALIZED ──amend──► AMENDED (self) + new DRAFT (child)
new DRAFT ──finalize──► FINALIZED
```

---

## Validation Rules (server-side)

### Create / Update

- `shipmentId` must belong to tenant
- `type` in MVP whitelist
- `data` validated with class-validator nested DTO per type
- Shipment `CLOSED` → 409

### Finalize

- All type-specific required `data` fields present
- `status === DRAFT`
- No existing `fileHash` (idempotent retry: if already FINALIZED → 409)

### Amend

- `status === FINALIZED` on source
- Source not already `AMENDED` with open draft child (optional guard — warn if draft exists)

---

## Frontend Types (`src/lib/documents.ts`)

Mirror `DocumentItem` + payload types + label maps:

```ts
DOCUMENT_TYPE_LABELS: Record<DocumentType, string>
DOCUMENT_STATUS_PALETTE: Record<DocumentStatus, StatusPalette>
```

Status badges:
- DRAFT → slate
- FINALIZED → emerald
- AMENDED → yellow

---

## ShipmentDetail extension

`ShipmentDetail.documents[]` در 002 فقط `{ id, type, docNumber }` — پس از implement می‌تواند غنی‌تر شود یا panel مستقل از `GET /documents?shipmentId=` استفاده کند (**Decision**: panel از `GET /documents?shipmentId=` — منبع حقیقت کامل‌تر).
