# Feature Specification: Shipment Creation Wizard

**Feature Branch**: `003-shipment-wizard`

**Created**: 2026-07-06

**Status**: Draft — plan complete 2026-07-06

**Input**: User description: «Wizard چندمرحله‌ای ایجاد محموله جدید — از انتخاب مشتری و نوع حمل تا ثبت مسیر، جزئیات کشتی/پرواز و افزودن Legهای اولیه. جایگزین دکمه غیرفعال 'ایجاد محموله' در shipments-list که در فیچر 002 عمداً از scope خارج مونده بود.»

**Governance**: این spec تابع [Constitution v1.0.0](.specify/memory/constitution.md) است — بدون تغییر در constitution. الزامات Design System از `.cursor/rules/design-system.mdc` و `.cursor/rules/component-structure.mdc` ارجاع داده می‌شوند.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - مشتری، نوع حمل و مسیر پایه (Priority: P1)

اپراتور از لیست محموله‌ها دکمه «+ محموله جدید» را می‌زند و وارد wizard می‌شود. در **گام ۱** مشتری را از لیست tenant انتخاب می‌کند، نوع حمل (mode) را مشخص می‌کند، و مسیر پایه (مبدا/مقصد — port یا city) را وارد می‌کند. با تأیید گام، سیستم محموله **DRAFT** می‌سازد و شناسه آن برای گام‌های بعدی نگه داشته می‌شود.

**Why this priority**: بدون مشتری، mode و مسیر، هیچ پرونده حمل معتبری وجود ندارد؛ این گام حداقل MVP wizard است و دکمه غیرفعال 002 را فعال می‌کند.

**Independent Test**: باز کردن wizard → پر کردن گام ۱ با فیلدهای اجباری → «بعدی» → تأیید ایجاد محموله DRAFT در API و نمایش گام ۲ بدون خروج از shell.

**Acceptance Scenarios**:

1. **Given** اپراتور در `/shipments` است، **When** «+ محموله جدید» (`shipment-create-btn`) را می‌زند، **Then** به صفحه wizard (`data-testid=shipment-wizard`) هدایت می‌شود و گام ۱ (`wizard-step-1`) با progress indicator فعال نمایش داده می‌شود.
2. **Given** گام ۱ باز است، **When** `customerId` یا `mode` خالی است و کاربر «بعدی» می‌زند، **Then** پیام اعتبارسنجی فارسی نمایش داده می‌شود و محموله ساخته نمی‌شود.
3. **Given** مشتری و mode معتبر و حداقل یکی از مبدا/مقصد (port یا city) پر شده، **When** «بعدی» (`wizard-next-btn`) زده می‌شود، **Then** محموله با وضعیت **DRAFT** ایجاد می‌شود، `shipmentNo` تخصیص می‌یابد، و گام ۲ (`wizard-step-2`) نمایش داده می‌شود.
4. **Given** API ایجاد محموله خطا می‌دهد، **When** submit گام ۱ انجام می‌شود، **Then** باکس خطای rose با متن فارسی و امکان تلاش مجدد نمایش داده می‌شود.
5. **Given** لیست مشتریان در حال بارگذاری است، **When** گام ۱ رندر می‌شود، **Then** Loader2 amber یا اسکلتون select نمایش داده می‌شود.

---

### User Story 2 - جزئیات عملیاتی (Priority: P2)

پس از ایجاد DRAFT، اپراتور در **گام ۲** جزئیات عملیاتی متناسب با mode را تکمیل می‌کند: برای حمل دریایی کشتی و voyage؛ برای هوایی شماره پرواز/هواپیما (در فیلد vesselName طبق قرارداد API)؛ ETA و free time days. فیلدهای غیرمرتبط با mode مخفی یا غیرفعال می‌شوند.

**Why this priority**: جزئیات عملیاتی برای رزرو و پیگیری ضروری است؛ ولی وابسته به وجود پرونده DRAFT از US1.

**Independent Test**: ادامه wizard از محموله DRAFT ساخته‌شده → پر کردن فیلدهای mode-specific → «بعدی» → PATCH موفق و ورود به گام ۳.

**Acceptance Scenarios**:

