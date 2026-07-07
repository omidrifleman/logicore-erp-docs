# Project Backlog

موارد زیر **خارج از scope** فیچرهای تکمیل‌شده هستند و در تیکت/فیچر جداگانه پیگیری می‌شوند.

---

## BL-001 — Unit test بک‌اند: `shipment.controller.ts`

| فیلد | مقدار |
|------|--------|
| **منبع** | خروجی T0 / فیچر `002-shipments` (۱۴۰۵/۰۴/۱۶) |
| **اولویت** | Medium |
| **وضعیت** | Open |
| **Repo** | `Shipping-Project-V2/production-ready/nestjs-backend` |

**شرح:** Unit test اختصاصی برای `shipment.controller.ts` هنوز نوشته نشده است. e2e موجود کافی نیست؛ باید در یک تیکت جداگانه پوشش unit برای endpointهای CRUD، transition، و legs اضافه شود.

---

## BL-002 — `.env` رسمی: `DATABASE_URL` / `WORKER_DATABASE_URL`

| فیلد | مقدار |
|------|--------|
| **منبع** | T0 local dev / `specs/002-shipments/quickstart.md` |
| **اولویت** | High (قبل از deploy) |
| **وضعیت** | Open |
| **Repo** | `Shipping-Project-V2/production-ready/` |

**شرح:** در محیط local، `production-ready/.env` به‌صورت پیش‌فرض `DATABASE_URL` و `WORKER_DATABASE_URL` ندارد و باید دستی ست شوند (مطابق quickstart). **قبل از هر دیپلوی staging/production** باید `.env` رسمی (یا secret manager) به‌روزرسانی و مستند شود تا بوت backend بدون تنظیم دستی ممکن باشد.

**اقدام پیشنهادی:**
- افزودن متغیرها به `.env.example` با placeholder واضح
- چک‌لیست pre-deploy در CI یا runbook ops

---

## BL-003 — Known issue (non-blocking): Visual QA موبایل `shipments-mobile` — 5.05%

| فیلد | مقدار |
|------|--------|
| **منبع** | Phase 7 T040 / `specs/002-shipments/quickstart.md` Notes |
| **اولویت** | Low |
| **وضعیت** | Known issue — non-blocking |
| **فیچر مرجع** | `002-shipments` (تأیید نهایی با انحراف مشروط) |

**شرح:** `shipments-mobile` در Visual QA **5.05%** pixel diff دارد (۰.۰۵٪ بالای آستانه ۵٪). layout موبایل (۳ ستون، nav `w-14`، بدون overflow) تأیید DOM شده؛ اختلاف عمدتاً **محتوای داینامیک API** است نه layout.

**راه‌حل پیشنهادی (فاز بعدی — مثلاً همراه با ویرایش عملیاتی):**
1. **Mock seed ثابت** برای capture مرجع `10-shipments-mobile.png` و اجرای visual-qa
2. **Mask ناحیه جدول** در `visual-qa.mjs` برای نادیده‌گرفتن سلول‌های داده‌محور

**معیار بسته‌شدن:** diff ≤ 5% پس از یکی از راه‌حل‌های بالا، یا تأیید صریح انحراف دائمی در design sign-off.

---

## BL-004 — `npm run capture` کامل: `finance-tab-match` حذف‌شده از UI

| فیلد | مقدار |
|------|--------|
| **منبع** | Polish T051 / `design-reference/capture-screenshots.mjs` — release `004-shipment-documents` (۱۴۰۵/۰۴/۱۷) |
| **اولویت** | Low |
| **وضعیت** | Open — non-blocking |
| **فیچر مرجع** | خارج از scope `004-shipment-documents` (004 بسته شد) |
| **Repo** | `Shipping` (`design-reference/capture-screenshots.mjs`) |

**شرح:** اسکریپت `npm run capture` پس از گرفتن `13-shipment-documents-desktop.png` منتظر `getByTestId('finance-tab-match')` می‌ماند؛ این testid در UI فعلی ERP وجود ندارد (ماژول finance بازطراحی شده). capture کامل روی finance/invoices/CRM متوقف می‌شود؛ PNGهای ۱۳–۱۵ و visual-qa مربوطه جداگانه گرفته و پاس شده‌اند.

**اقدام پیشنهادی:**
1. به‌روزرسانی `capture-screenshots.mjs` با selectorهای فعلی finance (یا ناوبری مستقیم به route)
2. اجرای مجدد capture کامل برای baselineهای `06-finance` تا `08-crm`

**معیار بسته‌شدن:** `npm run capture` بدون timeout از ابتدا تا `=== capture complete ===`.

---

*آخرین به‌روزرسانی: ۱۴۰۵/۰۴/۱۷ — پس از release `004-shipment-documents` (v0.3.0 / v0.2.0)*
