# Research: Shipments Management

**Feature**: 002-shipments | **Date**: 2026-07-06

## R0: Backend API Verification (CRITICAL)

**Decision**: بک‌اند NestJS در `Shipping-Project-V2/production-ready/nestjs-backend` **پیاده‌سازی واقعی** دارد — Plan بر اساس API فرضی ساخته **نمی‌شود**.

**Evidence (کد)**:

| منبع | مسیر | وضعیت |
|------|------|--------|
| Controller | `src/modules/shipment/shipment.controller.ts` | `@Controller('shipments')` + global prefix `api` |
| Service | `src/modules/shipment/shipment.service.ts` | CRUD، timeline، legs، transition، handovers |
| State machine | `src/modules/shipment/shipment-transitions.ts` | transitions + guards |
| DTOs | `src/modules/shipment/dto/*.ts` | class-validator |

**Evidence (تست)**:

| نوع تست | مسیر | پوشش |
|---------|------|------|
| E2E Playwright | `Shipping-Project-V2/e2e/tests/shipment-workflow.spec.ts` | لیست، جزئیات، timeline، transition، ویرایش |
| E2E Playwright | `Shipping-Project-V2/e2e/tests/shipment-legs.spec.ts` | CRUD leg، sync وضعیت، timeline leg |
| E2E Playwright | `Shipping-Project-V2/e2e/tests/customer-shipment.spec.ts` | ایجاد محموله از مشتری |
| Unit tests NestJS | — | **وجود ندارد** برای ماژول shipment |

**Live probe (T0 — 2026-07-06)**: Docker Postgres (`e2e/docker-compose.yml`) + NestJS `:3001` روشن شد. Login `maryam@logicore.com` → workspace «سیر راه آبی». Probe زنده:

| Endpoint | نتیجه |
|----------|--------|
| `GET /api/shipments` | 200 — آرایه ۳۶ آیتم (DB شامل داده e2e قبلی) |
| `GET /api/shipments/{id}/timeline` | 200 — `SHP-000001` (`cmr26brlh000iiitc9cvtl60r`) |

**مغایرت با نمونه قبلی research (اصلاح شد)**:

| موضوع | واقعیت live | اقدام |
|--------|-------------|--------|
| فیلدهای لیست | علاوه بر subset مستند، API برمی‌گرداند: `tenantId`, `customerId`, `parentShipmentId`, `voyageNo`, `actualArrival`, `freeTimeStart`, `freeTimeDays`, `statusHistory`, `updatedAt` | contracts + data-model به‌روز شد — frontend می‌تواند subset استفاده کند |
| `statusHistory` در لیست | در response لیست **هم** embed است (Prisma full row) | data-model اصلاح شد (قبلاً فقط detail ذکر شده بود) |
| `guard` در transitions | می‌تواند `null` یا نام کلاس مثل `"BorderGateway"` باشد | بدون تغییر contract — نمونه live اضافه شد |
| Timeline shape | `currentStatus`, `entries`, `legEntries`, `activeLeg`, `availableTransitions` — **هم‌خوان** | ✅ |
| وضعیت seed `SHP-000001` | live: `ARRIVED` (نه `IN_TRANSIT` seed اولیه) — به‌خاطر e2e transitions قبلی | داده تست؛ shape مهم است نه status |

**نکته env**: `.env` در `production-ready/` فاقد `DATABASE_URL`/`WORKER_DATABASE_URL` است — برای boot محلی باید ست شوند (مقادیر e2e در `quickstart.md`).

**Gaps شناخته‌شده (مهم برای Plan)**:

| موضوع | واقعیت API | تصمیم frontend |
|--------|------------|----------------|
| صفحه‌بندی سرور | `GET /api/shipments` آرایه کامل برمی‌گرداند؛ **بدون** `page`/`limit` | صفحه‌بندی **client-side** با TanStack Table `getPaginationRowModel` (الگوی V2) |
| جستجو | `?search=` سمت سرور (فیلتر روی shipmentNo، companyName، vesselName) | debounce 300ms + query param |
| فیلتر وضعیت | `?status=DRAFT` (exact match) | dropdown «همه وضعیت‌ها» + enum |
| Wizard ایجاد | `POST /api/shipments` موجود | خارج از MVP spec — فقط CTA |

---

## R1: API Surface — Endpoints & Response Shapes