1. **Given** mode برابر `SEA` است، **When** گام ۲ نمایش داده می‌شود، **Then** فیلدهای `vesselName` و `voyageNo` قابل ویرایش و `eta` / `freeTimeDays` اختیاری نمایش داده می‌شوند.
2. **Given** mode برابر `AIR` است، **When** گام ۲ باز است، **Then** برچسب vessel مناسب پرواز نمایش داده می‌شود و فیلد voyage اختیاری/مخفی طبق قواعد mode.
3. **Given** mode برابر `ROAD` یا `RAIL` است، **When** گام ۲ باز است، **Then** فیلدهای کشتی/voyage نمایش داده نمی‌شوند (یا غیرفعال) و ETA اختیاری باقی می‌ماند.
4. **Given** اپراتور فیلدها را پر کرده، **When** «بعدی» می‌زند، **Then** جزئیات با PATCH روی همان محموله DRAFT ذخیره و گام ۳ نمایش داده می‌شود.
5. **Given** اپراتور «قبلی» (`wizard-back-btn`) می‌زند، **When** از گام ۲ به گام ۱ برمی‌گردد، **Then** مقادیر گام ۱ (مشتری، mode، مسیر) حفظ شده و قابل ویرایش هستند (با PATCH در صورت تغییر).

---

### User Story 3 - Leg اولیه اختیاری (Priority: P3)

در **گام ۳** اپراتور می‌تواند یک Leg اولیه اضافه کند (mode، carrier، origin، destination، تاریخ‌ها) — مشابه الگوی `leg-form` فیچر 002 — یا با «رد کردن» بدون Leg به گام بازبینی برود.

**Why this priority**: محموله‌های multimodal از Leg بهره می‌برند؛ ولی wizard بدون Leg هم باید کامل شود.

**Independent Test**: از گام ۳ → افزودن یک Leg → تأیید در API → یا skip → ورود به گام ۴.

**Acceptance Scenarios**:

1. **Given** گام ۳ باز است، **When** اپراتور «افزودن Leg» را انتخاب می‌کند، **Then** فرم inline با `erpTokens.input` (mode، carrier، origin، destination، departure، arrival) نمایش داده می‌شود.
2. **Given** فرم Leg معتبر است، **When** Leg ذخیره می‌شود، **Then** Leg با `POST .../legs` ایجاد و در خلاصه گام ۳ نمایش داده می‌شود (`sequence` 1).
3. **Given** اپراتور Leg نمی‌خواهد، **When** «بعدی» بدون افزودن Leg می‌زند، **Then** بدون خطا به گام ۴ (`wizard-step-4`) می‌رود.
4. **Given** ذخیره Leg ناموفق است، **When** API خطا برمی‌گرداند، **Then** پیام rose فارسی در همان گام نمایش داده می‌شود.

---

### User Story 4 - بازبینی نهایی و ثبت (Priority: P4)

در **گام ۴** اپراتور خلاصه readonly همه ورودی‌ها (مشتری، mode، مسیر، جزئیات عملیاتی، Legهای اضافه‌شده) را می‌بیند. با «ثبت نهایی» (`wizard-submit-btn`) wizard بسته می‌شود و به صفحه جزئیات محموله تازه‌ساخته‌شده (`/shipments/{id}`) هدایت می‌شود.

**Why this priority**: بازبینی خطای انسانی را کم می‌کند؛ ولی تا US1–US3 تکمیل نشوند معنایی ندارد.

**Independent Test**: تکمیل گام‌های ۱–۳ (با یا بدون Leg) → گام ۴ → submit → redirect به `shipment-detail-view` همان id.

**Acceptance Scenarios**:

1. **Given** اپراتور به گام ۴ رسیده، **When** صفحه بازبینی رندر می‌شود، **Then** `wizard-step-4` و خلاصه فیلدهای کلیدی با برچسب فارسی نمایش داده می‌شود.
2. **Given** بازبینی نمایش داده شده، **When** «ثبت نهایی» زده می‌شود، **Then** به `/shipments/{id}` redirect می‌شود و `shipment-detail-view` با وضعیت DRAFT و شماره amber نمایش داده می‌شود.
3. **Given** اپراتور در گام ۴ «قبلی» می‌زند، **When** به گام ۳ برمی‌گردد، **Then** Legهای قبلاً ثبت‌شده در خلاصه/gام ۳ قابل مشاهده‌اند.
4. **Given** اپراتور wizard را نیمه‌کاره رها می‌کند (بازگشت به لیست)، **When** به `/shipments` برمی‌گردد، **Then** محموله DRAFT ایجادشده در لیست قابل مشاهده است (پرونده حفظ شده).

---

### Edge Cases

