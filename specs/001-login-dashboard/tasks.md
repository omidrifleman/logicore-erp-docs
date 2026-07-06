# Tasks: Login + Dashboard

**Status**: ✅ **Complete** (۱۴۰۵/۰۴/۱۶) — Visual QA passed: login 1.09%, dashboard desktop 1.98%, dashboard mobile 1.55%

**Input**: Design documents from `/specs/001-login-dashboard/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: Visual QA via `design-reference/visual-qa.mjs` (not unit tests for MVP)

**Organization**: Tasks grouped by user story for independent implementation

## Format: `[ID] [P?] [Story] Description`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Scaffold Next.js project and tooling without page implementation

- [x] T001 Create Next.js App Router project with TypeScript at repo root (`src/app/`)
- [x] T002 Install core deps in `package.json`: tailwindcss v4, framer-motion, lucide-react, @tanstack/react-query, recharts
- [x] T003 [P] Configure `src/app/globals.css` with DESIGN-SYSTEM tokens (glass, amber primary, body gradients)
- [x] T004 [P] Configure `src/app/layout.tsx` with Vazirmatn `next/font/google`, `lang="fa" dir="rtl"`
- [x] T005 [P] Create `src/lib/erp-tokens.ts` and `src/lib/format.ts` per design-system.mdc
- [x] T006 [P] Configure `.cursor/mcp.json` with shadcn + playwright MCP servers
- [x] T007 [P] Add shadcn `components.json` targeting `src/components/ui/` and run MCP add: button, input, card, skeleton, toast
- [x] T008 [P] Add `design-reference/visual-qa.mjs` and devDependencies (playwright, pixelmatch, pngjs) in `package.json`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Auth utilities and API proxy — MUST complete before user stories

**⚠️ CRITICAL**: No user story work until this phase is complete

- [x] T009 Implement `src/lib/auth.ts` — session read/write/clear (cookie + localStorage keys per data-model.md)
- [x] T010 [P] Configure `next.config.ts` API rewrite/proxy to backend `:3001` (`/api/*`)
- [x] T011 [P] Create `src/components/shared/error-box.tsx` using erpTokens.errorBox pattern
- [x] T012 [P] Create `src/components/shared/loading-state.tsx` (Loader2 amber centered)
- [x] T013 [P] Create `src/components/shared/empty-state.tsx` (icon + muted text pattern)
- [x] T014 Add auth middleware in `src/middleware.ts` — protect `/`, redirect anonymous to `/login`
- [x] T015 [P] Setup TanStack Query provider in `src/app/providers.tsx` and wrap root layout

**Checkpoint**: Foundation ready — user story implementation can begin

---

## Phase 3: User Story 1 — ورود به سیستم (Priority: P1) 🎯 MVP

**Goal**: فارسی RTL login with email/password, workspace select, error/loading states

**Independent Test**: `/login` → valid credentials → ERP home; invalid → error box

### Implementation for User Story 1

- [x] T016 [US1] Create `src/app/login/page.tsx` — glass-strong card, Ship logo, framer entrance animation
- [x] T017 [US1] Add login form fields with `data-testid="login-email"` and `data-testid="login-password"` (dir="ltr")
- [x] T018 [US1] Implement login submit handler calling `POST /api/auth/login` in `src/lib/auth.ts` or colocated hook
- [x] T019 [US1] Add workspace select UI (conditional on `type: workspace_select`) in `src/app/login/page.tsx`
- [x] T020 [US1] Wire Loading state (Loader2) and Error state (ErrorBox) on login page
- [x] T021 [US1] Redirect to `/` on successful auth; persist session per contracts/auth-dashboard-api.md

**Checkpoint**: Login flow works end-to-end independently

---

## Phase 4: User Story 2 — مشاهده داشبورد KPI (Priority: P2)

**Goal**: KPI cards, revenue chart, status bar — matches `02-dashboard-desktop.png`

**Independent Test**: Authenticated `/` shows `dashboard-view` + `kpi-active-shipments` + period label

### Implementation for User Story 2

- [x] T022 [US2] Create `src/components/erp/management-dashboard.tsx` with KPI glass cards (amber/emerald/sky tones)
- [x] T023 [US2] Add `data-testid="dashboard-view"` and `data-testid="kpi-active-shipments"` markers
- [x] T024 [US2] Implement `useDashboardSummary` query hook fetching `GET /api/dashboard/summary`
- [x] T025 [US2] Add skeleton KPI state while loading in `management-dashboard.tsx`
- [x] T026 [US2] Add error and empty states for dashboard in `management-dashboard.tsx`
- [x] T027 [US2] Integrate recharts area chart + status bar per DESIGN-SYSTEM chart tokens
- [x] T028 [US2] Wire `src/app/(erp)/page.tsx` to render `management-dashboard` inside shell

**Checkpoint**: Dashboard displays real KPI data with all async states

---

## Phase 5: User Story 3 — ناوبری Shell و واکنش‌گرایی (Priority: P3)

**Goal**: erp-shell header/sidebar, mobile layout, logout, auth guard

**Independent Test**: Nav active amber state; mobile Pixel 5 layout; logout clears session

### Implementation for User Story 3

- [x] T029 [US3] Create `src/components/erp/erp-shell.tsx` — glass-strong header h-14, sidebar w-14/md:w-56
- [x] T030 [US3] Add nav items with active/inactive erpTokens styles; dashboard as default active on `/`
- [x] T031 [US3] Add tenant chip and user avatar in header per DESIGN-SYSTEM
- [x] T032 [US3] Implement logout action clearing session and redirecting to `/login`
- [x] T033 [US3] Apply responsive padding `p-4 md:p-6` and KPI grid `xl:grid-cols-4` in dashboard
- [x] T034 [US3] Add framer-motion view transition wrapper (opacity/y 0.2s) on page content in shell

**Checkpoint**: Full shell + responsive dashboard matches mobile reference

---

## Phase 6: Polish & Visual QA

**Purpose**: Cross-cutting validation before merge

- [x] T035 [P] Run `node design-reference/capture-screenshots.mjs` — verify `01-login-desktop.png`, `02-dashboard-desktop.png`, `09-dashboard-mobile.png`
- [x] T036 [P] Run `node design-reference/visual-qa.mjs --pages login,dashboard` — fix spacing/color/font diffs
- [x] T037 Validate all DESIGN-SYSTEM checklist items from `.cursor/rules/design-system.mdc`
- [x] T038 Run `specs/001-login-dashboard/quickstart.md` manual validation checklist
- [x] T039 Document any intentional visual deviations in `specs/001-login-dashboard/quickstart.md` Notes

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 — BLOCKS all user stories
- **US1 (Phase 3)**: Depends on Phase 2
- **US2 (Phase 4)**: Depends on Phase 2 + US1 (needs auth to view dashboard)
- **US3 (Phase 5)**: Depends on Phase 2; integrates with US1/US2
- **Polish (Phase 6)**: Depends on US1–US3

### Parallel Opportunities

- T003–T008 can run in parallel after T001–T002
- T010–T013 can run in parallel in Phase 2
- T035–T036 can run in parallel in Phase 6

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 + Phase 2
2. Complete Phase 3 (Login)
3. **STOP** — validate login independently
4. Proceed to US2/US3 only after user approves `/speckit-implement` scope

### Suggested Execution Order (post-approval)

```
T001→T002 → T003-T008 (parallel) → T009-T015 → T016-T021 (US1) → T022-T028 (US2) → T029-T034 (US3) → T035-T039
```

**Total tasks**: 39 | **US1**: 6 | **US2**: 7 | **US3**: 6 | **MVP scope**: Phase 1–3 (T001–T021)

---

## Notes

- Do NOT implement Shipments/Finance/CRM modules in this feature
- Use shadcn MCP for T007 — not manual component copies
- `/speckit-implement` requires explicit user approval before starting T016+
- **Intentional deviation (T022)**: KPI icons natural-size بدون دایره رنگی — مرجع `02-dashboard-desktop.png` دایره دارد؛ طبق تصمیم Phase 4 حفظ شد
- **Content fix (T036)**: بازه تاریخ به فرمت ISO `YYYY-MM-DD تا YYYY-MM-DD` هم‌تراز مرجع capture شد
