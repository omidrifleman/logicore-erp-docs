# Research: Shipment Creation Wizard

**Feature**: 003-shipment-wizard | **Date**: 2026-07-06

## R0: Backend API Verification

**Decision**: endpointهای لازم wizard در NestJS **از قبل موجود** هستند — Plan بر اساس mock کامل ساخته نمی‌شود.

### R0.1 — Customer list (`GET /api/customers`)

| مورد | مقدار |
|------|--------|
| Controller | `production-ready/nestjs-backend/src/modules/customer/customer.controller.ts` |
| Route | `@Controller('customers')` + `@Get() findAll()` |
| Auth | `@TenantScoped()` — JWT Bearer |
| Service | `customerService.findAll(tx, tenantId)` — `orderBy: companyName asc` |
| **BL-004** | **NOT REQUIRED** — API exists |

**Frontend gap**: `src/lib/customers.ts` و `useCustomers` **وجود ندارد** — در Setup این فیچر اضافه می‌شود.

**Defensive fallback** (فقط اگر T0 یا runtime fetch fail):
- Error box rose + retry
- Optional: derive unique customers from cached `useShipments` data — **not** hardcoded mock list

### R0.2 — Shipment create (`POST /api/shipments`)

| مورد | مقدار |
|------|--------|
| DTO | `CreateShipmentDto` — `mode`, `customerId` required؛ origin/destination optional |
| Initial status | `DRAFT` (service sets + `statusHistory` trigger `created`) |
| Enum mode | `SEA`, `AIR`, `ROAD`, `RAIL`, `MULTIMODAL` |
| Frontend gap | `createShipment()` **not yet** in `src/lib/shipments.ts` |

**Note**: `002` types include `LAND` alias in UI — wizard MUST send `ROAD` to API when user picks زمینی.

### R0.3 — Shipment update (`PATCH /api/shipments/:id`)

| مورد | مقدار |
|------|--------|
| Exists | ✅ `updateShipment` in `src/lib/shipments.ts` |
| Wizard step 2 fields | `vesselName`, `voyageNo`, `eta`, `freeTimeDays` (+ route fields if step 1 edited) |

### R0.4 — Leg create (`POST /api/shipments/:id/legs`)

| مورد | مقدار |
|------|--------|
| Exists | ✅ `createShipmentLeg` + `useCreateShipmentLeg` from 002 |
| Wizard | Reuse leg form field pattern from `shipment-legs-panel.tsx` |

### R0.5 — Carriers (`GET /api/carriers`)

| مورد | مقدار |
|------|--------|
| Exists | ✅ `fetchCarriers` + `useCarriers` |

### T0 Probe Checklist (قبل از implement)

1. `GET /api/customers` → 200، آرایه با `id` + `companyName`
2. `POST /api/shipments` با `{ mode: "SEA", customerId, originPort, destinationPort }` → 201/200، `status: "DRAFT"`
3. `PATCH /api/shipments/:id` با `vesselName`, `eta` → 200
4. `POST /api/shipments/:id/legs` → 200 (optional در T0)

---

## R1: Wizard UX Pattern

**Decision**: صفحه تمام‌عرض داخل `main` با progress bar — **نه** Dialog/Sheet fullscreen.

**Rationale**:
- spec FR-002 — سازگار با ErpShell
- الگوی `leg-form` inline در detail — فیلدها `erpTokens.input`
- framer-motion 0.2s بین گام‌ها (مطابق 002 view switch)

**Alternatives considered**:
- Modal wizard — rejected (spec forbids)
- Single-page all fields — rejected (poor mobile UX)

---

## R2: Step 1 Create vs Deferred Create

**Decision**: `POST` در پایان گام ۱ (پس از validation client).

**Rationale**:
- spec US1: «ساخت DRAFT شیپمنت» در گام ۱
- `shipmentId` برای PATCH/Leg در گام‌های بعد لازم است
- abandon scenario: DRAFT persists (spec edge case) — QA task در Polish

**Alternatives considered**:
- Single POST at step 4 — rejected (no id for legs in step 3)

---

## R3: Mode-Conditional Fields (Step 2)

| mode | vesselName | voyageNo | eta | freeTimeDays |
|------|------------|----------|-----|--------------|
| SEA | shown, required | shown, optional | optional | optional |
| AIR | shown (label: پرواز/هواپیما), optional | hidden | optional | optional |
| ROAD / RAIL | hidden | hidden | optional | optional |
| MULTIMODAL | shown, optional | optional | optional | optional |

**Validation**: client-side قبل از PATCH؛ server class-validator backup.

---

## R4: Origin/Destination Input UX

**Decision**: گام ۱ چهار فیلد اختیاری اما rule: `(originPort OR originCity) AND (destinationPort OR destinationCity)` — حداقل یک مبدا و یک مقصد.

**Rationale**: matches `CreateShipmentDto` optional fields + spec validation.

---

## R5: Visual Reference

**Decision**: فعلاً **بدون** PNG مرجع اختصاصی — checklist دستی + optional capture `11-shipment-wizard-desktop.png` در Polish.

**Rationale**: design-reference فعلی wizard ندارد؛ glass + erpTokens از 002 کافی تا capture بعدی.

---

## R6: Invalidate Queries

| Event | Query keys |
|-------|------------|
| POST DRAFT | `['shipments']` |
| PATCH step 2 | `['shipments', id]`, `['shipment-detail', id]` |
| POST leg | `['shipment-legs', id]`, `['shipment-timeline', id]` |