- Token منقضی حین wizard؟ redirect به `/login`؛ DRAFT ایجادشده در backend باقی می‌ماند.
- Backend در دسترس نیست؟ پیام خطای شبکه فارسی در هر گام؛ امکان retry.
- مشتری tenant حذف/غیرفعال شده پس از انتخاب؟ خطای API در گام ۱ یا ۴ با پیام فارسی.
- تغییر mode در گام ۱ پس از ایجاد DRAFT؟ PATCH با اعتبارسنجی مجدد؛ فیلدهای mode-specific گام ۲ reset شوند.
- `freeTimeDays` خارج از بازه ۰–۶۰؟ پیام اعتبارسنجی قبل از PATCH.
- دو تب همزمان روی یک wizard؟ هر تب session مستقل؛ آخرین PATCH برنده (رفتار استاندارد ERP).
- `prefers-reduced-motion`؟ انیمیشن transition گام‌ها 0.2s یا خنثی.
- کاربر مستقیماً `/shipments/new` بدون auth؟ middleware → login.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST جایگزین کردن دکمه غیرفعال `shipment-create-btn` در `shipment-list.tsx` با لینک/دکمه فعال به مسیر wizard (پیشنهاد: `/shipments/new`).
- **FR-002**: System MUST رندر wizard داخل `ErpShell` موجود — **بدون** modal تمام‌صفحه و **بدون** بازسازی shell.
- **FR-003**: System MUST پیاده‌سازی wizard چهارگام با progress indicator افقی بالای فرم (شماره/عنوان گام، حالت active/completed/upcoming مطابق design-system).
- **FR-004**: System MUST expose `data-testid=shipment-wizard` روی ریشه wizard و `wizard-step-{n}` (n=1..4) روی محتوای هر گام.
- **FR-005**: System MUST دکمه‌های `wizard-next-btn`، `wizard-back-btn` (گام ۲–۴)، و `wizard-submit-btn` (فقط گام ۴) با استایل `erpTokens.btnPrimary` / secondary ghost.
- **FR-006**: System MUST گام ۱: انتخاب `customerId` (اجباری)، `mode` (اجباری)، `originPort`/`originCity` و `destinationPort`/`destinationCity` (حداقل مبدا یا مقصد پر) — سپس `POST` ایجاد محموله با وضعیت **DRAFT** طبق state machine موجود.
- **FR-007**: System MUST گام ۲: `PATCH` فیلدهای عملیاتی `vesselName`, `voyageNo`, `eta`, `freeTimeDays` با اعتبارسنجی شرطی بر اساس mode (مثلاً `vesselName` توصیه‌شده/الزامی برای `SEA`).
- **FR-008**: System MUST گام ۳: افزودن اختیاری Leg اولیه از طریق همان قرارداد API و الگوی فرم Leg فیچر 002 (`POST /api/shipments/:id/legs`).
- **FR-009**: System MUST گام ۴: نمایش خلاصه readonly و redirect به `/shipments/{id}` پس از submit موفق.
- **FR-010**: System MUST استفاده از `erpTokens.input` برای همه فیلدهای متنی/select wizard.
- **FR-011**: System MUST پیاده‌سازی Loading / Error / Empty در هر گام (Loader2 amber، error box rose، empty مشتری/ carrier).
- **FR-012**: System MUST انیمیشن ورود/تعویض گام با framer-motion 0.2s (الگوی فیچر 002).
- **FR-013**: System MUST فرمت تاریخ با `Intl` locale `fa-IR` و فیلدهای LTR (تاریخ ISO در input) با `dir="ltr"` در صورت نیاز.
- **FR-014**: System MUST invalidate کش لیست محموله (`useShipments`) پس از ایجاد DRAFT تا ردیف جدید در `/shipments` دیده شود.
- **FR-015**: System MUST نگه‌داری `shipmentId` ایجادشده در state wizard پس از گام ۱ تا پایان flow (امکان resume با query `?id=` خارج از scope v1 — اختیاری).

### UI Contract (data-testid)

| Element | data-testid |
|---------|-------------|
| Wizard root | `shipment-wizard` |
| Step container | `wizard-step-{n}` (n = 1, 2, 3, 4) |
| Next button | `wizard-next-btn` |
| Back button | `wizard-back-btn` |
| Final submit | `wizard-submit-btn` |
| Customer select | `wizard-customer-select` |
| Mode select | `wizard-mode-select` |
| Origin field | `wizard-origin` |
| Destination field | `wizard-destination` |
| Vessel field | `wizard-vessel` |
| Voyage field | `wizard-voyage` |
| ETA field | `wizard-eta` |
| Free time field | `wizard-free-time` |
| Leg form (step 3) | `wizard-leg-form` |
| Review summary | `wizard-review-summary` |

> testidهای `shipment-create-btn` و detail از فیچر 002 بدون تغییر باقی می‌مانند.

### Key Entities

