# Feature 003: Shipment Creation Wizard

**Status**: ✅ Closed — `v0.2.0` (۱۴۰۵/۰۴/۱۶)  
**Depends on**: `002-shipments`  
**Branch**: `003-shipment-wizard` → merged to `main`

## خلاصه

Wizard چهارمرحله‌ای برای ایجاد محموله جدید از لیست (`/shipments` → «محموله جدید» → `/shipments/new`). جایگزین دکمه غیرفعال `shipment-create-btn` در فیچر 002.

| گام | محتوا | API |
|-----|--------|-----|
| ۱ | مشتری، نوع حمل، مبدأ/مقصد | `POST /api/shipments` → DRAFT |
| ۲ | جزئیات عملیاتی (mode-conditional) | `PATCH /api/shipments/:id` |
| ۳ | Leg اختیاری (حداکثر یکی) | `POST .../legs` یا skip |
| ۴ | بازبینی + ثبت نهایی | redirect به `/shipments/{id}` |

## تحویل‌های کلیدی

- Route: `/shipments/new` داخل `ErpShell`
- کامپوننت: `shipment-wizard.tsx` + reuse `ShipmentLegForm` از `shipment-legs-panel`
- API helpers: `fetchCustomers`, `createShipment`, hooks تان‌استک
- Visual QA: `11-shipment-wizard-desktop.png`, `12-shipment-wizard-mobile.png`
- E2E smoke: `design-reference/wizard-smoke-qa.mjs`

## اسناد

| سند | نقش |
|-----|-----|
| [spec.md](./spec.md) | User stories + FR |
| [plan.md](./plan.md) | معماری و تصمیمات |
| [tasks.md](./tasks.md) | ۳۴ تسک (T0 + T001–T033) |
| [quickstart.md](./quickstart.md) | راهنمای validation + Notes |
| [contracts/](./contracts/) | API + UI testids |

## نکات عملیاتی

- پس از افزودن Leg، بک‌اند ممکن است وضعیت را `BOOKED` کند (`leg-status-sync`) — review وضعیت زنده از API می‌خواند.
- قبل از Visual QA: یک dev server، `.next` تمیز، تأیید HTTP 200 (نه فقط pixel-diff).
