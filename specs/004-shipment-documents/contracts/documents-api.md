# API Contracts: Shipment Documents

**Feature**: 004-shipment-documents | **Base URL**: `http://localhost:3001` (proxied `/api`)

**Backend source**: `Shipping-Project-V2/production-ready/nestjs-backend/src/modules/document/`

**Extends**: endpointهای موجود documents — [research.md](../research.md#r2-api-design--route-ordering-nestjs)

**Auth**: Bearer JWT + `@TenantScoped()` — **except** `GET /api/verify/:documentId` (public)

---

## GET /api/documents

**Query**:

| Param | Type | Notes |
|-------|------|-------|
| shipmentId | string? | **required for shipment tab** |
| type | DocumentType? | filter |
| page | number | default 1 |
| limit | number | default 20, max 100 |

**Response**:

```json
{
  "items": [DocumentItem],
  "total": 0,
  "page": 1,
  "limit": 20,
  "totalPages": 0
}
```

**DocumentItem** (extended):

```json
{
  "id": "string",
  "shipmentId": "string",
  "type": "BILL_OF_LADING",
  "docNumber": "BL-000001",
  "direction": "EXPORT",
  "language": "FA",
  "status": "DRAFT",
  "data": { "shipper": "...", "consignee": "..." },
  "fileName": "BL-000001.pdf",
  "mimeType": "application/pdf",
  "fileSize": 12345,
  "fileHash": "sha256hex",
  "isWatermarked": false,
  "isLocked": false,
  "supersedesId": null,
  "issuedAt": null,
  "createdAt": "2026-07-06T12:00:00.000Z",
  "updatedAt": "2026-07-06T12:00:00.000Z",
  "_count": { "accessLogs": 0 }
}
```

**Errors**: 401 unauthorized

**Status**: ✅ exists — **extend response** with new fields (B4)

---

## GET /api/documents/:id

**Response**: `DocumentItem` + optional `accessLogs` (last 20) + `shipment.shipmentNo`

**Status**: ✅ exists — extend response

---

## POST /api/documents *(NEW)*

**Purpose**: ایجاد سند ساخت‌یافته در وضعیت DRAFT (بدون فایل PDF تا finalize)

**Request**:

```json
{
  "shipmentId": "string",
  "type": "BILL_OF_LADING",
  "direction": "EXPORT",
  "language": "FA",
  "data": {
    "shipper": "شرکت فرستنده",
    "consignee": "شرکت گیرنده",
    "vesselName": "MSC OSCAR",
    "voyageNo": "V123",
    "portOfLoading": "Bandar Abbas",
    "portOfDischarge": "Jebel Ali",
    "cargoDescription": "General cargo"
  }
}
```

| Field | Required |
|-------|----------|
| shipmentId | yes |
| type | yes — `BILL_OF_LADING` \| `COMMERCIAL_INVOICE` \| `PACKING_LIST` |
| data | yes |
| direction | no (default EXPORT) |
| language | no (default FA) |

**Response**: `201` + `DocumentItem` (`status: DRAFT`, `docNumber: null`)

**Errors**:
- 400 — validation (missing shipper, etc.)
- 404 — shipment not found
- 409 — shipment CLOSED

---

## PATCH /api/documents/:id *(NEW)*

**Purpose**: ویرایش فقط در DRAFT

**Request** (partial):

```json
{
  "direction": "IMPORT",
  "language": "BILINGUAL",
  "data": {
    "cargoDescription": "Updated description"
  }
}
```

**Merge rule**: `data` shallow-merge با existing `data`

**Response**: `200` + `DocumentItem`

**Errors**:
- 404 — not found
- 409 — `status !== DRAFT`

---

## POST /api/documents/:id/finalize *(NEW)*

**Purpose**: قفل سند، تخصیص `docNumber` (**فقط اینجا** — `BL-000001` / `INV-000001` / `PL-000001` per-tenant per-type sequence)، تولید PDF، hash، watermark، `status → FINALIZED`

**Request**: `{}` (empty body acceptable)

**Response**: `200` + `DocumentItem` (FINALIZED, docNumber set, fileHash set)

**Side effects**:
- PDF stored via `StorageService`
- `DocumentAccessLog` action `CREATE` or `FINALIZE`
- QR in PDF points to `/api/verify/{id}`

**Errors**:
- 400 — missing required data fields for PDF template
- 404 — not found
- 409 — already FINALIZED or shipment CLOSED

---

## POST /api/documents/:id/amend *(NEW)*

**Purpose**: ایجاد نسخه جدید DRAFT از سند FINALIZED

**Request** (optional):

```json
{
  "data": { "cargoDescription": "Corrected after finalize" }
}
```

If `data` omitted → copy source `data`.

**Response**: `201` + new `DocumentItem` (DRAFT, `supersedesId` set)

**Side effects**: source document `status → AMENDED`

**Errors**:
- 404 — not found
- 409 — source not FINALIZED

**UI v1**: endpoint implemented — **no ERP button** (future feature)

---

## POST /api/documents/upload *(existing — unchanged)*

**Purpose**: آپلود فایل خارجی (multipart)

**Form fields**: `file`, `shipmentId`, `type`, `docNumber?`

**Response**: `DocumentItem` (legacy — may lack structured `data`)

**Note**: رکوردهای upload `status` از backfill — معمولاً DRAFT مگر watermarked.

---

## POST /api/documents/generate-locked-pdf *(existing — unchanged)*

**Purpose**: `QUOTE_COMMITMENT` locked PDF — خارج از MVP tab اما regression required.

---

## GET /api/documents/:id/download *(existing)*

**Response**: binary PDF + `Content-Disposition: attachment`

**Side effects**: `DocumentAccessLog` DOWNLOAD

**Errors**:
- 400 — integrity check failed (`fileHash` mismatch)

---

## DELETE /api/documents/:id *(existing)*

**Rule**: only if `!isLocked` (DRAFT without finalize)

**Errors**: 400 — locked documents cannot be deleted

---

## GET /api/verify/:documentId *(existing — public)*

**Auth**: none

**Response**:

```json
{
  "documentId": "string",
  "type": "BILL_OF_LADING",
  "docNumber": "BL-000001",
  "sha256": "hex",
  "isWatermarked": true,
  "watermarkType": "ISSUED",
  "issuedAt": "2026-07-06T12:00:00.000Z",
  "issuedBy": "Tenant Legal Name",
  "valid": true
}
```

**Errors**: 404 — invalid document id

**T0**: confirm `valid: true` for finalized MVP documents

---

## Error envelope (standard)

```json
{
  "statusCode": 409,
  "message": "Shipment is closed — cannot create documents",
  "error": "Conflict"
}
```

Frontend: نمایش `message` در error box rose.

---

## UI Contract cross-reference

| testid | API action |
|--------|------------|
| `document-create-btn` | opens form → `POST /documents` |
| `document-form` | `POST` or `PATCH` |
| `document-finalize-btn` | `POST .../finalize` |
| `document-download-btn` | `GET .../download` |
| `document-qr-preview` | client render URL `/api/verify/{id}` |

---

## Frontend client (`src/lib/documents.ts`)

```ts
fetchShipmentDocuments(shipmentId, params?)
createDocument(payload)
updateDocument(id, payload)
finalizeDocument(id)
amendDocument(id, payload?)  // API only v1 — no UI button
downloadDocument(id): Promise<Blob>
deleteDocument(id)
getVerifyUrl(documentId): string
```

Invalidate keys: `['shipment-documents', shipmentId]`, `['shipment', shipmentId]`.
