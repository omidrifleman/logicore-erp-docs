# Tasks: Shipment Creation Wizard

**Status**: ✅ Complete — closed ۱۴۰۵/۰۴/۱۶ (۳۴/۳۴ tasks)

**Input**: Design documents from `/specs/003-shipment-wizard/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, `002-shipments` Complete ✅

**Tests**: Manual quickstart + optional visual QA capture

**Organization**: Tasks grouped by user story

## Format: `[ID] [P?] [Story] Description`

---

## Phase 0: Backend Live Verification ⚠️ BLOCKER

**Purpose**: تأیید `GET /api/customers` + `POST /api/shipments` قبل از UI

**⚠️ CRITICAL**: No US work until T0 pass + user sign-off

- [x] T0 Backend Live Verification — ✅ 2026-07-06: health 200؛ login `maryam@logicore.com` → workspace «سیر راه آبی کشتیرانی»؛ `GET /api/customers` → 200 (42 items، نمونه: `شرکت بازرگانی پارس`)؛ `POST /api/shipments` → 201 `SHP-000038` status `DRAFT`

**Checkpoint**: API live confirmed → Phase 1

---

## Phase 1: Setup (Shared Infrastructure)

- [x] T001 Create `src/lib/customers.ts` — `CustomerListItem` type + `fetchCustomers()` → `GET /api/customers`
- [x] T002 Extend `src/lib/shipments.ts` — `CreateShipmentPayload` type + `createShipment()` → `POST /api/shipments`
- [x] T003 [P] Create `src/hooks/use-customers.ts` — `useCustomers()` with Loading/Error states
- [x] T004 [P] Create `src/hooks/use-create-shipment.ts` — `useCreateShipment()` mutation + invalidate `['shipments']`

---

## Phase 2: Foundational (Blocking Prerequisites)

**⚠️ CRITICAL**: No user story until Phase 0 + Phase 2 complete

- [x] T005 Create route `src/app/(erp)/shipments/new/page.tsx` — render wizard placeholder
- [x] T006 Create `src/components/erp/shipment-wizard.tsx` — shell: `data-testid=shipment-wizard`, progress bar (`wizard-progress`), step state machine (1–4), framer 0.2s
- [x] T007 Add wizard navigation buttons — `wizard-next-btn`, `wizard-back-btn`, `wizard-cancel-btn` (link `/shipments`)
- [x] T008 Wire `shipment-list.tsx` — enable `shipment-create-btn` as Link to `/shipments/new`
- [x] T009 Verify `erp-shell.tsx` active state still works for `/shipments/new` (no rebuild)

**Checkpoint**: Empty wizard route navigable from list

---

## Phase 3: User Story 1 — گام ۱: مشتری + mode + مسیر (Priority: P1) 🎯 MVP

**Goal**: POST DRAFT shipment

**Independent Test**: Step 1 only → DRAFT in API + advance to step 2

- [x] T010 [US1] Implement `wizard-step-1` — customer select (`wizard-customer-select`), mode (`wizard-mode-select`), origin/destination fields
- [x] T011 [US1] Client validation — customerId + mode + origin/destination rules per data-model.md
- [x] T012 [US1] Loading/Error/Empty for customer picker (Loader2, rose box, «مشتری‌ای یافت نشد»)
- [x] T013 [US1] Wire step 1 submit → `useCreateShipment` → store `shipmentId` + `shipmentNo` in wizard state
- [x] T014 [US1] Map UI `LAND` → API `ROAD` on POST
- [x] T015 [US1] Defensive fallback if customers API fails — error + retry (optional derive from shipments cache per research R0.1)

**Checkpoint**: Step 1 creates DRAFT independently

---

## Phase 4: User Story 2 — گام ۲: جزئیات عملیاتی (Priority: P2)

**Goal**: PATCH operational fields with mode-conditional UI

**Independent Test**: From step 2 → PATCH → step 3

- [x] T016 [US2] Implement `wizard-step-2` — vessel, voyage, eta, freeTimeDays with mode-conditional visibility
- [x] T017 [US2] Labels: SEA vessel/voyage؛ AIR vessel label «پرواز»؛ hide vessel for ROAD/RAIL
- [x] T018 [US2] Client validation — SEA vesselName required؛ freeTimeDays 0–60
- [x] T019 [US2] Wire `useUpdateShipment` (existing hook) on step 2 next
- [x] T020 [US2] Back navigation step 2→1 — PATCH step1 fields if edited after create

**Checkpoint**: Operational details saved on DRAFT

---

## Phase 5: User Story 3 — گام ۳: Leg اختیاری (Priority: P3)

**Goal**: Optional first leg via existing legs API

**Independent Test**: Skip leg OR add one leg → step 4

- [x] T021 [US3] Implement `wizard-step-3` — skip path + optional `wizard-leg-form` (reuse leg-form patterns from `shipment-legs-panel.tsx`)
- [x] T022 [US3] Wire `useCreateShipmentLeg` + `useCarriers` for step 3
- [x] T023 [US3] Show leg summary in step 3 after successful create
- [x] T024 [US3] Error handling rose box on leg POST failure

**Checkpoint**: Wizard completable with or without leg

---

## Phase 6: User Story 4 — گام ۴: بازبینی و ثبت (Priority: P4)

**Goal**: Review summary + redirect to detail

**Independent Test**: Step 4 submit → `/shipments/{id}` detail DRAFT

- [x] T025 [US4] Implement `wizard-step-4` — `wizard-review-summary` readonly (customer, mode, route, ops, leg if any)
- [x] T026 [US4] Wire `wizard-submit-btn` → `router.push(/shipments/{shipmentId})`
- [x] T027 [US4] Detail page loads created shipment — verify `shipment-detail-view` + DRAFT badge

**Checkpoint**: Full happy-path wizard complete

---

## Phase 7: Polish & Validation

**Purpose**: Cross-cutting QA before merge

- [x] T028 [P] **QA: Abandon wizard mid-flow** — per quickstart «Abandon Wizard»: complete step 1 only → `wizard-cancel-btn` or back to list → verify DRAFT appears in `/shipments` list with correct customer (spec US4 scenario 4)
- [x] T029 [P] Validate all wizard UI Contract testids from `contracts/shipment-wizard-api.md`
- [x] T030 [P] Run `quickstart.md` manual checklist (US1–US4 + abandon)
- [x] T031 [P] Mobile smoke: wizard usable on Pixel 5 viewport (no page overflow)
- [x] T032 [P] Add visual-qa page key `shipment-wizard` + capture `11-shipment-wizard-desktop.png` (optional diff ≤5%)
- [x] T033 Document deviations in `quickstart.md` Notes (if any)

---

## Dependencies & Execution Order

```
T0 → T001–T004 → T005–T009 → T010–T015 (US1 MVP)
                              → T016–T020 (US2)
                              → T021–T024 (US3)
                              → T025–T027 (US4)
                              → T028–T033 (Polish)
```

### User Story Dependencies

```
T0 → Setup → Foundation → US1 (MVP)
                        → US2 (needs shipmentId from US1)
                        → US3 (needs shipmentId)
                        → US4 (needs all prior steps)
                        → Polish (T028 abandon QA after US1+ list wired)
```

---

## Parallel Opportunities

- T003 + T004 after T001–T002
- T028–T032 polish tasks parallel after US4

---

**Total tasks**: 34 (T0 + T001–T033) | **US1**: 6 | **US2**: 5 | **US3**: 4 | **US4**: 3 | **MVP scope**: T0 + T001–T015
