# Implementation Plan: Shipments Management

**Branch**: `002-shipments` | **Date**: 2026-07-06 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-shipments/spec.md`

## Summary

پیاده‌سازی ماژول مدیریت محموله‌ها در LogiCore ERP: لیست با فیلتر/جستجو/صفحه‌بندی client-side،
صفحه جزئیات با Timeline وضعیت و transition، پنل Legهای چندمرحله‌ای، و لیست واکنش‌گرا موبایل —
همه داخل `ErpShell` موجود، با اتصال به API واقعی NestJS (`/api/shipments/*`) و تطابق بصری
با `design-reference/03–05` و `10`.

**Backend status**: ✅ تأیید شده — controller/service/e2e موجود (جزئیات در [research.md](./research.md#r0-backend-api-verification-critical)).

## Technical Context

**Language/Version**: TypeScript 5.x, Node.js 22 LTS, React 19.x, Next.js 15.x (App Router)

**Primary Dependencies**: Tailwind CSS v4, shadcn/ui, framer-motion, lucide-react, Vazirmatn,
TanStack Query 5, **@tanstack/react-table** (جدید — برای sort/pagination لیست، الگوی V2)

**Storage**: Cookie + localStorage session (از 001)؛ داده محموله از PostgreSQL via NestJS

**Testing**: Playwright visual QA (`design-reference/visual-qa.mjs`); smoke از V2 e2e
(`shipment-workflow.spec.ts`, `shipment-legs.spec.ts`)

**Target Platform**: Web — desktop 1280×800 + mobile Pixel 5

**Project Type**: Web application (frontend `Shipping/` + backend `Shipping-Project-V2/.../nestjs-backend`)

**Performance Goals**: لیست اولین ردیف <5s؛ جزئیات + timeline <5s (شبکه عادی)

**Constraints**: DESIGN-SYSTEM tokens only؛ ErpShell reuse؛ StatusBadge shared؛
صفحه‌بندی client-side (API بدون page/limit)

**Scale/Scope**: 2 routes + 3–4 ERP components؛ wizard ایجاد خارج از scope

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Tech Stack | ✅ PASS | Next.js + Tailwind v4 + shadcn + framer-motion + Vazirmatn |
| II. RTL Persian | ✅ PASS | `dir="rtl"`؛ LTR فقط فیلدهای کد/شماره در صورت نیاز |
| III. Design System | ✅ PASS | glass table، status palettes، timeline dots |
| IV. Component Reuse | ✅ PASS | shared StatusBadge؛ erpTokens؛ پورت adapt از V2 |
| V. Async States | ✅ PASS | Loading/Error/Empty در list، detail، legs |
| VI. Simplicity | ✅ PASS | بدون wizard؛ بدون rebuild shell |

**Post-design re-check**: ✅ All gates pass. TanStack Table اضافه برای pagination — justified در research R3.

## Project Structure

### Documentation (this feature)

```text
specs/002-shipments/
├── plan.md              # This file
├── research.md          # Backend verification + API + table decision
├── data-model.md        # Entities from spec
├── quickstart.md        # Validation guide
├── contracts/
│   └── shipments-api.md
├── checklists/
│   └── requirements.md
└── tasks.md             # (/speckit-tasks — not yet)
```

### Source Code (repository root)

```text
src/
├── app/
│   └── (erp)/
│       ├── layout.tsx              # ErpShell (existing)
│       ├── page.tsx                # Dashboard (existing)
│       └── shipments/
│           ├── page.tsx            # US1 + US4 list
│           └── [id]/
│               └── page.tsx        # US2 + US3 detail
├── components/
│   ├── shared/
│   │   └── status-badge.tsx        # NEW — shared badge
│   └── erp/
│       ├── erp-shell.tsx           # existing — nav wire only
│       ├── types.tsx               # extend leg status labels
│       ├── shipment-list.tsx       # NEW
│       ├── shipment-detail.tsx     # NEW
│       └── shipment-legs-panel.tsx # NEW
├── hooks/
│   ├── use-shipments.ts            # list + delete
│   ├── use-shipment-detail.ts      # detail + patch
│   ├── use-shipment-timeline.ts    # timeline + transition
│   └── use-shipment-legs.ts        # legs CRUD
└── lib/
    └── shipments.ts                # types + fetch helpers
```

**Structure Decision**: توسعه فقط در repo `Shipping/`؛ بک‌اند خارجی در
`Shipping-Project-V2/production-ready/nestjs-backend` — بدون تغییر backend در این feature
مگر باگ blocker.

## Phase Overview

| Phase | Scope | Deliverable |
|-------|--------|-------------|
| 0 | Research | `research.md` ✅ |
| 1 | Design | `data-model.md`, `contracts/`, `quickstart.md` ✅ |
| 2 | Tasks | `tasks.md` — **await `/speckit-tasks` + user approval** |
| 3 | Implement | US1→US2→US3→US4 + visual QA |

## Implementation Notes

### Routing

- `/shipments` → `ShipmentList` در `(erp)/shipments/page.tsx`
- `/shipments/[id]` → `ShipmentDetail` + `ShipmentLegsPanel` در `(erp)/shipments/[id]/page.tsx`
- `erp-shell.tsx`: لینک `/shipments` از قبل — active state با `pathname.startsWith('/shipments')`

### API Layer

- `src/lib/shipments.ts` — mirror `shipmentApi` از V2 `api.ts`
- Proxy `/api/*` → `:3001` (موجود از 001)

### Pagination Strategy

Backend `GET /api/shipments` → full array. Frontend:
1. Fetch with `status` + `search` query params
2. TanStack Table `getPaginationRowModel` با `pageSize` 25
3. Document در contract — **not** a backend gap blocker

### Visual QA Pages (to add in tasks)

| Page key | Reference PNG |
|----------|---------------|
| `shipments-list` | `03-shipments-list-desktop.png` |
| `shipment-detail` | `04-shipment-detail-desktop.png` |
| `shipment-legs` | `05-shipment-legs-desktop.png` |
| `shipments-mobile` | `10-shipments-mobile.png` |

### Port Map (V2 → Shipping)

| V2 source | Shipping target |
|-----------|-----------------|
| `shipment-list.tsx` | `shipment-list.tsx` |
| `shipment-detail.tsx` | `shipment-detail.tsx` |
| `shipment-legs-panel.tsx` | `shipment-legs-panel.tsx` |
| `types.tsx` StatusBadge | `shared/status-badge.tsx` |

## Complexity Tracking

> No constitution violations requiring justification.

| Item | Note |
|------|------|
| `@tanstack/react-table` | New dep — required for sort/pagination parity with design reference |

## Dependencies

- ✅ `001-login-dashboard` Complete (auth, shell, visual QA infra)
- ✅ Backend shipments API implemented (see research R0)
- ✅ Design references 03, 04, 05, 10 in `design-reference/`

## Next Steps

1. User reviews `plan.md` + `research.md` (API verification)
2. `/speckit-tasks` — generate `tasks.md`
3. User approves → `/speckit-implement`
