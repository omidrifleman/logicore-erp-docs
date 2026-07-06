# Feature Specification: Shipment Documents (Document Management)

**Feature Branch**: `004-shipment-documents`

**Created**: 2026-07-06

**Status**: Draft — tasks ready 2026-07-06 (گزینه B: backend + frontend)

**Input**: User description: «مدیریت اسناد حمل از صفحه جزئیات محموله — مشاهده، ایجاد، نهایی‌سازی (finalize)، دانلود PDF، و مشاهده QR برای تأیید اصالت. انواع MVP: Bill of Lading، Invoice، Packing List.»

**Governance**: این spec تابع [Constitution v1.0.0](.specify/memory/constitution.md) است. الزامات Design System از `.cursor/rules/design-system.mdc` و `.cursor/rules/component-structure.mdc` ارجاع داده می‌شوند.

**Depends on**: `002-shipments` (تب Documents در `shipment-detail.tsx` — placeholder)، `003-shipment-wizard` (ایجاد محموله)

---

## Backend Readiness Assessment *(بررسی زمینه‌ای — ۲۰۲۶-۰۷-۰۶)*

> این بخش نتیجه بررسی `Shipping-Project-V2/production-ready/nestjs-backend` است و محدودیت‌های مهم برای فاز plan را ثبت می‌کند.

### آنچه در بک‌اند **حاضر** است

| قابلیت | وضعیت |
|--------|--------|
| ماژول `src/modules/document/` | ✅ پیاده‌سازی شده |
| `GET /api/documents` (فیلتر `shipmentId`, `type`, pagination) | ✅ |
| `GET /api/documents/:id` (جزئیات + access logs) | ✅ |
| `POST /api/documents/upload` (آپلود فایل multipart) | ✅ |
| `GET /api/documents/:id/download` (با audit + integrity hash) | ✅ |
| `DELETE /api/documents/:id` (فقط غیرقفل) | ✅ |
| `GET /api/verify/:documentId` (QR عمومی — بدون auth) | ✅ |
| `POST /api/documents/generate-locked-pdf` | ✅ فقط نوع `QUOTE_COMMITMENT` |
| ذخیره فایل | ✅ local filesystem یا MinIO/S3 (`StorageService`) |
| تولید PDF + QR | ✅ `pdf-lib` + `qrcode` — فقط قالب commitment |
| Prisma `Document` + `DocumentType` enum | ✅ شامل `BILL_OF_LADING`, `COMMERCIAL_INVOICE`, `PACKING_LIST` |
| نسخه‌بندی schema (`supersedesId`) | ✅ در schema — **بدون API amend** |
| E2E smoke | ✅ `e2e/tests/documents-api.spec.ts` (لیست pagination) |

### آنچه با طراحی اولیه `SPEC.md` §۴ **مطابقت ندارد** یا **وجود ندارد**

| طراحی اولیه (`SPEC.md`) | وضعیت فعلی بک‌اند |
|-------------------------|-------------------|
| `direction`, `language`, `status` (DRAFT/FINALIZED) | ❌ فیلد جداگانه ندارد — «قفل» با `watermarkType` / `isWatermarked` تقریب می‌شود |
| `data JSONB` (فیلدهای فرم سند) | ❌ وجود ندارد — فقط فایل ذخیره‌شده |
| `POST /documents` (ایجاد پیش‌نویس با داده ساخت‌یافته) | ❌ |
| `POST /documents/:id/finalize` | ❌ |
| `POST /documents/:id/amend` | ❌ |
| `GET /documents/:id/pdf` (تولید on-demand) | ❌ — فقط download فایل موجود |
| `GET /documents/:id/qr` | ❌ — QR فقط داخل PDF commitment + verify عمومی |
| قالب PDF BL/Invoice/PL (Handlebars + Playwright) | ❌ — فقط PDF برنامه‌نویسی‌شده commitment |
| `signedHash`, `qrCodeData` جدا | ❌ — `fileHash` (SHA-256) وجود دارد |

### پیامد برای این فیچر

