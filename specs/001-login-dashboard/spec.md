# Feature Specification: Login + Dashboard

**Feature Branch**: `001-login-dashboard`

**Created**: 2026-07-05

**Status**: Draft

**Input**: User description: "ماژول Login + Dashboard برای LogiCore ERP — احراز هویت کاربر و داشبورد مدیریتی KPI پس از ورود"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - ورود به سیستم (Priority: P1)

کاربر داخلی سازمان (اپراتور/مدیر) با ایمیل و رمز عبور وارد LogiCore می‌شود. در صورت داشتن
چند workspace، workspace موردنظر را انتخاب می‌کند و به محیط ERP هدایت می‌شود.

**Why this priority**: بدون احراز هویت هیچ ماژول ERP قابل دسترسی نیست؛ این دروازه ورود کل
سیستم است.

**Independent Test**: باز کردن `/login`، ورود با اعتبارنامه معتبر، مشاهده redirect به
داشبورد یا انتخاب workspace — بدون نیاز به ماژول‌های دیگر.

**Acceptance Scenarios**:

1. **Given** کاربر در صفحه لاگین است، **When** ایمیل و رمز معتبر وارد و «ورود» می‌زند،
   **Then** به داشبورد ERP هدایت می‌شود و اطلاعات tenant در session ذخیره می‌شود.
2. **Given** کاربر چند tenant دارد، **When** لاگین می‌کند، **Then** لیست workspace نمایش
   داده می‌شود و پس از انتخاب، وارد ERP می‌شود.
3. **Given** اعتبارنامه نادرست است، **When** ورود می‌زند، **Then** پیام خطای فارسی در
   باکس rose نمایش داده می‌شود و فرم قابل اصلاح می‌ماند.
4. **Given** درخواست در حال پردازش است، **When** کاربر منتظر می‌ماند، **Then** حالت Loading
   (Loader2 amber) نمایش داده می‌شود.

---

### User Story 2 - مشاهده داشبورد KPI (Priority: P2)

کاربر احراز هویت‌شده پس از ورود، داشبورد مدیریتی را می‌بیند: کارت‌های KPI (محموله فعال،
درآمد، وضعیت‌ها)، نمودار درآمد، و توزیع وضعیت محموله‌ها — مطابق مرجع بصری
`02-dashboard-desktop.png`.

**Why this priority**: داشبورد اولین صفحه پس از ورود است و ارزش فوری برای مدیران فراهم
می‌کند.

**Independent Test**: با session معتبر، باز کردن `/` و مشاهده KPIها و نمودار با داده واقعی
از API — مستقل از لیست محموله/مالی.

**Acceptance Scenarios**:

1. **Given** کاربر لاگین کرده، **When** صفحه اصلی ERP باز می‌شود، **Then** کارت‌های KPI با
   مقادیر عددی و برچسب فارسی نمایش داده می‌شوند (`data-testid=dashboard-view`).
2. **Given** داده داشبورد در حال بارگذاری است، **When** API پاسخ نداده، **Then** اسکلتون
   KPI نمایش داده می‌شود.
3. **Given** API خطا برمی‌گرداند، **When** داشبورد لود می‌شود، **Then** باکس خطای rose با
   متن فارسی نمایش داده می‌شود.
4. **Given** داده خالی است، **When** summary خالی برمی‌گردد، **Then** حالت Empty مناسب
   نمایش داده می‌شود (نه صفحه سفید).

---

### User Story 3 - ناوبری Shell و واکنش‌گرایی (Priority: P3)

کاربر از header و sidebar ERP بین بخش‌ها جابه‌جا می‌شود. در موبایل (Pixel 5) sidebar
فشرده (آیکون‌محور) و داشبورد قابل مشاهده است — مطابق `09-dashboard-mobile.png`.

**Why this priority**: Shell مشترک همه ماژول‌هاست؛ باید از ابتدا با Login/Dashboard یکپارچه
باشد.

**Independent Test**: پس از لاگین، کلیک روی آیتم «داشبورد» در nav و بررسی layout در viewport
موبایل — بدون پیاده‌سازی ماژول‌های دیگر.

**Acceptance Scenarios**:

