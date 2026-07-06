# Implementation Plan: Shipment Creation Wizard

**Branch**: `003-shipment-wizard` | **Date**: 2026-07-06 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/003-shipment-wizard/spec.md`

## Summary

پیاده‌سازی wizard چهارمرحله‌ای ایجاد محموله جدید داخل `ErpShell`: گام ۱ (مشتری + mode + مسیر → `POST` DRAFT)، گام ۲ (جزئیات عملیاتی → `PATCH`)، گام ۳ (Leg اختیاری → `POST .../legs`)، گام ۴ (بازبینی + redirect به جزئیات). دکمه غیرفعال `shipment-create-btn` در لیست 002 فعال و به `/shipments/new` لینک می‌شود.

**Backend status**: ✅ `POST/PATCH /api/shipments`، `POST .../legs`، **`GET /api/customers`** — همه در NestJS موجود (جزئیات [research.md](./research.md#r0-backend-api-verification)).

---

## Customer API — بررسی صریح (الزام کاربر)

| سؤال | نتیجه |
|------|--------|
| آیا `GET /api/customers` در NestJS وجود دارد؟ | **✅ بله** — `CustomerController` در `production-ready/nestjs-backend/src/modules/customer/customer.controller.ts`، متد `@Get() findAll()` |
| مسیر کامل | `GET /api/customers` (tenant-scoped، Bearer JWT) |
| پاسخ | آرایه `Customer[]` با `id`, `companyName` و فیلدهای CRM اضافی — wizard فقط subset picker استفاده می‌کند |
| آیا BL-004 لازم است؟ | **❌ خیر** — endpoint از قبل پیاده‌سازی شده؛ نیازی به تیکت بک‌لاگ برای ساخت API نیست |
| تصمیم frontend | `useCustomers()` + `fetchCustomers()` در `src/lib/customers.ts` (جدید) |
| Fallback موقت | **فقط در خطای شبکه/API** (نه نبود endpoint): نمایش پیام rose + دکمه retry؛ اختیاری استخراج `customer` یکتا از `GET /api/shipments` به‌عنوان last-resort picker — **بدون mock ثابت hardcode** |

> اگر در T0 probe زنده `GET /api/customers` شکست خورد (مثلاً ۵۰۱)، قبل از implement گزارش به کاربر — در آن صورت BL-004 ثبت و mock فعال می‌شود.

---

## Technical Context

**Language/Version**: TypeScript 5.x, React 19.x, Next.js 15.x (App Router)

**Primary Dependencies**: Tailwind v4, shadcn/ui, framer-motion, TanStack Query 5, موجود از 001/002

**Storage**: Session cookie + PostgreSQL via NestJS

**Testing**: Playwright/manual quickstart؛ Visual QA wizard (capture جدید در tasks)

**Target Platform**: Desktop ≥768px + mobile Pixel 5 (wizard usable در shell)

**Project Type**: Frontend `Shipping/` + backend خارجی `Shipping-Project-V2/.../nestjs-backend`

**Performance Goals**: تکمیل wizard <3 دقیقه؛ اعتبارسنجی گام ۱ <1s client-side

**Constraints**: ErpShell reuse؛ `erpTokens`؛ بدون modal fullscreen؛ وضعیت اولیه همیشه DRAFT

**Scale/Scope**: 1 route جدید (`/shipments/new`) + 1 کامپوننت wizard + hook مشتری + توسعه `shipments.ts`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Tech Stack | ✅ PASS | Next.js + Tailwind + shadcn + framer + Vazirmatn |
| II. RTL Persian | ✅ PASS | wizard RTL؛ ETA/date `dir="ltr"` |
| III. Design System | ✅ PASS | glass card، progress indicator amber، erpTokens.input |
| IV. Component Reuse | ✅ PASS | reuse leg-form patterns، useCarriers، useShipmentLegs |
| V. Async States | ✅ PASS | هر گام Loading/Error/Empty |
| VI. Simplicity | ✅ PASS | یک کامپوننت wizard؛ بدون state machine جدید |

**Post-design re-check**: ✅ All gates pass.

## Project Structure

### Documentation (this feature)

```text
specs/003-shipment-wizard/
├── plan.md              # This file
├── research.md          # API verification + wizard flow decisions
├── data-model.md        # Create DTOs + validation + wizard state
├── quickstart.md        # Manual validation
├── contracts/
│   └── shipment-wizard-api.md
├── checklists/
│   └── requirements.md  # (from specify)
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── app/(erp)/shipments/
│   ├── page.tsx                 # existing list — enable create btn
│   ├── new/
│   │   └── page.tsx             # NEW — wizard route
│   └── [id]/page.tsx            # existing detail (redirect target)
├── components/erp/
│   ├── shipment-wizard.tsx      # NEW — 4-step wizard + progress
│   ├── shipment-list.tsx        # MODIFY — link create btn
│   └── shipment-wizard-steps/   # optional subcomponents per step
├── hooks/
│   ├── use-customers.ts         # NEW
│   ├── use-create-shipment.ts   # NEW — POST mutation
│   └── use-shipment-detail.ts   # existing — PATCH reuse
└── lib/
    ├── shipments.ts             # MODIFY — add createShipment + CreateShipmentDto
    └── customers.ts             # NEW — types + fetchCustomers