- **فرانت‌اند** (`Shipping/`): هیچ `src/lib/documents.ts` یا hook مربوطه وجود ندارد؛ تب «اسناد» placeholder است.
- **مرجع قابل پورت**: `Shipping-Project-V2/src/lib/api.ts` (`documentApi`) و hooks در `use-logicore-queries.ts`.
- **برای MVP کامل US2–US5** (ایجاد فرم‌محور + finalize + PDF تولیدی + QR): **نیاز به گسترش بک‌اند** یا **تصمیم scope محدود v1** (لیست + آپلود + دانلود فایل خارجی) در فاز plan/T0.
- این محدودیت در Assumptions و FR-Backend ثبت شده و در `/speckit-plan` با T0 Live Verification باید قطعی شود.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - مشاهده لیست اسناد محموله (Priority: P1)

اپراتور در صفحه جزئیات محموله (`/shipments/{id}`) تب «اسناد» را باز می‌کند و فهرست تمام اسناد مرتبط با آن محموله را می‌بیند: نوع سند، شماره سند (در صورت وجود)، نام فایل، وضعیت (پیش‌نویس / نهایی‌شده)، تاریخ ایجاد، و تعداد دسترسی‌ها.

**Why this priority**: بدون لیست، هیچ نقطه ورودی به مدیریت اسناد وجود ندارد؛ جایگزین مستقیم placeholder فعلی در فیچر 002.

**Independent Test**: باز کردن جزئیات محموله seed → تب اسناد → نمایش لیست (حتی خالی) با loading/error/empty states استاندارد ERP.

**Acceptance Scenarios**:

1. **Given** اپراتور در جزئیات محموله است، **When** تب «اسناد» (`shipment-tab-documents`) را می‌زند، **Then** پنل `shipment-documents-panel` با عنوان و لیست/empty state نمایش داده می‌شود.
2. **Given** محموله حداقل یک سند دارد، **When** تب اسناد باز است، **Then** هر ردیف نوع سند (برچسب فارسی)، `docNumber`، تاریخ، و badge وضعیت (پیش‌نویس/نهایی) را نشان می‌دهد.
3. **Given** API لیست در حال بارگذاری است، **When** تب اسناد باز می‌شود، **Then** Loader2 amber مرکز‌چین نمایش داده می‌شود.
4. **Given** API خطا می‌دهد، **When** تب اسناد باز است، **Then** باکس خطای rose استاندارد ERP با متن فارسی نمایش داده می‌شود.
5. **Given** محموله بدون سند است، **When** تب اسناد باز است، **Then** empty state با آیکون `FileText` و متن «سندی ثبت نشده» + دکمه «سند جدید» نمایش داده می‌شود.

---

### User Story 2 - ایجاد سند جدید (Priority: P2)

اپراتور از تب اسناد دکمه «سند جدید» را می‌زند، نوع سند را انتخاب می‌کند (Bill of Lading، Commercial Invoice، Packing List)، فیلدهای مرتبط با نوع را پر می‌کند (با prefill از داده محموله: مشتری، vessel/voyage، بنادر، شرح کالا)، و سند را به‌عنوان **پیش‌نویس** ذخیره می‌کند.

**Why this priority**: ایجاد سند هسته ارزش عملیاتی است؛ وابسته به US1 برای context محموله.

**Independent Test**: انتخاب نوع BL → پر کردن فیلدهای اجباری → ذخیره → سند در لیست US1 با وضعیت پیش‌نویس ظاهر می‌شود.

**Acceptance Scenarios**:

1. **Given** تب اسناد باز است، **When** «سند جدید» (`document-create-btn`) زده می‌شود، **Then** مودال/پنل `document-form` با انتخاب نوع سند (`document-type-select`) نمایش داده می‌شود.
2. **Given** نوع `BILL_OF_LADING` انتخاب شده، **When** فرم باز است، **Then** فیلدهای shipper/consignee/notify، vessel/voyage، بنادر، شرح کالا نمایش و vessel/voyage/بنادر از محموله prefill می‌شوند.
3. **Given** نوع `COMMERCIAL_INVOICE` انتخاب شده، **When** فرم باز است، **Then** فیلدهای مبلغ/ارز/شرح کالا/طرف حساب نمایش داده می‌شوند.
4. **Given** نوع `PACKING_LIST` انتخاب شده، **When** فرم باز است، **Then** فیلدهای بسته‌بندی (تعداد بسته، وزن، حجم) نمایش داده می‌شوند.
5. **Given** فیلد اجباری خالی است، **When** ذخیره زده می‌شود، **Then** اعتبارسنجی فارسی و عدم submit.
6. **Given** ذخیره موفق، **When** پاسخ API برمی‌گردد، **Then** سند در لیست با وضعیت پیش‌نویس ظاهر و فرم بسته می‌شود.

---

### User Story 3 - نهایی‌سازی سند (Finalize) (Priority: P3)

اپراتور یک سند پیش‌نویس را «نهایی» می‌کند: سیستم شماره رسمی سند تخصیص می‌دهد، PDF تولید و ذخیره می‌کند، hash یکپارچگی ثبت می‌شود، سند **غیرقابل ویرایش** می‌شود، و QR تأیید اصالت به PDF اضافه می‌شود.

**Why this priority**: finalize ارزش قانونی/عملیاتی سند را فعال می‌کند؛ وابسته به US2.

**Independent Test**: سند پیش‌نویس BL → finalize → وضعیت نهایی + شماره رسمی + عدم امکان ویرایش/حذف.

**Acceptance Scenarios**:

1. **Given** سند در وضعیت پیش‌نویس است، **When** اپراتور «نهایی‌سازی» (`document-finalize-btn`) را می‌زند و تأیید می‌کند، **Then** سند به وضعیت نهایی تغییر می‌کند و `docNumber` رسمی تخصیص می‌یابد.
2. **Given** finalize موفق، **When** سند نهایی است، **Then** فیلدهای فرم غیرقابل ویرایش و دکمه حذف مخفی/غیرفعال است.
3. **Given** finalize موفق، **When** PDF تولید شده، **Then** فایل PDF با watermark/QR در storage ذخیره و `fileHash` ثبت می‌شود.
4. **Given** API finalize خطا می‌دهد، **When** عملیات انجام می‌شود، **Then** باکس خطای rose و سند در وضعیت پیش‌نویس باقی می‌ماند.
5. **Given** سند از قبل نهایی است، **When** کاربر تلاش به ویرایش مستقیم می‌کند، **Then** پیام «سند نهایی‌شده قابل ویرایش نیست — از اصلاح (amend) استفاده کنید» (amend در v1 خارج از scope است).

---

### User Story 4 - دانلود PDF (Priority: P4)

اپراتور از لیست یا جزئیات سند، فایل PDF سند را دانلود می‌کند. برای اسناد نهایی‌شده، فایل تولیدشده سیستم؛ برای پیش‌نویس‌های آپلودشده، فایل اصلی.

**Why this priority**: خروجی عملیاتی اصلی برای مشتری و عملیات؛ وابسته به وجود سند (US2/US3).

**Independent Test**: کلیک دانلود روی سند نهایی → فایل PDF با نام معنادار در مرورگر ذخیره می‌شود.

**Acceptance Scenarios**:

1. **Given** سند دارای فایل است، **When** «دانلود» (`document-download-btn`) زده می‌شود، **Then** فایل PDF با نام `{docNumber}_{type}.pdf` (یا `fileName`) دانلود می‌شود.
2. **Given** دانلود موفق، **When** عملیات کامل می‌شود، **Then** رویداد دسترسی (DOWNLOAD) در audit log سند ثبت می‌شود.
3. **Given** integrity check شکست می‌خورد، **When** دانلود درخواست می‌شود، **Then** خطای فارسی «خطای یکپارچگی فایل» نمایش داده می‌شود.

---

### User Story 5 - مشاهده QR و تأیید اصالت (Priority: P5)

