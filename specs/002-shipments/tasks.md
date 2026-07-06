# Tasks: Shipments Management

**Status**: ✅ Complete — closed ۱۴۰۵/۰۴/۱۶ (۴۴/۴۴ tasks)

**Input**: Design documents from `/specs/002-shipments/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, `001-login-dashboard` Complete ✅

**Tests**: Visual QA via `design-reference/visual-qa.mjs` (not unit tests for MVP)

**Organization**: Tasks grouped by user story for independent implementation

## Format: `[ID] [P?] [Story] Description`

---

## Phase 0: Backend Live Verification ⚠️ BLOCKER

**Purpose**: تأیید زنده API قبل از هر پیاده‌سازی frontend — جایگزین فرضیات `research.md`

**⚠️ CRITICAL**: هیچ Task مربوط به US1+ تا تکمیل و **تأیید کاربر** T0 اجرا نشود.

- [x] T0 Backend Live Verification — (1) روشن کردن Docker Postgres (`Shipping-Project-V2/e2e/docker-compose.yml`) و NestJS backend روی `:3001` با env معتبر؛ (2) login `maryam@logicore.com` / workspace «سیر راه آبی»؛ (3) اجرای واقعی `GET /api/shipments` و `GET /api/shipments/{seedId}/timeline` (مثلاً `SHP-000001` از seed)؛ (4) مقایسه JSON واقعی با نمونه‌های `specs/002-shipments/research.md` و `contracts/shipments-api.md` — در صورت مغایرت (فیلد جدید، nullable، شکل `availableTransitions`) **توقف**، به‌روزرسانی `research.md` + `contracts/shipments-api.md` + `data-model.md` و **گزارش به کاربر** قبل از T001

**Checkpoint**: کاربر تأیید کرد API live با contract هم‌خوان است → ادامه Phase 1

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: وابستگی‌ها و لایه داده مشترک

- [x] T001 Add `@tanstack/react-table` to `package.json` and install
- [x] T002 Create `src/lib/shipments.ts` — types (`ShipmentListItem`, `ShipmentDetail`, `ShipmentTimeline`, `ShipmentLeg`, `ShipmentListQuery`) + `authFetch` helpers per `contracts/shipments-api.md`
- [x] T003 [P] Extend `src/components/erp/types.tsx` — `legStatusLabels`, `legStatusPalette`, `TRANSPORT_MODES` icons/labels per design-system
- [x] T004 [P] Add visual-qa page keys in `design-reference/visual-qa.mjs` for `shipments-list`, `shipment-detail`, `shipment-legs`, `shipments-mobile` + npm scripts in `package.json`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: کامپوننت‌ها و hookهای مشترک — MUST complete before user stories

**⚠️ CRITICAL**: No user story work until Phase 0 + Phase 2 complete

- [x] T005 Create `src/components/shared/status-badge.tsx` — shared `StatusBadge` with `config` prop + dot pattern per `component-structure.mdc`
- [x] T006 [P] Create `src/hooks/use-shipments.ts` — `useShipments(query)` + `useDeleteShipment()` calling `GET/DELETE /api/shipments`
- [x] T007 [P] Create `src/hooks/use-shipment-detail.ts` — `useShipmentDetail(id)` + `useUpdateShipment(id)` for `GET/PATCH /api/shipments/:id`
- [x] T008 [P] Create `src/hooks/use-shipment-timeline.ts` — `useShipmentTimeline(id)` + `useShipmentTransition(id)` for timeline + `POST .../transition`
- [x] T009 [P] Create `src/hooks/use-shipment-legs.ts` — legs list/create/update/status/delete mutations
- [x] T010 Create route scaffold `src/app/(erp)/shipments/page.tsx` and `src/app/(erp)/shipments/[id]/page.tsx` (placeholder یا shell خالی تا US1/US2 پر شود)

**Checkpoint**: Foundation ready — user story implementation can begin

---

## Phase 3: User Story 1 — لیست محموله‌ها (Priority: P1) 🎯 MVP

**Goal**: فهرست با جستجو، فیلتر وضعیت، صفحه‌بندی client-side، جدول glass — `03-shipments-list-desktop.png`

**Independent Test**: `/shipments` → `shipment-list-view` + rows + filter/search + `shipment-create-btn`

### Implementation for User Story 1

- [x] T011 [US1] Create `src/components/erp/shipment-list.tsx` — header stats، search debounce، status filter، TanStack Table columns per reference
- [x] T012 [US1] Add `data-testid="shipment-list-view"` and `data-testid="shipment-create-btn"` in `shipment-list.tsx`
- [x] T013 [US1] Implement client-side pagination/sort via `@tanstack/react-table` (`getPaginationRowModel`, `getSortedRowModel`) in `shipment-list.tsx`
- [x] T014 [US1] Wire Loading skeleton, ErrorBox (retry), Empty state (Package icon) in `shipment-list.tsx`
- [x] T015 [US1] Add row actions: `shipment-view-{id}` (Link to `/shipments/[id]`) and `shipment-delete-{id}` with DRAFT-only guard
- [x] T016 [US1] Render `StatusBadge` with shipment config on status column
- [x] T017 [US1] Wire `src/app/(erp)/shipments/page.tsx` to render `ShipmentList` (CTA «محموله جدید» — placeholder/disabled per spec out-of-scope wizard)
- [x] T018 [US1] Verify `erp-shell.tsx` nav `/shipments` active state on list route

**Checkpoint**: List page independently testable at `/shipments`

---

## Phase 4: User Story 2 — جزئیات و Timeline (Priority: P2)

**Goal**: جزئیات محموله + timeline + transitions — `04-shipment-detail-desktop.png`

**Independent Test**: `/shipments/{id}` → `shipment-detail-no` + `shipment-timeline` + transition buttons

### Implementation for User Story 2

- [x] T019 [US2] Create `src/components/erp/shipment-detail.tsx` — header (shipmentNo amber, StatusBadge, customer, route), tabs shell (overview/documents/handover — content tabs placeholder)
- [x] T020 [US2] Add `data-testid="shipment-detail-view"` and `data-testid="shipment-detail-no"` in `shipment-detail.tsx`
- [x] T021 [US2] Implement operational info card (شماره، مشتری، کشتی، voyage، مبدا/مقصد، ETA، free time، کانتینر)
- [x] T022 [US2] Implement timeline section with `data-testid="shipment-timeline"`, `timeline-entry`, `timeline-leg-entry` dots per status palette
- [x] T023 [US2] Wire `availableTransitions` → `transition-btn-{to}` buttons (`erpTokens.btnSuccess`) + error display on failure
- [x] T024 [US2] Add «بازگشت به لیست» with `ArrowRight rotate-180` Link to `/shipments`
- [x] T025 [US2] Add Loading/Error/404 empty states — **operational info card read-only** (voyage، free time، vessel و غیره فقط نمایش؛ بدون PATCH/edit toggle — ویرایش به فاز بعدی موکول)
- [x] T026 [US2] Wire `src/app/(erp)/shipments/[id]/page.tsx` to render `ShipmentDetail`

**Checkpoint**: Detail + timeline work independently (legs panel may be empty)

---

## Phase 5: User Story 3 — پنل Legها (Priority: P3)

**Goal**: CRUD Leg + empty state — `05-shipment-legs-desktop.png`

**Independent Test**: Detail page → `shipment-legs-panel`, `legs-empty`, add leg flow

### Implementation for User Story 3

- [x] T027 [US3] Create `src/components/erp/shipment-legs-panel.tsx` — glass card «مسیر حمل (Legها)»
- [x] T028 [US3] Add testids: `shipment-legs-panel`, `legs-empty`, `leg-create-btn`, `legs-list`, `leg-item-{sequence}`, `leg-form`, `leg-form-submit`
- [x] T029 [US3] Implement empty state copy per spec + inline leg form (`erpTokens.input`, mode/carrier/origin/destination/dates)
- [x] T030 [US3] Implement leg list with sequence badge amber + `StatusBadge` leg palette + status select + edit/delete actions
- [x] T031 [US3] Wire leg mutations to `use-shipment-legs.ts`; refresh timeline on leg status change
- [x] T032 [US3] Integrate `ShipmentLegsPanel` into `shipment-detail.tsx` above timeline grid

**Checkpoint**: Leg panel fully functional on detail page

---

## Phase 6: User Story 4 — لیست موبایل (Priority: P4)

**Goal**: لیست واکنش‌گرا در viewport Pixel 5 — `10-shipments-mobile.png`

**Independent Test**: Pixel 5 `/shipments` — nav `w-14`, table usable, core columns visible

### Implementation for User Story 4

- [x] T033 [US4] Add responsive column visibility / horizontal scroll strategy in `shipment-list.tsx` for mobile breakpoint
- [x] T034 [US4] Verify header stats and search/filter stack correctly on narrow viewport
- [x] T035 [US4] Tune padding/typography to match `10-shipments-mobile.png` (no shell rebuild)

**Checkpoint**: Mobile list matches reference layout

---

## Phase 7: Polish & Visual QA

**Purpose**: Cross-cutting validation before merge

- [x] T036 [P] Run `npm run visual-qa:shipments-list` (or `visual-qa.mjs --pages shipments-list`) — target `03-shipments-list-desktop.png`, diff ≤5%
- [x] T037 [P] Run visual QA for `shipment-detail` vs `04-shipment-detail-desktop.png`
- [x] T038 [P] Run visual QA for `shipment-legs` vs `05-shipment-legs-desktop.png`
- [x] T039 [P] Run visual QA for `shipments-mobile` vs `10-shipments-mobile.png`
- [x] T040 Fix content/visual mismatches if diff >5% (content first, like Login module)
- [x] T041 Validate all UI Contract testids from `contracts/shipments-api.md`
- [x] T042 Run `specs/002-shipments/quickstart.md` manual checklist
- [x] T043 Document intentional deviations in `specs/002-shipments/quickstart.md` Notes (if any)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 0 (T0)**: BLOCKS everything — live API verification + user sign-off
- **Phase 1 (Setup)**: Depends on T0 pass
- **Phase 2 (Foundational)**: Depends on Phase 1 — BLOCKS all user stories
- **US1 (Phase 3)**: Depends on Phase 2 — MVP
- **US2 (Phase 4)**: Depends on Phase 2; integrates with US1 navigation
- **US3 (Phase 5)**: Depends on US2 detail page scaffold
- **US4 (Phase 6)**: Depends on US1 list component
- **Polish (Phase 7)**: Depends on US1–US4

### User Story Dependencies

```
T0 → T001–T010 → T011–T018 (US1 MVP)
              → T019–T026 (US2) — needs list navigation from US1
              → T027–T032 (US3) — needs detail from US2
              → T033–T035 (US4) — extends US1 list
              → T036–T043 (Polish)
