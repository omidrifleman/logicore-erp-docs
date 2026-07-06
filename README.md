# LogiCore ERP — Public Documentation

مستندات عمومی پروژه LogiCore ERP (Speckit specs، SPEC مرجع، Design System).

**این ریپو فقط documentation است** — بدون سورس‌کد اپلیکیشن.

| ریپوی کد | GitHub | نقش |
|----------|--------|-----|
| Frontend | [logicore-erp-frontend](https://github.com/omidrifleman/logicore-erp-frontend) | Next.js ERP UI (private) |
| Backend | [logicore-erp-backend](https://github.com/omidrifleman/logicore-erp-backend) | NestJS API (private) |
| **Docs** | **این ریپو (public)** | Specs + مراجع طراحی |

## ساختار

```text
specs/              # Speckit feature specs (001, 002, 003, 004, …)
SPEC.md             # Master product spec (از frontend)
DESIGN-SYSTEM.md    # Design tokens reference
README.frontend.md  # README پروژه frontend
design-reference/   # اسکرین‌شات‌های UI (PNG) — اختیاری
SOURCES.md          # گزارش آخرین sync (تولید خودکار)
```

## Sync

از ریپوی frontend (`Shipping/`) اجرا کنید:

```powershell
# Windows
.\scripts\sync-docs.ps1

# Git Bash / macOS / Linux
./scripts/sync-docs.sh
```

Checkpointها: پایان `/speckit-specify`, `/speckit-plan`, `/speckit-tasks`, و پایان phaseهای بزرگ (B*, F*).

## لینک raw (مثال فیچر 004)

```text
https://raw.githubusercontent.com/omidrifleman/logicore-erp-docs/main/specs/004-shipment-documents/spec.md
https://raw.githubusercontent.com/omidrifleman/logicore-erp-docs/main/specs/004-shipment-documents/plan.md
https://raw.githubusercontent.com/omidrifleman/logicore-erp-docs/main/specs/004-shipment-documents/tasks.md
```