اپراتور برای سند نهایی‌شده QR code را مشاهده می‌کند (در UI یا داخل PDF). اسکن QR به صفحه/endpoint عمومی تأیید هدایت می‌کند که نوع سند، شماره، hash، تاریخ صدور، و اعتبار را **بدون اطلاعات حساس** نشان می‌دهد.

**Why this priority**: تأیید اصالت ارزش افزوده امنیتی؛ وابسته به US3.

**Independent Test**: سند نهایی → نمایش QR در پنل جزئیات → اسکن/باز کردن URL verify → نمایش `valid: true`.

**Acceptance Scenarios**:

1. **Given** سند نهایی‌شده است، **When** جزئیات سند باز می‌شود، **Then** QR code (`document-qr-preview`) و لینک verify نمایش داده می‌شود.
2. **Given** URL verify عمومی فراخوانی می‌شود، **When** `documentId` معتبر است، **Then** پاسخ شامل `type`, `docNumber`, `sha256`, `issuedAt`, `valid` است — **بدون** مسیر فایل یا نام مشتری.
3. **Given** `documentId` نامعتبر است، **When** verify فراخوانی می‌شود، **Then** 404 با پیام فارسی/انگلیسی مناسب.

---

### Edge Cases

- محموله **CLOSED**: ایجاد سند جدید ممنوع یا فقط با تأیید صریح (در plan مشخص می‌شود؛ پیش‌فرض: ممنوع).
- **دو سند هم‌نوع** برای یک محموله: هشدار غیرمسدودکننده + امکان ادامه (طبق `SPEC.md` §۵.۳).
- **آپلود فایل بزرگ** (>۵۰MB): خطای فارسی از API.
- **قطع شبکه** هنگام finalize: سند در پیش‌نویس بماند؛ امکان retry.
- **فونت فارسی در PDF**: در صورت متن فارسی در قالب، embed فونت اجباری است (محدودیت plan/backend).
- **سند قفل‌شده (locked)**: حذف ممنوع — پیام خطای API نمایش داده شود.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: سیستم MUST لیست اسناد یک محموله را در تب «اسناد» جزئیات محموله نمایش دهد.
- **FR-002**: سیستم MUST از سه نوع سند MVP پشتیبانی کند: Bill of Lading، Commercial Invoice، Packing List.
- **FR-003**: اپراتور MUST بتواند سند جدید از نوع‌های MVP با فیلدهای نوع‌محور ایجاد کند؛ داده محموله برای prefill استفاده شود.
- **FR-004**: سیستم MUST وضعیت سند را به‌صورت پیش‌نویس یا نهایی تمایز دهد و پس از نهایی‌سازی ویرایش مستقیم را مسدود کند.
- **FR-005**: سیستم MUST هنگام نهایی‌سازی شماره رسمی سند (`docNumber`) تخصیص دهد و PDF تولید کند.
- **FR-006**: سیستم MUST SHA-256 hash فایل نهایی را ذخیره و هنگام دانلود یکپارچگی را بررسی کند.
- **FR-007**: سیستم MUST QR code تأیید اصالت در PDF نهایی و در UI جزئیات سند نمایش دهد.
- **FR-008**: اپراتور MUST بتواند PDF سند را دانلود کند؛ هر دانلود در audit log ثبت شود.
- **FR-009**: UI MUST loading، error، و empty states استاندارد ERP را برای همه viewهای async رعایت کند.
- **FR-010**: UI MUST از Design System (glass، erpTokens، statusPalettes) و RTL فارسی پیروی کند.

### Backend Constraints *(ثبت‌شده از بررسی زمینه‌ای)*

- **FR-B01**: اگر endpointهای `finalize` / `POST documents` (پیش‌نویس ساخت‌یافته) / قالب PDF نوع‌محور در T0 موجود نباشند، plan MUST یا **گسترش بک‌اند** را پیش‌نیاز کند یا **scope v1** را به لیست + آپلود + دانلود محدود کند — با تأیید صریح کاربر.
- **FR-B02**: endpoint عمومی `GET /api/verify/:documentId` موجود است و برای US5 قابل استفاده است.
- **FR-B03**: `POST /api/documents/upload` برای آپلود فایل خارجی موجود است و می‌تواند مسیر موقت v1 باشد تا finalize پیاده شود.

