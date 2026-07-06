# Quickstart: Shipment Documents Validation

**Feature**: 004-shipment-documents

## Prerequisites

1. Docker Postgres — `Shipping-Project-V2/e2e/docker-compose.yml`
2. NestJS backend `:3001` — **با migration جدید Document** (Phase B1)
3. Frontend `:3000` — `BACKEND_URL=http://localhost:3001`
4. Login: `maryam@logicore.com` / `maryam123` → workspace «سیر راه آبی»

```powershell
# Backend (from Shipping-Project-V2/production-ready/nestjs-backend)
npx prisma migrate deploy
npm run start:dev

# Frontend (from Shipping)
npm run dev
```

## T0 Smoke (after Backend B5 — before Frontend F1)

```powershell
# Obtain TOKEN from login cookie or API

$SHIPMENT_ID = "<existing-shipment-uuid>"
$BASE = "http://localhost:3001"

# 1. Create BL draft
curl -X POST "$BASE/api/documents" `
  -H "Authorization: Bearer $TOKEN" `
  -H "Content-Type: application/json" `
  -d "{\"shipmentId\":\"$SHIPMENT_ID\",\"type\":\"BILL_OF_LADING\",\"language\":\"FA\",\"data\":{\"shipper\":\"Shipper Co\",\"consignee\":\"Consignee Co\",\"vesselName\":\"TEST VESSEL\",\"portOfLoading\":\"BND\",\"portOfDischarge\":\"DXB\",\"cargoDescription\":\"Test cargo\"}}"

# Save DOCUMENT_ID from response

# 2. Patch draft
curl -X PATCH "$BASE/api/documents/$DOCUMENT_ID" `
  -H "Authorization: Bearer $TOKEN" `
  -H "Content-Type: application/json" `
  -d "{\"data\":{\"remarks\":\"T0 test\"}}"

# 3. Finalize
curl -X POST "$BASE/api/documents/$DOCUMENT_ID/finalize" `
  -H "Authorization: Bearer $TOKEN" `
  -H "Content-Type: application/json" `
  -d "{}"

# Expect: status FINALIZED, docNumber matches BL-000001 pattern (6-digit suffix, assigned ONLY on finalize)

# 4. Download
curl -o test-bl.pdf "$BASE/api/documents/$DOCUMENT_ID/download" `
  -H "Authorization: Bearer $TOKEN"

# 5. Public verify
curl "$BASE/api/verify/$DOCUMENT_ID"
# Expect: valid: true, docNumber, sha256

# 6. Regression — list
curl "$BASE/api/documents?shipmentId=$SHIPMENT_ID&page=1&limit=20" `
  -H "Authorization: Bearer $TOKEN"

# 7. Regression — upload still works
# curl -F "file=@test.pdf" -F "shipmentId=$SHIPMENT_ID" -F "type=PACKING_LIST" ...
```

**Checkpoint**: کاربر تأیید T0 → شروع Frontend F1

---

## Manual Validation (Frontend)

### US1 — Document list

1. `/shipments` → open shipment with seed data
2. Click `shipment-tab-documents`
3. `shipment-documents-panel` visible
4. Empty state OR `document-list` rows with status badges
5. Loading spinner on slow network

### US2 — Create document

1. `document-create-btn` → `document-form`
2. Select `BILL_OF_LADING` in `document-type-select`
3. Verify prefill: vessel, ports from shipment
4. Submit → row appears with `DRAFT` badge
5. Repeat for `COMMERCIAL_INVOICE` and `PACKING_LIST`

### US3 — Finalize

1. Open DRAFT row → `document-finalize-btn`
2. Confirm dialog → finalize succeeds
3. Status → `FINALIZED`, `docNumber` visible
4. Form fields read-only; delete disabled

### US4 — Download PDF

1. `document-download-btn` on FINALIZED doc
2. PDF saves; opens with QR corner
3. Network: `GET .../download` → 200

### US5 — QR / Verify

1. `document-qr-preview` shows QR
2. Open `/api/verify/{id}` in browser (no auth)
3. JSON shows `valid: true`, `docNumber`, `sha256`

### Edge cases

| Case | Expected |
|------|----------|
| CLOSED shipment | create/finalize blocked — rose error |
| PATCH on FINALIZED | error message |
| Second BL same shipment | warning (non-blocking) + allow |
| Delete DRAFT | removed from list |
| Delete FINALIZED | disabled / API 400 |

---

## Automated QA

```bash
# Backend E2E (Shipping-Project-V2)
cd Shipping-Project-V2/e2e
npx playwright test documents-api.spec.ts

# Frontend build
cd Shipping
npm run build
```

### Visual QA (Polish)

```bash
# After capture keys added to visual-qa.mjs
npm run visual-qa:shipment-documents
```

Reference targets: `13-shipment-documents-desktop.png`, `14-shipment-documents-mobile.png` (to be added in tasks)

---

## Dual-repo commit notes

| Repo | Expected changes |
|------|-------------------|
| `logicore-erp-backend` | Prisma migration, document module, pdf templates, e2e |
| `logicore-erp-frontend` | `documents.ts`, hooks, `shipment-documents-panel`, detail tab |

Tag suggestion after merge: backend patch/minor + frontend `v0.3.0`

---

## Notes

- **T0 blocker**: Frontend work MUST NOT start until T0 passes on new endpoints
- **Amend**: API tested in T0 step 8 (optional); no UI button in v1
- **HTTP 200 check**: before Visual QA capture, verify page returns 200 (lesson from 003 wizard incident)
- Existing `POST /api/documents/upload` and `generate-locked-pdf` — regression in T0