```

### Parallel Opportunities

- T003 + T004 parallel after T002
- T006 + T007 + T008 + T009 parallel after T005
- T036–T039 visual QA scripts parallel after implementation

---

## Parallel Example: Foundational (after T0)

```bash
# After T005 StatusBadge:
Task T006: use-shipments.ts
Task T007: use-shipment-detail.ts
Task T008: use-shipment-timeline.ts
Task T009: use-shipment-legs.ts
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. **T0** — Backend Live Verification → user approval
2. Phase 1 + Phase 2 (T001–T010)
3. Phase 3 US1 (T011–T018)
4. **STOP** — visual QA list + user approval
5. Continue US2 → US3 → US4

### Suggested Execution Order

```
T0 → T001–T004 → T005–T010 → T011–T018 (US1) → T019–T026 (US2) → T027–T032 (US3) → T033–T035 (US4) → T036–T043
```

**Total tasks**: 44 (T0 + T001–T043) | **US1**: 8 | **US2**: 8 | **US3**: 6 | **US4**: 3 | **MVP scope**: T0 + T001–T018

---

## Notes

- Backend path: `Shipping-Project-V2/production-ready/nestjs-backend` (external to this repo)
- Wizard ایجاد محموله کامل خارج از scope — `shipment-create-btn` فقط CTA
- صفحه‌بندی client-side — API بدون `page`/`limit` (see research R0)
- Do NOT rebuild `erp-shell.tsx`
- T0 mismatch → update contracts before T001
- **US2 Detail**: کارت اطلاعات عملیاتی read-only در این فاز — `PATCH /api/shipments/:id` فقط در hook (T007) برای فاز بعد؛ UI ویرایش ندارد