- **Shipment (Create)**: `customerId`, `mode`, `originPort`, `originCity`, `destinationPort`, `destinationCity`, `vesselName`, `voyageNo`, `eta`, `freeTimeDays` — فیلدها مطابق [data-model فیچر 002](../002-shipments/data-model.md). وضعیت اولیه همیشه **DRAFT**.
- **ShipmentLeg (Create — optional step 3)**: `mode`, `origin`, `destination`, `carrierId`, `departureAt`, `arrivalAt`, `status` (پیش‌فرض SCHEDULED), `sequence` — مطابق Leg فیچر 002.
- **Customer (picker)**: `id`, `companyName` — از API مشتریان tenant.
- **WizardStepState**: شماره گام فعلی (1–4)، `shipmentId` پس از گام ۱، snapshot فیلدها برای بازبینی.

### Validation Rules (خلاصه — جزئیات در data-model.md)

| Field | Rule |
|-------|------|
| customerId | اجباری |
| mode | اجباری — enum: `SEA`, `AIR`, `ROAD`, `RAIL`, `MULTIMODAL` (API backend) |
| origin / destination | حداقل یک جفت مبدا یا مقصد (port یا city) |
| vesselName | شرطی — الزامی/پیشنهادی برای `SEA`؛ برچسب متفاوت برای `AIR` |
| voyageNo | اختیاری؛ مرتبط با `SEA` |
| eta | اختیاری؛ ISO date |
| freeTimeDays | اختیاری؛ ۰–۶۰ |
| Leg (step 3) | اختیاری؛ در صورت پر شدن origin/destination/carrier/mode اجباری |

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: اپراتور می‌تواند از کلیک «محموله جدید» تا مشاهده جزئیات DRAFT تازه‌ساخته در کمتر از ۳ دقیقه (شبکه عادی) wizard را کامل کند.
- **SC-002**: گام ۱ با فیلدهای ناقص در کمتر از ۱ ثانیه پیام اعتبارسنجی فارسی نشان می‌دهد (بدون round-trip API).
- **SC-003**: پس از تکمیل wizard، محموله در لیست `/shipments` با وضعیت DRAFT و مشتری صحیح ظاهر می‌شود.
- **SC-004**: حداقل ۹۰٪ سناریوهای acceptance US1–US4 با Playwright یا تست دستی quickstart قابل تأیید هستند.
- **SC-005**: همه `data-testid`های UI Contract wizard در DOM گام مربوطه موجودند.
- **SC-006**: wizard در viewport دسکتاپ (≥768px) و موبایل (Pixel 5) بدون overflow مخرب صفحه و با shell `w-14` قابل استفاده است.

## Assumptions

- فیچر `002-shipments` تکمیل و deploy شده؛ hooks (`useShipments`, `useShipmentLegs`, `useCarriers`) و types موجودند.
- `POST /api/shipments` و `PATCH /api/shipments/:id` و `POST .../legs` روی بک‌اند NestJS فعال و تست‌شده‌اند (قرارداد 002).
- API مشتریان tenant از endpoint موجود بک‌اند (`GET /api/customers` یا معادل) قابل fetch است؛ در صورت نبود hook، در plan اضافه می‌شود.
- افزودن کانتینر در wizard **خارج از scope** v1 است (فیلد `containers` در DTO اختیاری — فاز بعدی).
- وضعیت پس از wizard **DRAFT** می‌ماند؛ transition به CONFIRMED از صفحه جزئیات (فیچر 002) انجام می‌شود.
- Progress indicator و layout wizard مرجع بصری جداگانه ندارد؛ از الگوی glass card + erpTokens فیچر 002 پیروی می‌کند.
- Visual QA اختصاصی wizard در فاز plan/tasks تعریف می‌شود (capture جدید یا checklist دستی).

## Out of Scope

- ویرایش محموله موجود از طریق wizard (فقط ایجاد جدید)
- wizard resume از URL برای پرونده نیمه‌کاره (v2)
- افزودن چند Leg در یک wizard (حداکثر یک Leg در گام ۳ v1)
- ثبت کانتینر، اسناد، یا quote در wizard
- تغییر constitution یا بازسازی `erp-shell.tsx`
- Kanban / بستن پرونده / پرتال مشتری

## Dependencies

| وابستگی | وضعیت |
|---------|--------|
| `002-shipments` (لیست، detail، legs API، types) | ✅ Complete |
| `001-login-dashboard` (auth, shell, middleware) | ✅ Complete |
| `.cursor/rules/design-system.mdc` | ارجاع |
| `.cursor/rules/component-structure.mdc` | ارجاع |
| `.cursor/rules/rtl-persian.mdc` | ارجاع |
| Constitution v1.0.0 | ارجاع بدون تغییر |
| Backend `CreateShipmentDto` | موجود در nestjs-backend |