```

**Structure Decision**: توسعه فقط در `Shipping/`؛ بک‌اند بدون تغییر مگر blocker در T0.

## Phase Overview

| Phase | Scope | Deliverable |
|-------|--------|-------------|
| 0 | T0 live API probe | customers + POST shipments |
| 1 | Setup | `customers.ts`, `createShipment`, hooks |
| 2 | Foundation | route `/shipments/new`, wizard shell + progress |
| 3 | US1 | Step 1 — customer/mode/route → DRAFT |
| 4 | US2 | Step 2 — operational PATCH |
| 5 | US3 | Step 3 — optional leg |
| 6 | US4 | Step 4 — review + submit redirect |
| 7 | Polish | testids, quickstart, **abandon-wizard QA**, visual capture |

## Wizard Flow (Technical)

```
/shipments/new
  Step 1: validate → POST /api/shipments → store shipmentId
  Step 2: validate mode-fields → PATCH /api/shipments/:id
  Step 3: optional POST /api/shipments/:id/legs OR skip
  Step 4: review readonly → wizard-submit-btn → router.push(/shipments/:id)
```

**State**: React `useState` + TanStack Query mutations؛ `shipmentId` پس از گام ۱ در component state (نه URL v1).

**Progress indicator**: افقی ۴ مرحله — `glass` bar، active=amber، completed=emerald dot، upcoming=muted.

**Abandon flow**: لینک «بازگشت به لیست» در header wizard — DRAFT از گام ۱ در DB باقی می‌ماند؛ QA در Polish phase (tasks T028).

## Routing

| Path | Component |
|------|-----------|
| `/shipments/new` | `ShipmentWizard` |
| `/shipments` | `ShipmentList` — `shipment-create-btn` → Link `/shipments/new` |

`erp-shell.tsx`: بدون تغییر ساختاری؛ active state `/shipments*` از قبل.

## API Layer

| Action | Method | Path |
|--------|--------|------|
| List customers | GET | `/api/customers` |
| Create shipment | POST | `/api/shipments` |
| Update shipment | PATCH | `/api/shipments/:id` |
| Create leg | POST | `/api/shipments/:id/legs` |
| List carriers | GET | `/api/carriers` (existing) |

جزئیات در [contracts/shipment-wizard-api.md](./contracts/shipment-wizard-api.md).

## Complexity Tracking

> No constitution violations.

| Item | Note |
|------|------|
| `src/lib/customers.ts` | جدید — سبک `shipments.ts` |
| `createShipment` در `shipments.ts` | قبلاً در frontend نبود — اضافه در Setup |

## Dependencies

- ✅ `002-shipments` Complete
- ✅ `001-login-dashboard` Complete
- ✅ `GET /api/customers` در backend
- ⏳ T0 live verification قبل از implement

## Next Steps

1. User reviews plan + design artifacts
2. `/speckit-implement` پس از تأیید
3. T0 → Setup → US1–US4 → Polish
