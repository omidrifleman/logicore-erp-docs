# مرجع بصری LogiCore

اسکرین‌شات‌های مرجع با اسکریپت زیر گرفته می‌شوند.

## پیش‌نیاز

1. Postgres (Docker e2e)
2. Backend روی `:3001`
3. Frontend روی `:3000`

```bash
# ترمینال ۱ — backend
cd production-ready/nestjs-backend && npm run start

# ترمینال ۲ — frontend
$env:BACKEND_URL="http://localhost:3001"; npx next dev -p 3000 --webpack

# ترمینال ۳ — capture
node design-reference/capture-screenshots.mjs
```

اسکریپت خودش از API بک‌اند login می‌کند (بدون نیاز به `e2e/.auth/sirrah.json`).

## نکات capture

- قبل از هر عکس، منتظر **marker محتوا** (KPI، ردیف جدول، تب مالی و …) می‌ماند — نه فقط shell/nav.
- پاسخ API (`/api/dashboard/summary`, `/api/shipments`, …) را await می‌کند.
- انیمیشن Framer Motion (opacity:0) را با `MutationObserver` + `reducedMotion` خنثی می‌کند تا عکس خالی نگیرد.

فایل‌های خروجی:

| فایل | صفحه | viewport |
|------|------|----------|
| `01-login-desktop.png` | لاگین | 1280×800 |
| `02-dashboard-desktop.png` | داشبورد KPI | 1280×800 |
| `03-shipments-list-desktop.png` | لیست محموله | 1280×800 |
| `04-shipment-detail-desktop.png` | جزئیات محموله | 1280×800 |
| `05-shipment-legs-desktop.png` | پنل Leg | 1280×800 |
| `06-finance-desktop.png` | مالی / تطبیق | 1280×800 |
| `07-invoices-desktop.png` | فاکتورها | 1280×800 |
| `08-crm-desktop.png` | مشتریان | 1280×800 |
| `09-dashboard-mobile.png` | داشبورد | Pixel 5 |
| `10-shipments-mobile.png` | لیست محموله | Pixel 5 |

**وضعیت فعلی (۱۴۰۵/۰۴/۱۵):** هر ۱۰ اسکرین‌شات با محتوای واقعی (KPI، جدول، نمودار) گرفته شد.

> **یادداشت:** Docker + backend + frontend باید قبل از capture در حال اجرا باشند.