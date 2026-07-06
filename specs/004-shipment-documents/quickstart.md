# Quickstart: Shipment Documents Validation

**Feature**: 004-shipment-documents

## Prerequisites

1. Docker Postgres — `Shipping-Project-V2/e2e/docker-compose.yml`
2. NestJS backend `:3001` — **با migration جدید Document** (Phase B1+)
3. Frontend `:3000` — `BACKEND_URL=http://localhost:3001`
4. Login: `maryam@logicore.com` / `maryam123` → workspace «سیر راه آبی»

```powershell
# Backend (from Shipping-Project-V2/production-ready/nestjs-backend)
npx prisma migrate deploy
npm run start:dev

# Frontend (from Shipping)
npm run dev
```

---

## T0 Smoke (Backend — before Frontend F1)

```powershell
$SHIPMENT_ID = "<existing-shipment-uuid>"
$BASE = "http://localhost:3001"
# ... POST create → PATCH → finalize → download → verify API
```

**API verify (JSON):** `curl "$BASE/api/verify/$DOCUMENT_ID"` → `valid: true`

**Checkpoint**: E2E `documents-api.spec.ts` 12/12 ✅ — see [CHANGES.md](./CHANGES.md)

---

## Manual Validation (Frontend US1–US5)

### US1 — Document list

1. `/shipments` → open any shipment
2. Click `shipment-tab-documents`
3. `shipment-documents-panel` visible
4. Empty state OR `document-list` with `document-row-{id}`, status badges (DRAFT slate / FINALIZED emerald / AMENDED yellow)
5. Loading: amber `Loader2` on slow network

### US2 — Create / edit DRAFT

1. `document-create-btn` → `document-type-select` (BL / INV / PL)
2. Duplicate warning (amber) if same-type DRAFT/FINALIZED exists — **non-blocking**
3. `document-form` with type-specific fields + prefill from shipment
4. First submit → `POST /documents` (DRAFT, `docNumber` null)
5. Further saves → `PATCH /documents/:id`; form stays open
6. Repeat for `COMMERCIAL_INVOICE` and `PACKING_LIST`

### US3 — Finalize

1. Complete required fields → `document-finalize-btn` enabled (form or row)
2. `document-finalize-confirm-dialog` → `document-finalize-confirm-btn`
3. Status → `FINALIZED`, `docNumber` e.g. `BL-000001`
4. Form read-only; amend message (no amend UI in v1)

### US4 — Download PDF

1. `document-download-btn` or `document-row-download-{id}` on FINALIZED/AMENDED
2. File saves as `{docNumber}.pdf` or `fileName`
3. Spinner on button during download; `toast.error` on failure

### US5 — QR / Verify

1. Open FINALIZED doc in form → `document-qr-code` + `document-verify-link`
2. URL format: `http://localhost:3000/verify/{documentId}` (**HTML page**, not raw JSON)
3. Open in browser **without login** → «این سند معتبر است» + type, number, date, issuer, hash
4. Scan QR from ERP or from printed PDF (new finalize after verify-page deploy)
5. API still available for machines: `GET /api/verify/{id}`

### Edge cases

| Case | Expected |
|------|----------|
| CLOSED shipment | banner + no create/finalize; API 409 |
| PATCH on FINALIZED | 409 → rose error |
| Duplicate same type | amber warning; creation allowed |
| Delete DRAFT | removed (if implemented via API) |
| Delete FINALIZED | disabled / API 400 locked |
| Invalid `/verify/bad-id` | «سند یافت نشد» |

---

## Automated QA

```bash
# Backend E2E
cd Shipping-Project-V2/e2e
docker compose -f docker-compose.yml up -d
CI=true npx playwright test documents-api.spec.ts

# Frontend build
cd Shipping
npm run build

# Backend build
cd Shipping-Project-V2/production-ready/nestjs-backend
npm run build
```

### Visual QA (Polish T051)

```bash
# Capture reference PNGs (dev + backend required)
npm run capture

# Compare
npm run visual-qa:shipment-documents
npm run visual-qa:shipment-documents-mobile
# Optional (needs FINALIZED doc in seed):
npm run visual-qa:document-verify
```

References: `13-shipment-documents-desktop.png`, `14-shipment-documents-mobile.png`, `15-document-verify-desktop.png`

---

## Dual-repo commit notes

| Repo | Scope |
|------|--------|
| `logicore-erp-backend` | Prisma, document module, PDF, verify SQL, E2E |
| `logicore-erp` (frontend) | documents lib/hooks, panel, form, verify page, `v0.3.0` |

**Tag suggestion:** backend `v0.2.0` + frontend `v0.3.0`

---

## Notes

- **docNumber**: `BL-000001` / `INV-000001` / `PL-000001` — **only on finalize**
- **Amend**: API only — no UI button in v1
- **Verify UX**: public HTML at `/verify/[id]` — see [CHANGES.md](./CHANGES.md) for deviation from original plan
- **HTTP 200**: before Visual QA, confirm pages return 200 (lesson from 003 wizard)
- Regression: `POST /api/documents/upload`, `generate-locked-pdf` unchanged
