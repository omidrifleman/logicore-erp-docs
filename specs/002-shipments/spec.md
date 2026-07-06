# Feature Specification: Shipments Management

**Feature Branch**: `002-shipments`

**Created**: 2026-07-06

**Status**: Complete ✅ (closed ۱۴۰۵/۰۴/۱۶ — ۴۴/۴۴ tasks, Phase 7 approved)

**Input**: User description: «ماژول مدیریت محموله‌ها — لیست، جزئیات با Timeline، پنل Leg، نسخه موبایل — مطابق مراجع بصری design-reference و بخش ۵.۱ Shipment Core در SPEC.md»

**Governance**: این spec تابع [Constitution v1.0.0](.specify/memory/constitution.md) است — بدون تغییر در constitution. الزامات Design System از `.cursor/rules/design-system.mdc` و `.cursor/rules/component-structure.mdc` ارجاع داده می‌شوند.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - لیست محموله‌ها (Priority: P1)

اپراتور داخلی پس از ورود به ERP، از منوی «محموله‌ها» فهرست پرونده‌های حمل را می‌بیند:
خلاصه آماری (تعداد کل، در ترانزیت، نیازمند توجه)، جستجوی متنی، فیلتر وضعیت، جدول
شیشه‌ای با ستون‌های شماره/مشتری/وضعیت/نوع/مسیر/کشتی/ETA/کانتینر/اسناد و اقدامات
مشاهده/حذف — مطابق `design-reference/03-shipments-list-desktop.png`.

**Why this priority**: لیست نقطه ورود اصلی ماژول عملیات حمل است؛ بدون آن هیچ
جزئیات یا Legی قابل دسترسی نیست.

**Independent Test**: با session معتبر، باز کردن `/shipments`، مشاهده ردیف‌های جدول با
داده واقعی API، اعمال فیلتر وضعیت و جستجو، و صفحه‌بندی — بدون نیاز به صفحه جزئیات.

**Acceptance Scenarios**:

1. **Given** کاربر لاگین کرده، **When** `/shipments` باز می‌شود، **Then** جدول محموله‌ها
   در `glass` card با حداقل یک ردیف (در صورت وجود seed) و `data-testid=shipment-list-view`
   نمایش داده می‌شود.
2. **Given** لیست در حال بارگذاری است، **When** API پاسخ نداده، **Then** اسکلتون/Loader2
   amber نمایش داده می‌شود.
3. **Given** API خطا برمی‌گرداند، **When** لیست لود می‌شود، **Then** باکس خطای rose با
   متن فارسی و امکان تلاش مجدد نمایش داده می‌شود.
4. **Given** هیچ محموله‌ای با فیلتر فعلی وجود ندارد، **When** نتیجه خالی است، **Then**
   حالت Empty با آیکون Package و متن «محموله‌ای یافت نشد» نمایش داده می‌شود.
5. **Given** بیش از یک صفحه داده وجود دارد، **When** کاربر صفحه بعد/قبل را می‌زند،
   **Then** ردیف‌های جدید بدون reload کامل shell بارگذاری می‌شوند.
6. **Given** کاربر عبارت جستجو وارد می‌کند، **When** جستجو اعمال می‌شود، **Then** فقط
   محموله‌های منطبق (شماره، مشتری، کشتی) نمایش داده می‌شوند.
7. **Given** کاربر فیلتر وضعیت را تغییر می‌دهد، **When** «همه وضعیت‌ها» یا وضعیت خاص
   انتخاب می‌شود، **Then** لیست مطابق وضعیت فیلتر می‌شود و badge هر ردیف با
   `StatusBadge` و پالت shipment نمایش داده می‌شود.
8. **Given** کاربر روی آیکون مشاهده کلیک می‌کند، **When** ردیف انتخاب می‌شود، **Then**
   به صفحه جزئیات همان محموله هدایت می‌شود.

---

### User Story 2 - جزئیات محموله و Timeline (Priority: P2)

اپراتور از لیست یا لینک مستقیم، جزئیات یک محموله را می‌بیند: هدر با شماره amber،
badge وضعیت، عنوان مشتری، مسیر حمل، تب‌های «نمای کلی / اسناد / تحویل کانتینر»،
کارت اطلاعات عملیاتی، و خط زمانی وضعیت‌ها با دکمه‌های transition وضعیت بعدی —
مطابق `design-reference/04-shipment-detail-desktop.png`.

**Why this priority**: جزئیات هسته تصمیم‌گیری عملیاتی است؛ Timeline وضعیت چرخه حیات
محموله را شفاف می‌کند.

