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

*آخرین به‌روزرسانی: ۱۴۰۵/۰۴/۱۶ — پس از close رسمی `002-shipments`*
