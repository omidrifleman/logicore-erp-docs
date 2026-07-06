# CHANGES — 004-shipment-documents

**Feature**: Shipment Documents (BL / Invoice / Packing List)  
**Date**: 2026-07-07

این فایل انحراف‌های **عمدی** از spec/plan اولیه را مستند می‌کند.

---

## 1. صفحه عمومی تأیید اصالت `/verify/[id]` (افزوده پس از F6)

| مورد | plan/spec اولیه | پیاده‌سازی نهایی |
|------|----------------|------------------|
| مقصد QR | `GET /api/verify/:id` (JSON خام) | `{origin}/verify/:id` — صفحه HTML برنددار Next.js |
| API | همان endpoint عمومی | صفحه از `/api/verify/:id` داده می‌خواند (proxy به NestJS) |
| PDF QR | `{NEXTAUTH_URL}/api/verify/{id}` | `{NEXTAUTH_URL}/verify/{id}` (تغییر `verificationBaseUrl` در backend) |

**دلیل:** اسکن QR توسط طرف سوم (گمرک، بانک) نباید JSON خام نشان دهد.  
**توجه:** PDFهای finalize‌شده **قبل** deploy این تغییر هنوز QR قدیمی (`/api/verify/...`) دارند.

---

## 2. testidهای QR

| contract اولیه | کد نهایی |
|----------------|----------|
| `document-qr-preview` | `document-qr-code` + `document-verify-link` |

---

## 3. amend — بدون UI (طبق spec)

- `POST /api/documents/:id/amend` در backend پیاده‌سازی شده
- **هیچ دکمه amend** در ERP v1 — پیام read-only به کاربر ارجاع می‌دهد

---

## 4. docNumber

- فقط در **finalize** تخصیص می‌شود: `BL-000001`, `INV-000001`, `PL-000001` (per-tenant per-type)
- DRAFT همیشه `docNumber: null`

---

## 5. اطلاعات صفحه verify

نمایش داده می‌شود: نوع سند، شماره، تاریخ صدور، صادرکننده (نام حقوقی tenant)، SHA-256، وضعیت معتبر/نامعتبر.

**عمداً نمایش داده نمی‌شود:** شماره محموله، قیمت، نام مشتری، طرفین تجاری — مطابق محدودیت امنیتی API عمومی.

---

## 6. نسخه‌گذاری پیشنهادی (tag)

| Repo | Tag پیشنهادی | یادداشت |
|------|--------------|---------|
| `logicore-erp` (frontend) | **v0.3.0** | `package.json` به‌روز شد |
| `logicore-erp-backend` | **v0.2.0** | ماژول document + migration + PDF templates |

---

## 7. T0 نتایج

- Backend E2E `documents-api.spec.ts`: **12/12 پاس** روی Docker Postgres (branch `004-shipment-documents`)
- Frontend: `npm run build` — بدون خطای TypeScript (Polish T053)