1. **Given** کاربر در داشبورد است، **When** sidebar را می‌بیند، **Then** آیتم فعال با
   استایل amber مشخص است و header glass-strong نمایش داده می‌شود.
2. **Given** viewport موبایل (≤640px)، **When** داشبورد باز است، **Then** nav فشرده
   (`w-14`) و KPIها در grid واکنش‌گرا چیده می‌شوند.
3. **Given** کاربر لاگین نکرده، **When** `/` را باز می‌کند، **Then** به `/login` هدایت
   می‌شود.

---

### Edge Cases

- چه اتفاقی می‌افتد اگر token منقضی شود؟ کاربر به `/login` با پیام مناسب هدایت می‌شود.
- اگر backend در دسترس نباشد؟ پیام خطای شبکه فارسی در Login و Dashboard.
- اگر کاربر Enter در فیلد password بزند؟ همان رفتار دکمه ورود.
- اگر reduced-motion فعال باشد؟ انیمیشن‌ها طبق `prefers-reduced-motion` کوتاه/غیرفعال شوند.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST نمایش صفحه لاگین فارسی RTL در `/login` با فیلدهای email (LTR) و
  password (LTR) و دکمه primary amber.
- **FR-002**: System MUST احراز هویت از طریق API بک‌اند (`POST /api/auth/login`) و در صورت
  multi-tenant، `POST /api/auth/select-workspace`.
- **FR-003**: System MUST ذخیره token، user، tenant و role در cookie/localStorage مطابق
  الگوی موجود پروژه.
- **FR-004**: System MUST محافظت از routeهای ERP و redirect کاربر غیرمجاز به `/login`.
- **FR-005**: System MUST نمایش داشبورد KPI با فراخوانی `/api/dashboard/summary`.
- **FR-006**: System MUST پیاده‌سازی Loading/Error/Empty برای Login و Dashboard.
- **FR-007**: System MUST استفاده از `erp-shell` (header + sidebar) برای layout پس از ورود.
- **FR-008**: System MUST تطابق بصری با `design-reference/01-login-desktop.png` و
  `02-dashboard-desktop.png` (و موبایل `09-dashboard-mobile.png`).
- **FR-009**: System MUST expose `data-testid` برای عناصر کلیدی: `login-email`,
  `login-password`, `dashboard-view`, `kpi-active-shipments`.
- **FR-010**: Users MUST be able to خروج از سیستم (logout) و پاک‌سازی session.

### Key Entities

- **User**: ایمیل، نام، نقش، وضعیت احراز هویت.
- **Tenant/Workspace**: شناسه، نام حقوقی/تجاری، کشور — برای multi-tenant.
- **AuthSession**: token، tenant انتخاب‌شده، role، تاریخ انقضا.
- **DashboardSummary**: KPIهای محموله، درآمد، بازه زمانی، توزیع وضعیت، سری زمانی درآمد.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: کاربر می‌تواند ورود تا مشاهده اولین KPI را در کمتر از ۳۰ ثانیه (شبکه عادی)
  تکمیل کند.
- **SC-002**: ۹۵٪ تلاش‌های ورود با اعتبارنامه صحیح در اولین تلاش موفق می‌شوند.
- **SC-003**: داشبورد با داده واقعی در کمتر از ۵ ثانیه محتوای KPI را نمایش می‌دهد.
- **SC-004**: مقایسه بصری خودکار با اسکرین‌شات مرجع: حداکثر ۵٪ تفاوت پیکسلی (perceptual)
  در layout اصلی Login و Dashboard دسکتاپ.
- **SC-005**: Lighthouse Accessibility score ≥ ۹۰ برای صفحات Login و Dashboard.

## Assumptions

- بک‌اند NestJS روی `:3001` با endpointهای auth و dashboard از قبل یا همزمان آماده می‌شود.
- کاربر تست: `maryam@logicore.com` / tenant «سیر راه آبی» مطابق `capture-screenshots.mjs`.
- MVP فقط نقش داخلی (internal ERP) است؛ پرتال agent/customer خارج از این spec است.
- MFA و SSO در فاز بعدی هستند؛ فقط email/password + workspace select.
- داده KPI از API واقعی می‌آید؛ mock فقط در dev بدون backend.