**Independent Test**: باز کردن `/shipments/{id}` با شناسه معتبر و مشاهده timeline،
اطلاعات عملیاتی و transitionهای مجاز — مستقل از پنل Leg (که می‌تواند empty باشد).

**Acceptance Scenarios**:

1. **Given** شناسه محموله معتبر است، **When** صفحه جزئیات باز می‌شود، **Then**
   `data-testid=shipment-detail-view` و `data-testid=shipment-detail-no` با شماره
   mono amber نمایش داده می‌شود.
2. **Given** داده در حال بارگذاری است، **When** جزئیات fetch می‌شود، **Then** اسکلتون
   یا Loader2 نمایش داده می‌شود.
3. **Given** محموله یافت نشد (404)، **When** جزئیات درخواست می‌شود، **Then** پیام خطا
   یا Empty مناسب با لینک بازگشت به لیست نمایش داده می‌شود.
4. **Given** timeline رویداد دارد، **When** بخش «خط زمانی وضعیت‌ها» رندر می‌شود،
   **Then** هر entry با dot رنگی وضعیت، عنوان فارسی، تاریخ `fa-IR` و meta نمایش
   داده می‌شود (`data-testid=shipment-timeline`).
5. **Given** transition مجاز وجود دارد، **When** اپراتور دکمه وضعیت بعدی (مثلاً
   «رزرو فضا») را می‌زند، **Then** وضعیت به‌روز و timeline تازه‌سازی می‌شود؛ در
   خطا پیام rose نمایش داده می‌شود.
6. **Given** کاربر «بازگشت به لیست» می‌زند، **When** navigation انجام می‌شود، **Then**
   به `/shipments` برمی‌گردد (فلش ArrowRight با `rotate-180` در RTL).
7. **Given** تب «نمای کلی» فعال است، **When** جزئیات نمایش داده می‌شود، **Then**
   کارت اطلاعات عملیاتی شامل شماره، مشتری، کشتی، مبدا/مقصد، ETA، free time و
   تعداد کانتینر نمایش داده می‌شود.

---

### User Story 3 - پنل مدیریت Legهای حمل (Priority: P3)

اپراتور در صفحه جزئیات محموله، بخش «مسیر حمل (Legها)» را می‌بیند: لیست
مراحل چندوجهی (sequence، mode، carrier، مبدا/مقصد، وضعیت leg)، افزودن/ویرایش/حذف
Leg، و empty state در صورت نبود Leg — مطابق
`design-reference/05-shipment-legs-desktop.png`.

**Why this priority**: محموله‌های multimodal بدون Leg قابل رزرو نیستند؛ این پنل
عملیات چندمرحله‌ای را پوشش می‌دهد.

**Independent Test**: باز کردن جزئیات محموله‌ای بدون Leg → empty state؛ افزودن Leg
جدید → نمایش در لیست با badge وضعیت leg — بدون وابستگی به تب اسناد.

**Acceptance Scenarios**:

1. **Given** محموله Leg ندارد، **When** پنل Leg رندر می‌شود، **Then** متن empty
   «هنوز Legی ثبت نشده» با `data-testid=legs-empty` و دکمه «+ افزودن Leg»
   (`data-testid=leg-create-btn`) نمایش داده می‌شود.
2. **Given** Legها وجود دارند، **When** پنل بارگذاری می‌شود، **Then** هر Leg با
   sequence badge amber، وضعیت `StatusBadge` (پالت leg) و اقدامات ویرایش/حذف
   (`data-testid=legs-list`, `data-testid=leg-item-{sequence}`) نمایش داده می‌شود.
3. **Given** کاربر «افزودن Leg» می‌زند، **When** فرم inline باز می‌شود، **Then**
   فیلدهای mode، carrier، origin، destination، departure، arrival با `erpTokens.input`
   نمایش و با submit ذخیره می‌شوند (`data-testid=leg-form`).
4. **Given** ذخیره Leg ناموفق است، **When** API خطا برمی‌گرداند، **Then** پیام خطای
   فارسی در همان پنل نمایش داده می‌شود.
5. **Given** وضعیت Leg قابل تغییر است، **When** اپراتور وضعیت را عوض می‌کند،
   **Then** badge leg به‌روز و پالت مطابق `leg status` (SCHEDULED/IN_TRANSIT/ARRIVED/DELAYED)
   اعمال می‌شود.

---

### User Story 4 - لیست محموله موبایل (Priority: P4)

اپراتور در viewport موبایل (Pixel 5) همان فهرست محموله را با shell فشرده (`w-14`
sidebar آیکون‌محور) می‌بیند: عنوان، آمار خلاصه، جستجو، فیلتر و جدول واکنش‌گرا با
ستون‌های ضروری — مطابق `design-reference/10-shipments-mobile.png`.

