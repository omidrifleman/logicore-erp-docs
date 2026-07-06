# Quickstart: Shipment Wizard Validation

**Feature**: 003-shipment-wizard

## Prerequisites

همان [002-shipments quickstart](../002-shipments/quickstart.md):

1. Docker Postgres + NestJS `:3001` + Frontend `:3000` (`BACKEND_URL=http://localhost:3001`)
2. Login: `maryam@logicore.com` / `maryam123` → «سیر راه آبی»

## T0 Smoke (before implement)

```powershell
# After obtaining token from login:
curl -H "Authorization: Bearer $TOKEN" http://localhost:3001/api/customers
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" `
  -d '{"mode":"SEA","customerId":"<id>","originPort":"BND","destinationPort":"DXB"}' `
  http://localhost:3001/api/shipments
```

Expect: customers array 200؛ POST returns `status: "DRAFT"`.

## Manual Validation

### US1 — Step 1 (Customer + Mode + Route)

1. `/shipments` → click `shipment-create-btn` → `/shipments/new`
2. `data-testid="shipment-wizard"` + `wizard-step-1` visible
3. `wizard-progress` shows step 1 active
4. Submit empty → validation message (no API call)
5. Fill customer + mode + origin/destination → `wizard-next-btn`
6. Step 2 appears; verify DRAFT in network tab (`POST /api/shipments`)

### US2 — Step 2 (Operational)

1. SEA mode: fill vessel + optional ETA/free time
2. `wizard-next-btn` → `PATCH` success → `wizard-step-3`
3. `wizard-back-btn` → step 1 values preserved

### US3 — Step 3 (Optional Leg)

1. Add leg via `wizard-leg-form` OR skip with next
2. If leg added: `POST .../legs` → sequence 1 in review

### US4 — Step 4 (Review + Submit)

1. `wizard-review-summary` shows all entered data
2. `wizard-submit-btn` → redirect `/shipments/{id}`
3. `shipment-detail-view` + `shipment-detail-no` + status badge (DRAFT or synced status)

### Abandon Wizard (Polish QA — T028)

1. Start wizard → complete step 1 only (DRAFT created)
2. Click `wizard-cancel-btn` or nav «بازگشت به لیست»
3. `/shipments` → new DRAFT row visible with correct customer
4. Optional: delete DRAFT from list (002 flow)

## Automated QA

```bash
# E2E smoke (Playwright — US1–US4 + abandon + mobile overflow)
node design-reference/wizard-smoke-qa.mjs

# Visual QA
npm run visual-qa:shipment-wizard
npm run visual-qa:shipment-wizard-mobile
```

Capture reference: `design-reference/11-shipment-wizard-desktop.png`, `12-shipment-wizard-mobile.png`

## Notes

- `GET /api/customers` verified in plan — no BL-004
- If customers fetch fails at runtime: error box + retry + fallback from shipments cache
- **Leg sync (T033)**: پس از `POST .../legs` با وضعیت `SCHEDULED`، بک‌اند `leg-status-sync` ممکن است وضعیت محموله را از `DRAFT` به `BOOKED` تغییر دهد (`shipment.service.ts` → `deriveShipmentStatusFromLegs`). گام ۴ review وضعیت **زنده** را از `useShipmentDetail` نشان می‌دهد؛ مسیر بدون Leg همچنان `DRAFT` می‌ماند.
- **Wizard entry**: برای بارگذاری مشتریان، ورود از `/shipments` → `shipment-create-btn` توصیه می‌شود (مستقیم `/shipments/new` گاهی تا hydration کامل Loader نشان می‌دهد).
- **Conditional testids (T029)**: `wizard-vessel/voyage/eta/free-time` فقط در گام ۲ و mode مربوط؛ `wizard-leg-form` فقط هنگام باز بودن فرم؛ `wizard-submit-btn` و `wizard-review-summary` فقط گام ۴.
- **Visual QA (2026-07-06)**: `shipment-wizard` desktop **0.89%**؛ `shipment-wizard-mobile` **1.64%** — هر دو زیر آستانه ۵٪.
- **Dev server incident (2026-07-06)**: در حین QA، اجرای همزمان چند dev server instance باعث خطای 500 موقت و رندر بدون CSS شد؛ رفع شد با پاک‌سازی `.next` و restart تمیز. توصیه: قبل از Visual QA capture، وضعیت HTTP 200 صفحه را هم verify کن، نه فقط pixel-diff.
