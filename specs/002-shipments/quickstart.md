# Quickstart: Shipments Validation

**Feature**: 002-shipments

## Prerequisites

1. Node.js 18+ and npm
2. Docker Postgres (e2e) — `Shipping-Project-V2/e2e/docker-compose.yml`
3. Backend NestJS on `:3001` — `Shipping-Project-V2/production-ready/nestjs-backend`
4. Frontend on `:3000` — this repo (`Shipping`)

## Setup

```powershell
# Terminal 1 — Postgres
docker compose -f Shipping-Project-V2/e2e/docker-compose.yml up -d

# Terminal 2 — backend (from Shipping-Project-V2)
cd production-ready/nestjs-backend
# Set from production-ready/.env PLUS (not in .env by default):
#   DATABASE_URL=postgresql://logicore_app:APP_DB_PASSWORD@localhost:5432/logicore
#   WORKER_DATABASE_URL=postgresql://logicore_worker:WORKER_DB_PASSWORD@localhost:5432/logicore
#   PORT=3001
npm run start

# Terminal 3 — frontend (this repo)
cd Shipping
$env:BACKEND_URL="http://localhost:3001"
npm run dev
```

Login: `maryam@logicore.com` / `maryam123` → workspace «سیر راه آبی»

## Manual Validation

### US1 — List (`/shipments`)

1. Nav → «محموله‌ها» — item active amber
2. Confirm `data-testid="shipment-list-view"` and table rows
3. Summary line: total / in transit / needs attention
4. Filter status dropdown + search debounce
5. Pagination controls (client-side)
6. Click view → `/shipments/{id}`

### US2 — Detail + Timeline

1. `data-testid="shipment-detail-no"` mono amber
2. Status badge + tabs (overview active)
3. `data-testid="shipment-timeline"` with dots
4. Click `transition-btn-*` → new timeline entry
5. «بازگشت به لیست» → `/shipments`

### US3 — Legs Panel

1. `data-testid="shipment-legs-panel"`
2. Empty: `legs-empty` + `leg-create-btn`
3. Add leg → `leg-item-1` visible
4. Change leg status → badge update

### US4 — Mobile

1. DevTools → Pixel 5
2. `/shipments` — nav `w-14`, list usable
3. Compare `design-reference/10-shipments-mobile.png`

## Visual QA (after implementation)

```bash
npm run visual-qa -- --pages shipments-list,shipment-detail,shipment-legs,shipments-mobile
```

**Pass criteria**: diff ≤ 5% per reference PNG; all UI Contract testids present.

### Final Visual QA Results (Phase 7 — ۱۴۰۵/۰۴/۱۶)

| Page | Reference | Diff | Status |
|------|-----------|------|--------|
| `shipments-list` | `03-shipments-list-desktop.png` | **2.66%** | ✅ PASS |
| `shipment-detail` | `04-shipment-detail-desktop.png` | **1.54%** | ✅ PASS |
| `shipment-legs` | `05-shipment-legs-desktop.png` | **1.54%** | ✅ PASS |
| `shipments-mobile` | `10-shipments-mobile.png` | **5.05%** | ⚠️ CONDITIONAL (see Notes) |

Report artifact: `design-reference/qa-output/report.json`

## Notes — Intentional Deviations

### 1. `shipments-mobile` Visual QA — 5.05% (T040)

**وضعیت:** ۰.۰۵٪ بالای آستانه ۵٪ — **انحراف عمدی پذیرفته‌شده** پس از بررسی T040.

**بررسی layout (بدون تطبیق محتوا):**

- DOM موبایل: `nav` ≈ ۵۶px (`w-14`)، بدون overflow افقی صفحه، ۳ ستون اصلی (شماره / مشتری / وضعیت).
- search/filter ستونی (`flex-col`)، `main` با `p-4` — مطابق spec US4.
- ناحیه `header-top` در pixel diff ≈ ۴.۰٪ — اختلاف جزئی shell/محتوا.
- ناحیه `center-card` ≈ ۱۰.۴٪ با `avgColorDrift` ≈ ۹.۹ — **نشان‌دهنده اختلاف متن/رنگ داده زنده API** (برچسب وضعیت، نام مشتری، تعداد ترانزیت) نه layout.
- هیچ تنظیم layout معنادار (padding، breakpoint، ستون‌ها) بدون دستکاری محتوا diff را زیر ۵٪ نمی‌برد؛ تغییر کد صرفاً برای ۰.۰۵٪ توصیه **نمی‌شود**.

**راه‌حل آینده (اختیاری):** mock ثابت seed برای capture مرجع، یا `visual-qa` با mask ناحیه جدول.

### 2. ایجاد محموله (Wizard)

`shipment-create-btn` غیرفعال (`disabled`) — POST `/api/shipments` خارج از scope این فیچر.

### 3. جزئیات عملیاتی — read-only

کارت اطلاعات عملیاتی در detail فقط خواندنی است؛ PATCH از UI پیاده‌سازی نشده (T025).

### 4. Pagination

صفحه‌بندی **client-side** (`pageSize=25`) — backend آرایه کامل برمی‌گرداند (`research.md` R0).

### 5. `leg-form` / `legs-empty` testids

`leg-form` و `leg-form-submit` فقط هنگام باز بودن فرم create/edit در DOM هستند. `legs-empty` فقط وقتی لیست Leg خالی است.

### 6. `shipment-view-{id}` در موبایل

ستون actions در موبایل `hidden md:table-cell` است؛ testid در DOM باقی می‌ماند. ناوبری موبایل از لینک شماره محموله.

## Backend Smoke (API only)

```bash
# After login token obtained:
curl -H "Authorization: Bearer $TOKEN" http://localhost:3001/api/shipments
curl -H "Authorization: Bearer $TOKEN" http://localhost:3001/api/shipments/{id}/timeline
curl -H "Authorization: Bearer $TOKEN" http://localhost:3001/api/shipments/{id}/legs
```

## E2E Reference (V2 repo)

```bash
cd Shipping-Project-V2/e2e
npx playwright test tests/shipment-workflow.spec.ts tests/shipment-legs.spec.ts
```
