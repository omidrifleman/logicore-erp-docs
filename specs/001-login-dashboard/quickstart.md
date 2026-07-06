# Quickstart: Login + Dashboard Validation

**Feature**: 001-login-dashboard

## Prerequisites

1. Node.js 18+ and npm
2. Backend on `:3001` (or mock)
3. Frontend dev server on `:3000`

## Setup

```bash
# Install dependencies (after Next.js scaffold)
npm install

# Terminal 1 — backend
cd production-ready/nestjs-backend && npm run start

# Terminal 2 — frontend
$env:BACKEND_URL="http://localhost:3001"
npm run dev
```

## Manual Validation

### Login (US1)

1. Open `http://localhost:3000/login`
2. Enter `maryam@logicore.com` / `maryam123`
3. If workspace picker appears → select «سیر راه آبی»
4. Expect redirect to `/` with ERP shell visible
5. Invalid password → rose error box, form remains

### Dashboard (US2)

1. After login, confirm `data-testid="dashboard-view"` visible
2. Confirm `data-testid="kpi-active-shipments"` has numeric value
3. Confirm «بازه:» period label in main content
4. Throttle network → skeleton appears, then content

### Mobile (US3)

1. DevTools → Pixel 5 viewport
2. Dashboard KPI grid readable; nav `w-14` icon-only
3. Compare with `design-reference/09-dashboard-mobile.png`

## Visual QA (automated)

```bash
# Capture current screenshots
node design-reference/capture-screenshots.mjs

# Compare against reference (login + dashboard only)
node design-reference/visual-qa.mjs --pages login,dashboard
```

**Pass criteria**: Diff report ≤5% perceptual mismatch on layout regions; no missing testids.

## Logout

1. Trigger logout from header/menu
2. Confirm redirect to `/login` and cookies cleared

## Notes — intentional deviations (Visual QA)

| مورد | مرجع | پیاده‌سازی | دلیل |
|------|------|-----------|------|
| KPI icon background | دایره رنگی `w-9 rounded-xl` در `02-dashboard-desktop.png` | آیکون natural-size `w-4` بدون دایره | تصمیم Phase 4 / design-system |
| بازه تاریخ (اصلاح‌شده) | `2026-06-05 تا 2026-07-05` (ISO) | ابتدا Jalali — اصلاح به ISO | هم‌ترازی محتوایی با capture |
| center-card regional diff ~10.5% | — | باقی‌مانده: رندر recharts/SVG + آیکون KPI | diff کل 1.98% زیر آستانه — قابل قبول |