### Key Entities

- **Shipment Document**: سند مرتبط با یک محموله؛ نوع (`BILL_OF_LADING` | `COMMERCIAL_INVOICE` | `PACKING_LIST`)، شماره رسمی، وضعیت (پیش‌نویس/نهایی)، فایل PDF، hash یکپارچگی، تاریخ ایجاد/نهایی‌سازی.
- **Document Form Data**: داده‌های ساخت‌یافته فرم (shipper، consignee، بنادر، شرح کالا، مبالغ، بسته‌بندی) — در طراحی اولیه JSONB؛ در بک‌اند فعلی **هنوز ذخیره نمی‌شود**.
- **Document Access Log**: ردیابی CREATE / DOWNLOAD / VIEW برای audit.

---

## Out of Scope (v1)

- امضای دیجیتال واقعی (PKI / certificate-based signing)
- ارسال خودکار ایمیل سند به مشتری یا نماینده
- ویرایش مستقیم سند پس از finalize (اصلاح باید از flow جداگانه **amend** باشد — amend خودش در v1 خارج از scope)
- انواع سند خارج از BL / Invoice / Packing List (مثلاً Manifest، AWB، CMR)
- مدیریت قالب سند (لوگو، سربرگ، مهر سفارشی)
- پرتال مشتری (دانلود سند از Customer Portal)
- امضای Canvas روی سند

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: اپراتور می‌تواند لیست اسناد یک محموله را در کمتر از ۵ ثانیه پس از باز کردن تب «اسناد» ببیند (با اتصال عادی).
- **SC-002**: ایجاد سند BL با prefill از محموله در کمتر از ۳ دقیقه completable است (بدون تایپ مجدد vessel/voyage/بنادر).
- **SC-003**: پس از finalize، ۱۰۰٪ اسناد نهایی QR قابل اسکن و verify با `valid: true` دارند.
- **SC-004**: ۱۰۰٪ دانلودهای PDF موفق، hash یکپارچگی را pass می‌کنند.
- **SC-005**: placeholder «مدیریت اسناد در فاز بعدی» در `shipment-detail.tsx` با UI کامل جایگزین می‌شود.

---

## Assumptions

- اپراتور قبلاً login کرده و tenant scope فعال است (همان الگوی 002/003).
- بک‌اند NestJS روی `:3001` در دسترس است؛ T0 در plan باید endpointهای واقعی را با این spec تطبیق دهد.
- انواع سند MVP با enum `DocumentType` بک‌اند هم‌نام هستند (`BILL_OF_LADING`, `COMMERCIAL_INVOICE`, `PACKING_LIST`).
- وضعیت «نهایی» در v1 می‌تواند با `isWatermarked`/`watermarkType` یا فیلد status آینده map شود — جزئیات در plan/T0.
- مرجع UI/UX: تب overview و legs در `shipment-detail.tsx`؛ الگوی لیست/مودال مشابه finance و wizard.
- `Shipping-Project-V2/src/lib/api.ts` (`documentApi`) مرجع قرارداد API اولیه برای plan است، نه تضمین کامل MVP.

---

## UI Contract *(برای agent-driven delivery — هم‌راستا با 002/003)*

| testid | المان |
|--------|--------|
| `shipment-tab-documents` | تب اسناد (موجود) |
| `shipment-documents-panel` | پنل اصلی تب |
| `document-create-btn` | ایجاد سند جدید |
| `document-list` | جدول/لیست اسناد |
| `document-row-{id}` | ردیف سند |
| `document-type-select` | انتخاب نوع در فرم |
| `document-form` | فرم ایجاد/ویرایش پیش‌نویس |
| `document-finalize-btn` | نهایی‌سازی |
| `document-download-btn` | دانلود PDF |
| `document-qr-preview` | پیش‌نمایش QR |
| `document-detail-drawer` | جزئیات سند (اختیاری) |