**Why this priority**: اپراتورهای میدانی به دسترسی موبایل نیاز دارند؛ تکمیل
تجربه ERP پس از داشبورد موبایل.

**Independent Test**: viewport Pixel 5، `/shipments` — KPI/header، جستجو و حداقل
یک ردیف جدول بدون اسکرول افقی شکسته.

**Acceptance Scenarios**:

1. **Given** viewport ≤ mobile breakpoint، **When** `/shipments` باز است، **Then**
   nav sidebar `w-14` (آیکون‌فقط) و محتوای لیست در `main` با padding `p-4` قابل
   مشاهده است.
2. **Given** جدول در موبایل، **When** عرض کم است، **Then** ستون‌های اصلی (شماره،
   مشتری، وضعیت) حفظ و اسکرول افقی کنترل‌شده یا ستون‌های ثانویه مخفی می‌شوند.
3. **Given** داده بارگذاری شده، **When** لیست موبایل نمایش داده می‌شود، **Then**
   `data-testid=shipment-list-view` و `data-testid=shipment-create-btn` قابل مشاهده‌اند.

---

### Edge Cases

- اگر token منقضی شود هنگام مشاهده لیست/جزئیات؟ redirect به `/login`.
- اگر backend در دسترس نباشد؟ پیام خطای شبکه فارسی در هر view.
- حذف محموله دارای فاکتور/وابستگی؟ API خطا → پیام فارسی؛ ردیف حذف نشود.
- جستجو با کاراکتر خالی؟ بازگشت به لیست کامل فیلترشده.
- دو اپراتور همزمان transition وضعیت؟ نمایش خطای تعارض از API.
- `prefers-reduced-motion` فعال؟ انیمیشن framer 0.2s در shell/content خنثی شود.
- محموله قدیمی بدون Leg؟ empty state Leg نمایش داده شود ولی جزئیات/timeline کار کند.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST رندر همه صفحات این ماژول داخل `ErpShell` موجود در route
  group `(erp)` — **بدون** بازسازی shell.
- **FR-002**: System MUST مسیرهای `/shipments` (لیست) و `/shipments/[id]` (جزئیات)
  با محافظت middleware موجود.
- **FR-003**: System MUST نمایش لیست محموله با جستجو، فیلتر وضعیت، و صفحه‌بندی
  سمت سرور یا query-based مطابق API بک‌اند.
- **FR-004**: System MUST نمایش `StatusBadge` مشترک از `src/components/shared/` با
  `config` جداگانه برای shipment status و leg status (از `statusPalettes` موجود در
  `src/components/erp/types.tsx` — بدون بازتعریف رنگ).
- **FR-005**: System MUST پیاده‌سازی Loading / Error / Empty در هر سه view (لیست،
  جزئیات، پنل Leg) طبق الگوی `component-structure.mdc`.
- **FR-006**: System MUST استفاده از `erpTokens.glass`، `erpTokens.btnPrimary`،
  `erpTokens.input` و الگوی جدول ERP (header uppercase، row hover) برای لیست.
- **FR-007**: System MUST نمایش Timeline وضعیت در جزئیات با dot رنگی، متن 12px/10px
  meta، و دکمه‌های transition مجاز (emerald success style).
- **FR-008**: System MUST نمایش پنل Leg در جزئیات با CRUD پایه Leg و empty state
  استاندارد.
- **FR-009**: System MUST تطابق بصری با مراجع:
  `03-shipments-list-desktop.png`, `04-shipment-detail-desktop.png`,
  `05-shipment-legs-desktop.png`, `10-shipments-mobile.png`.
- **FR-010**: System MUST expose `data-testid`های کلیدی برای Visual QA و Playwright
  (جدول UI Contract زیر).
- **FR-011**: System MUST فعال‌سازی آیتم nav «محموله‌ها» در shell با `Link` به
  `/shipments` و حالت active amber روی مسیرهای `/shipments*`.
- **FR-012**: System MUST نمایش دکمه «+ محموله جدید» در لیست (`shipment-create-btn`);
  فرم wizard کامل ایجاد محموله **خارج از محدوده** این spec است (فقط CTA یا placeholder
  modal در صورت نیاز Visual QA).
- **FR-013**: System MUST فرمت تاریخ/عدد/ارز با `Intl` locale `fa-IR` و شماره
  محموله/فاکتور با `font-mono tabular-nums` طبق `rtl-persian.mdc`.
- **FR-014**: System MUST انیمیشن ورود محتوا با framer-motion 0.2s در shell موجود
  (بدون انیمیشن تکراری در کامپوننت‌های فرزند).