**Base URL**: `http://localhost:3001` (proxied as `/api` via `next.config.ts`)

**Auth**: `Authorization: Bearer {logicore-token}` — tenant از JWT + RLS

### لیست و CRUD محموله

| Method | Path | Query/Body | Response |
|--------|------|------------|----------|
| `GET` | `/api/shipments` | `?status=&search=` | `ShipmentListItem[]` |
| `POST` | `/api/shipments` | `CreateShipmentDto` | `ShipmentListItem` |
| `GET` | `/api/shipments/:id` | — | `ShipmentDetail` |
| `PATCH` | `/api/shipments/:id` | `UpdateShipmentDto` | `ShipmentDetail` |
| `DELETE` | `/api/shipments/:id` | — | `{ deleted: true, id }` |

### Timeline و Transition

| Method | Path | Body | Response |
|--------|------|------|----------|
| `GET` | `/api/shipments/:id/timeline` | — | `ShipmentTimeline` |
| `POST` | `/api/shipments/:id/transition` | `{ newStatus, trigger, guard? }` | `ShipmentDetail` |

### Legها

| Method | Path | Body | Response |
|--------|------|------|----------|
| `GET` | `/api/shipments/:id/legs` | — | `ShipmentLegItem[]` |
| `POST` | `/api/shipments/:id/legs` | `CreateShipmentLegDto` | `ShipmentLegItem` |
| `PATCH` | `/api/shipments/:id/legs/:legId` | `UpdateShipmentLegDto` | `ShipmentLegItem` |
| `PATCH` | `/api/shipments/:id/legs/:legId/status` | `{ status }` | `ShipmentLegItem` |
| `DELETE` | `/api/shipments/:id/legs/:legId` | — | `{ deleted: true, id }` |

### Handovers (خارج از MVP UI — contract برای فاز بعد)

| Method | Path |
|--------|------|
| `GET/POST` | `/api/shipments/:id/handovers` |
| `DELETE` | `/api/shipments/:id/handovers/:handoverId` |

### نمونه Response — `GET /api/shipments` (آیتم)

```json
{
  "id": "uuid",
  "shipmentNo": "SHP-000001",
  "tenantId": "uuid",
  "customerId": "uuid",
  "parentShipmentId": null,
  "status": "ARRIVED",
  "mode": "SEA",
  "originPort": "CNSHA",
  "destinationPort": "AEJEA",
  "originCity": null,
  "destinationCity": null,
  "vesselName": "MSC ATHENA",
  "voyageNo": "V.E2E-TEST",
  "eta": "2026-07-06T00:00:00.000Z",
  "actualArrival": "2026-07-01T16:31:54.013Z",
  "freeTimeStart": null,
  "freeTimeDays": 5,
  "statusHistory": [{ "from": "IN_TRANSIT", "to": "ARRIVED", "trigger": "vessel_arrived", "at": "ISO", "by": "userId", "guard": null }],
  "createdAt": "2026-07-01T14:31:32.214Z",
  "updatedAt": "2026-07-02T12:22:55.005Z",
  "customer": { "id": "uuid", "companyName": "شرکت بازرگانی پارس" },
  "_count": { "containers": 2, "documents": 0 }
}
```

> **T0 note**: Prisma `findMany` کل ستون‌های Shipment را برمی‌گرداند — subset بالا برای UI کافی است؛ فیلدهای اضافی را نادیده بگیرید مگر نیاز شود.

### نمونه Response — `GET /api/shipments/:id/timeline`

```json
{
  "currentStatus": "CONFIRMED",
  "entries": [
    { "from": null, "to": "DRAFT", "trigger": "created", "at": "ISO", "by": "userId", "guard": null },
    { "from": "DRAFT", "to": "CONFIRMED", "trigger": "customer_confirmed", "at": "ISO", "by": "userId", "guard": null }
  ],
  "legEntries": [],
  "activeLeg": null,
  "availableTransitions": [
    { "to": "CUSTOMS_HOLD", "label": "بازداشت گمرک", "trigger": "customs_hold", "guard": "BorderGateway" },
    { "to": "RELEASED", "label": "ترخیص مستقیم", "trigger": "customs_cleared", "guard": "BorderGateway" }
  ]
}
```

> **T0 live sample** (`SHP-000001`, status `ARRIVED`): `guard` می‌تواند نام کلاس guard باشد — در UI به‌صورت guard note amber نمایش داده شود.