### UI Contract (data-testid)

| Element | data-testid |
|---------|-------------|
| Shipment list root | `shipment-list-view` |
| Create shipment CTA | `shipment-create-btn` |
| View row action | `shipment-view-{id}` |
| Delete row action | `shipment-delete-{id}` |
| Shipment detail root | `shipment-detail-view` |
| Shipment number | `shipment-detail-no` |
| Status timeline | `shipment-timeline` |
| Timeline entry | `timeline-entry` |
| Status transition button | `transition-btn-{to}` |
| Legs panel | `shipment-legs-panel` |
| Legs empty | `legs-empty` |
| Add leg button | `leg-create-btn` |
| Leg list | `legs-list` |
| Leg item | `leg-item-{sequence}` |
| Leg form | `leg-form` |

### Key Entities

- **Shipment (ListItem)**: شناسه، شماره محموله (`shipmentNo`)، مشتری، وضعیت، نوع
  حمل (mode)، مبدا/مقصد، کشتی، ETA، تعداد کانتینر، تعداد اسناد.
- **Shipment (Detail)**: همه فیلدهای ListItem + voyage، free time، timeline events،
  transitions مجاز، تب فعال.
- **ShipmentTimelineEvent**: وضعیت، برچسب فارسی، timestamp، meta/guard note.
- **ShipmentLeg**: sequence، mode، carrier، origin، destination، ETD/ETA leg، وضعیت
  leg (SCHEDULED | IN_TRANSIT | ARRIVED | DELAYED).
- **ShipmentListQuery**: page، limit، search، status filter.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: اپراتور می‌تواند از ورود به `/shipments` تا مشاهده اولین ردیف جدول را
  در کمتر از ۵ ثانیه (شبکه عادی) انجام دهد.
- **SC-002**: جستجو یا فیلتر وضعیت نتایج را در کمتر از ۳ ثانیه به‌روز می‌کند.
- **SC-003**: باز کردن جزئیات محموله از لیست در یک کلیک و کمتر از ۵ ثانیه محتوای
  timeline را نمایش می‌دهد.
- **SC-004**: مقایسه بصری خودکار: diff پیکسلی ≤ ۵٪ برای هر چهار مرجع (03، 04، 05
  دسکتاپ + 10 موبایل) در ناحیه `main`.
- **SC-005**: همه `data-testid`های UI Contract در DOM موجود و قابل wait توسط
  `visual-qa.mjs` هستند.
- **SC-006**: در viewport Pixel 5، لیست بدون شکست layout اساسی (overflow مخرب)
  قابل استفاده است.

## Assumptions

- ماژول `001-login-dashboard` تکمیل شده: auth، middleware، `ErpShell`، و TanStack Query
  provider موجودند.
- بک‌اند NestJS روی `:3001` endpointهای shipments/legs/timeline از قبل موجودند
  (مطابق `SPEC.md` §۵.۱ و پیاده‌سازی `Shipping-Project-V2`).
- داده تست از seed tenant «سیر راه آبی» (`maryam@logicore.com`) — همان الگوی
  `capture-screenshots.mjs`.
- تب‌های «اسناد» و «تحویل کانتینر» در جزئیات **فقط shell تب** نمایش داده می‌شوند؛
  محتوای کامل آن تب‌ها در specهای بعدی (Documents / Handover) است.
- Wizard ایجاد محموله کامل (فرم چندمرحله‌ای) خارج از MVP این spec است؛ دکمه CTA در
  لیست الزامی است.
- حذف محموله با تأیید modal؛ در صورت خطای business rule پیام API به فارسی نمایش
  داده می‌شود.
- Visual QA با `design-reference/visual-qa.mjs` پس از پیاده‌سازی اجرا می‌شود
  (مشابه 001).

## Out of Scope

- بازسازی `erp-shell.tsx` یا تغییر constitution
- ماژول‌های Finance، CRM، Documents content، Demurrage dashboard
- Kanban board، تقویم ETD/ETA، بستن پرونده (close workflow)
- پرتال مشتری/نماینده
- فرم wizard کامل «محموله جدید» (جز CTA)

## Dependencies

| وابستگی | وضعیت |
|---------|--------|
| `001-login-dashboard` (auth + shell) | ✅ Complete |
| `design-reference/03–05, 10` PNG | ✅ موجود |
| `.cursor/rules/design-system.mdc` | ارجاع |
| `.cursor/rules/component-structure.mdc` | ارجاع |
| `.cursor/rules/rtl-persian.mdc` | ارجاع |
| Constitution v1.0.0 | ارجاع بدون تغییر |