### نمونه Response — `GET /api/shipments/:id/legs` (آیتم)

```json
{
  "id": "uuid",
  "sequence": 1,
  "mode": "LAND",
  "origin": "AEJEA",
  "destination": "تهران",
  "carrierId": "uuid|null",
  "carrier": { "id": "uuid", "name": "...", "code": "..." },
  "departureAt": null,
  "arrivalAt": null,
  "status": "SCHEDULED",
  "createdAt": "ISO",
  "updatedAt": "ISO"
}
```

**Errors**: 401 → redirect login؛ 404 `NotFoundException`؛ 409 `ConflictException` (حذف غیر-DRAFT، leg روی CLOSED)؛ 400 validation — `{ statusCode, message }`

**مرجع TypeScript کامل**: `Shipping-Project-V2/src/lib/api.ts` (`shipmentApi`, interfaces)

---

## R2: Data Fetching Pattern

**Decision**: TanStack Query hooks در `src/hooks/` — `useShipments`, `useShipmentDetail`, `useShipmentTimeline`, `useShipmentLegs` + mutations برای delete/transition/leg CRUD.

**Rationale**: هم‌راستا با `001-login-dashboard` و constitution V (async states).

**Alternatives considered**:
- Server Components — rejected (session client-side، transitions تعاملی)

---

## R3: Table Component Strategy

**Decision**: **جدول shadcn/ui جدید لازم نیست** برای MVP. از **الگوی جدول ERP موجود** در `management-dashboard.tsx` (native `<table>` + `glass` wrapper) استفاده شود؛ برای sort/pagination client-side از **TanStack Table** (الگوی `Shipping-Project-V2/shipment-list.tsx`) پورت می‌شود.

**Rationale**:
- Dashboard overdue table همان الگوی بصری design-reference را دارد (`glass`, header uppercase, row hover).
- V2 از `@tanstack/react-table` برای sort + pagination استفاده می‌کند — dependency از قبل در پروژه **نیست**؛ باید در tasks اضافه شود **یا** pagination ساده بدون TanStack (دکمه قبلی/بعدی روی slice) — **توصیه Plan**: اضافه کردن `@tanstack/react-table` چون مرجع V2 و feature sort/pagination را پوشش می‌دهد.

**Alternatives considered**:
- shadcn `Table` component only — کافی برای markup ولی sort/pagination دستی
- کپی کامل V2 shipment-list — rejected (وابستگی به wizard callback؛ باید adapt شود)

---

## R4: StatusBadge Shared Component

**Decision**: ایجاد `src/components/shared/status-badge.tsx` — یک کامپوننت با prop `config: Record<string, StatusPalette>` و label map از `src/components/erp/types.tsx` (shipment + leg palettes جدا).

**Rationale**: `component-structure.mdc` — تکرار badge ممنوع؛ V2 دارد `StatusBadge` در `types.tsx` که باید به shared منتقل/استخراج شود.

---

## R5: Routing & Shell

**Decision**: صفحات زیر `(erp)` layout موجود:
- `src/app/(erp)/shipments/page.tsx` — لیست
- `src/app/(erp)/shipments/[id]/page.tsx` — جزئیات + legs panel

Nav link `/shipments` در `erp-shell.tsx` از قبل تعریف شده — فقط wire صفحات.

**Rationale**: spec FR-001 — shell دوباره ساخته نمی‌شود.

---

## R6: Visual QA Strategy

**Decision**: گسترش `visual-qa.mjs` با pages `shipments-list`, `shipment-detail`, `shipment-legs`, `shipments-mobile` — markers از UI Contract spec.

**Rationale**: همان gate 5% از 001.

**Alternatives considered**:
- فقط capture-screenshots — کافی برای regression ولی بدون diff gate

---

## R7: Reference Implementation Port

**Decision**: پورت انتخابی از `Shipping-Project-V2/src/components/erp/`:
- `shipment-list.tsx` → `src/components/erp/shipment-list.tsx`
- `shipment-detail.tsx` → `src/components/erp/shipment-detail.tsx`
- `shipment-legs-panel.tsx` → `src/components/erp/shipment-legs-panel.tsx`

Adapt: imports به `@/lib/shipments` (جدید)، shared StatusBadge، حذف wizard از list (CTA فقط).

**Rationale**: مراجع بصری از همان UI گرفته شده؛ کاهش ریسک diff.
