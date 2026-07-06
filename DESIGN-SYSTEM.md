# LogiCore ERP — Design System Reference

> **منبع:** استخراج مستقیم از کدبیس (`src/`, `globals.css`, کامپوننت‌های shadcn/ui و ERP)  
> **هدف:** بازسازی دقیق ظاهر و رفتار بصری بدون دسترسی به کد اصلی  
> **تاریخ استخراج:** ۱۴۰۵/۰۴/۱۵

---

## فهرست

1. [پالت رنگ](#1-پالت-رنگ-کامل)
2. [تایپوگرافی](#2-تایپوگرافی)
3. [Spacing و Layout](#3-spacing-و-layout)
4. [کامپوننت‌های کلیدی](#4-کامپوننت‌های-کلیدی-component-inventory)
5. [آیکون‌ها](#5-آیکون‌ها)
6. [انیمیشن‌ها و Motion](#6-انیمیشن‌ها-و-motion)
7. [الگوهای RTL و فارسی‌سازی](#7-الگوهای-rtl-و-فارسی‌سازی)
8. [حالت‌های خاص](#8-حالت‌های-خاص-emptyerrorloading-states)
9. [اسکرین‌شات مرجع](#9-اسکرین‌شات-مرجع)
10. [Design Tokens قابل کپی](#10-خلاصه-design-tokens-قابل-کپی)
11. [چک‌لیست Validation](#چک‌لیست-validation)

---

## 1) پالت رنگ کامل

### 1.1 معماری تم

- **فریم‌ورک:** Tailwind CSS v4 (`@import "tailwindcss"` در `src/app/globals.css`)
- **تنها تم فعال:** **Dark Command Center** — متغیرها در `:root` تعریف شده‌اند؛ **light mode جداگانه وجود ندارد**
- `@custom-variant dark (&:is(.dark *))` تعریف شده ولی کلاس `.dark` روی `<html>` ست نمی‌شود؛ ERP همیشه تم تیره را نشان می‌دهد
- **پالت معنایی پروژه (طبق کامنت کد):** slate پایه + amber (کانتینر/عملیات) + emerald (موفقیت) + rose (خطر) — **بدون indigo/blue** در توکن‌های semantic

### 1.2 توکن‌های CSS (Semantic)

| متغیر | مقدار oklch | نقش | استفاده |
|-------|-------------|-----|---------|
| `--background` | `oklch(0.16 0.012 250)` | پس‌زمینه صفحه | `body`, shell |
| `--foreground` | `oklch(0.96 0.005 80)` | متن اصلی | بدنه، عناوین |
| `--card` | `oklch(0.205 0.014 250)` | سطح کارت shadcn | `Card` |
| `--card-foreground` | `oklch(0.96 0.005 80)` | متن کارت | — |
| `--popover` | `oklch(0.205 0.014 250)` | dropdown/select | `SelectContent` |
| `--primary` | `oklch(0.72 0.17 70)` | **Amber — CTA اصلی** | دکمه default، ring، focus |
| `--primary-foreground` | `oklch(0.18 0.02 70)` | متن روی primary | — |
| `--secondary` | `oklch(0.26 0.015 250)` | سطح ثانویه | دکمه secondary |
| `--secondary-foreground` | `oklch(0.94 0.005 80)` | متن secondary | — |
| `--muted` | `oklch(0.24 0.012 250)` | پس‌زمینه muted | tab list، input bg |
| `--muted-foreground` | `oklch(0.7 0.01 250)` | متن کم‌رنگ | label، placeholder، meta |
| `--accent` | `oklch(0.3 0.02 250)` | hover surface | ghost button، skeleton |
| `--destructive` | `oklch(0.65 0.22 22)` | **Rose — خطا/حذف** | دکمه destructive، toast خطا |
| `--border` | `oklch(1 0 0 / 8%)` | border پیش‌فرض | کارت‌ها، جدول، input |
| `--input` | `oklch(1 0 0 / 12%)` | border/input tint | input dark bg |
| `--ring` | `oklch(0.72 0.17 70 / 50%)` | focus ring | focus-visible همه کنترل‌ها |
| `--chart-1` | `oklch(0.72 0.17 70)` | amber | نمودار |
| `--chart-2` | `oklch(0.7 0.15 162)` | emerald | نمودار |
| `--chart-3` | `oklch(0.65 0.22 22)` | rose | نمودار |
| `--chart-4` | `oklch(0.78 0.16 95)` | gold | نمودار |
| `--chart-5` | `oklch(0.6 0.14 200)` | teal | نمودار |
| `--sidebar` | `oklch(0.185 0.013 250)` | پس‌زمینه sidebar shadcn | — |
| `--sidebar-primary` | `oklch(0.72 0.17 70)` | accent sidebar | — |
| `--sidebar-border` | `oklch(1 0 0 / 8%)` | border sidebar | — |
| `--sidebar-ring` | `oklch(0.72 0.17 70 / 50%)` | ring sidebar | — |

**Radius پایه:** `--radius: 0.75rem` (12px)  
**مشتقات:** `--radius-sm: calc(var(--radius) - 4px)`, `--radius-md: calc(var(--radius) - 2px)`, `--radius-lg: var(--radius)`, `--radius-xl: calc(var(--radius) + 4px)`

### 1.3 پس‌زمینه بدنه (گرادیان‌های محیطی)

```css
body {
  background-image:
    radial-gradient(circle at 15% 10%, oklch(0.72 0.17 70 / 0.06) 0%, transparent 35%),
    radial-gradient(circle at 85% 90%, oklch(0.7 0.15 162 / 0.05) 0%, transparent 40%),
    radial-gradient(circle at 50% 50%, oklch(0.78 0.16 95 / 0.03) 0%, transparent 50%);
  background-attachment: fixed;
}
```

### 1.4 Utilities سفارشی رنگ/سطح

| کلاس | مقدار | کاربرد |
|------|-------|--------|
| `.glass` | `background: oklch(0.205 0.014 250 / 0.7)`, `backdrop-filter: blur(14px) saturate(140%)`, `border: 1px solid oklch(1 0 0 / 0.08)` | کارت ERP، جدول، header |
| `.glass-strong` | `background: oklch(0.22 0.015 250 / 0.85)`, `blur(20px) saturate(160%)`, `border: 1px solid oklch(1 0 0 / 0.1)` | header، login card، CRM drawer |
| `.text-gradient-amber` | `linear-gradient(110deg, oklch(0.85 0.16 75) 0%, oklch(0.72 0.17 70) 60%, oklch(0.65 0.14 50) 100%)` clip text | عناوین consultation |
| `.text-gradient-emerald` | `linear-gradient(110deg, oklch(0.8 0.15 162) 0%, oklch(0.65 0.16 170) 100%)` | — |
| `.grid-bg` | خطوط `oklch(1 0 0 / 0.025)` هر 40px | پس‌زمینه consultation |
| `.shadow-glow-amber` | `0 0 0 1px oklch(0.72 0.17 70 / 0.2), 0 8px 40px -8px oklch(0.72 0.17 70 / 0.35)` | CTA لاگین، دکمه‌های amber |
| `.shadow-glow-emerald` | emerald معادل | match موفق |
| `.shadow-glow-rose` | rose معادل | variance/خطا |

### 1.5 پالت عملیاتی ERP (Tailwind palette + opacity)

ERP عمدتاً از **کلاس‌های Tailwind با opacity** استفاده می‌کند (نه متغیر semantic):

| نقش semantic | رنگ Tailwind | کلاس‌های متداول | کاربرد |
|--------------|--------------|-----------------|--------|
| **Primary / عملیات** | Amber | `text-amber-300/400`, `bg-amber-500/10`–`/30`, `border-amber-500/25`–`/40` | nav فعال، CTA، شماره محموله/فاکتور |
| **Success** | Emerald | `text-emerald-300`, `bg-emerald-500/10`–`/30`, `border-emerald-500/25`–`/40` | IN_TRANSIT، تأیید، تسویه، match |
| **Warning** | Yellow | `text-yellow-300`, `bg-yellow-500/10`, `border-yellow-500/25` | ARRIVED، پرداخت جزئی |
| **Danger** | Rose | `text-rose-300/400`, `bg-rose-500/10`–`/25`, `border-rose-500/25`–`/40` | خطا، DELAYED، رد فاکتور |
| **Info** | Sky | `text-sky-300`, `bg-sky-500/10`, `border-sky-500/25` | ISSUED، حمل هوایی، KPI |
| **Neutral** | Slate | `text-slate-300`, `bg-slate-500/10`, `border-slate-500/25` | DRAFT، SCHEDULED، CLOSED |
| **Special** | Violet | `text-violet-300`, `bg-violet-400/10`, `border-violet-400/25` | INVOICED، حمل ریلی |

**مقادیر تقریبی hex (Tailwind v4 default — برای مرجع طراحی):**

| Token | ~Hex |
|-------|------|
| amber-300 | `#fcd34d` |
| amber-400 | `#fbbf24` |
| amber-500 | `#f59e0b` |
| emerald-300 | `#6ee7b7` |
| emerald-400 | `#34d399` |
| rose-300 | `#fda4af` |
| rose-400 | `#fb7185` |
| sky-300 | `#7dd3fc` |
| sky-400 | `#38bdf8` |
| yellow-300 | `#fde047` |
| yellow-400 | `#facc15` |
| slate-300 | `#cbd5e1` |
| slate-400 | `#94a3b8` |
| violet-300 | `#c4b5fd` |
| violet-400 | `#a78bfa` |

### 1.6 Light mode

**وجود ندارد.** تم تیره تنها حالت UI است. تنها override light در `@media print` برای چاپ سفید است.

### 1.7 Focus و Scrollbar

- **Focus-visible:** `outline: 2px solid oklch(0.72 0.17 70)`, `outline-offset: 2px`, `box-shadow: 0 0 0 4px oklch(0.72 0.17 70 / 0.15)`
- **Scrollbar track:** `oklch(0.18 0.012 250)` | **thumb:** `oklch(0.35 0.015 250)`, radius 8px | **hover thumb:** `oklch(0.72 0.17 70 / 0.5)`

---

## 2) تایپوگرافی

### 2.1 فونت

| فونت | منبع | متغیر CSS | وزن‌های لودشده |
|------|------|-----------|----------------|
| **Vazirmatn** | Google Fonts via `next/font/google` | `--font-vazirmatn` | همه وزن‌های موجود در subset `arabic` + `latin` |

```tsx
// src/app/layout.tsx
const vazirmatn = Vazirmatn({
  subsets: ["arabic", "latin"],
  variable: "--font-vazirmatn",
  display: "swap",
});
```

- **فونت فارسی/RTL:** Vazirmatn — تنها فونت UI
- **اعمال:** `className="${vazirmatn.variable} font-sans antialiased"` روی `<body>`
- **Mono:** `--font-mono` هم به Vazirmatn map شده (`@theme inline`) — اعداد و کدها با `font-mono` همان فونت را می‌گیرند

### 2.2 مقیاس اندازه (استفاده واقعی در ERP)

| کلاس / سایز | مقدار | کاربرد |
|-------------|-------|--------|
| `text-[9px]` | 9px | guard label در transition |
| `text-[10px]` | 10px | meta، timeline، KPI sublabel، badge sm |
| `text-[11px]` | 11px | nav label موبایل، table header uppercase، form label |
| `text-[12px]` | 12px | **بدنه ERP پیش‌فرض**، input ERP، tab، دکمه کوچک |
| `text-[13px]` | 13px | nav item، search، login input |
| `text-sm` | 0.875rem (14px) | shadcn default، CardDescription |
| `text-base` | 1rem (16px) | input (mobile) |
| `text-lg` | 1.125rem (18px) | عنوان shipment detail |
| `text-xl` | 1.25rem (20px) | عنوان صفحه لیست |
| `text-2xl` | 1.5rem (24px) | KPI value، login title |
| `text-2xl font-black` | 24px / 900 | LogiCore ERP در login |

**shadcn Button/Input:** `text-sm` (14px) در desktop

### 2.3 وزن

| وزن | کلاس | کاربرد |
|-----|------|--------|
| 400 | (default) | متن بدنه |
| 500 | `font-medium` | badge، tab، nav |
| 600 | `font-semibold` | input label، table header |
| 700 | `font-bold` | عناوین کارت، KPI |
| 900 | `font-black` | عنوان login |

### 2.4 Line-height و Letter-spacing

- shadcn `CardTitle`: `leading-none`
- login `h1`: `tracking-tight`
- badge/table: `whitespace-nowrap` برای جلوگیری از شکست خط
- اعداد مالی: `tabular-nums` + `font-mono`

---

## 3) Spacing و Layout

### 3.1 سیستم Spacing

Tailwind v4 — **مقیاس 4px** (1 unit = 0.25rem = 4px):

| کلاس | مقدار |
|------|-------|
| `p-1` | 4px |
| `p-2` | 8px |
| `p-3` | 12px |
| `p-4` | 16px |
| `p-5` | 20px |
| `p-6` | 24px |
| `p-8` | 32px |
| `gap-1`–`gap-6` | همان مقیاس |
| `px-3.5 py-1.5` | 14px / 6px — دکمه‌های ERP |

**Padding محتوای اصلی:** `p-4 md:p-6`  
**Header height:** `h-14` (56px)

### 3.2 Border Radius

| کلاس | مقدار | کامپوننت |
|------|-------|----------|
| `rounded-xs` | ~2px | dialog close |
| `rounded-sm` | calc(--radius - 4px) ≈ 8px | select item |
| `rounded-md` | calc(--radius - 2px) ≈ 10px | shadcn button, input, badge |
| `rounded-lg` | 12px (--radius) | nav item، tab ERP، input ERP |
| `rounded-xl` | 16px | کارت ERP، search، logo box |
| `rounded-2xl` | 20px | glass card، login-adjacent panels |
| `rounded-3xl` | 24px | login card |
| `rounded-full` | 9999px | status badge، avatar، dot |

### 3.3 Shadow

| کلاس | مقدار (از کد) |
|------|---------------|
| `shadow-xs` | Tailwind v4 preset — دکمه/input shadcn |
| `shadow-sm` | Card shadcn |
| `shadow-md` | Select dropdown |
| `shadow-lg` | Dialog، Toast |
| `shadow-2xl` | Login card |
| `shadow-glow-*` | سفارشی — بخش 1.4 |

### 3.4 Max-width و Container

| المان | عرض |
|--------|-----|
| Login card | `max-w-md` (28rem / 448px) |
| CRM drawer | `max-w-md` |
| Dialog | `sm:max-w-lg` (32rem) |
| Toast viewport | `md:max-w-[420px]` |
| Sidebar (shadcn) | `16rem` / mobile `18rem` / icon `3rem` |
| ERP nav | `w-14` (56px) موبایل / `md:w-56` (224px) دسکتاپ |
| Invoice list | `max-h-[520px] overflow-y-auto` |
| Main content | `flex-1 overflow-y-auto` — full width |

### 3.5 Breakpoints (استفاده‌شده در کد)

| Prefix | min-width | کاربرد |
|--------|-----------|--------|
| `sm:` | 640px | نمایش label header، toast position |
| `md:` | 768px | nav گسترده، grid 2col، padding بیشتر |
| `xl:` | 1280px | KPI grid 4 ستونه |

**Z-index لایه‌ها:**

| لایه | z-index |
|------|---------|
| Toast | `z-[100]` |
| CRM drawer | `z-[120]` |
| Header/Nav ERP | `z-[130]` |

---

## 4) کامپوننت‌های کلیدی (Component Inventory)

### 4.1 دکمه‌ها

#### shadcn `Button` (`src/components/ui/button.tsx`)

| Variant | ظاهر |
|---------|------|
| **default (primary)** | `bg-primary text-primary-foreground shadow-xs`, hover `bg-primary/90`, h-9, rounded-md, text-sm |
| **secondary** | `bg-secondary text-secondary-foreground`, hover `/80` |
| **outline** | `border bg-background`, hover `bg-accent`, dark: `bg-input/30` |
| **ghost** | hover `bg-accent` |
| **destructive** | `bg-destructive text-white`, dark `bg-destructive/60` |
| **link** | `text-primary underline-offset-4` |

| Size | ابعاد |
|------|-------|
| default | `h-9 px-4 py-2` |
| sm | `h-8 px-3` |
| lg | `h-10 px-6` |
| icon | `size-9` |

**Disabled:** `opacity-50 pointer-events-none`  
**Focus:** `ring-ring/50 ring-[3px]`

#### دکمه‌های ERP (الگوی سفارشی — پرکاربردتر از shadcn در ERP)

| نوع | کلاس‌ها |
|-----|---------|
| **Primary amber** | `bg-amber-500/20 border border-amber-500/40 text-amber-200 hover:bg-amber-500/30 shadow-glow-amber rounded-xl px-4 py-2.5 text-[13px] font-semibold` |
| **Secondary ghost** | `border border-border text-muted-foreground hover:text-foreground hover:bg-muted/40 rounded-lg` |
| **Success** | `bg-emerald-500/15 border border-emerald-500/30 text-emerald-200 hover:bg-emerald-500/25` |
| **Danger** | `hover:text-rose-300 hover:bg-rose-500/10` |
| **Icon action** | `w-7 h-7 rounded-lg`, delete: `hover:bg-rose-500/15 hover:text-rose-300` |

### 4.2 کارت‌ها

#### shadcn `Card`
`bg-card rounded-xl border py-6 shadow-sm gap-6`, padding content `px-6`

#### ERP Glass Card (الگوی اصلی)
`glass rounded-2xl border border-border p-4` یا `p-5`  
Hover (customer): `card-hover hover:border-amber-500/30`

#### KPI Card (`management-dashboard.tsx`)
`glass rounded-2xl border p-4` + tone border:
- amber: `border-amber-500/25`, icon bg `bg-amber-500/10`, value `text-amber-300`
- rose / emerald / sky: مشابه

#### Shipment list row
`glass rounded-2xl border border-border overflow-hidden` (wrapper table)

#### Invoice list item
- selected: `bg-amber-500/10 border-amber-500/30`
- default: `bg-muted/10 border-border hover:bg-muted/20`
- `p-3 rounded-xl`

#### Finance match card
- quote hover: `hover:border-amber-500/30`
- DN hover: `hover:border-emerald-500/30`
- selected quote: `border-amber-500/40 bg-amber-500/[0.06] shadow-glow-amber`
- selected DN: `border-emerald-500/40 bg-emerald-500/[0.06] shadow-glow-emerald`

### 4.3 فیلدهای فرم

#### shadcn `Input`
`h-9 rounded-md border border-input px-3 text-sm shadow-xs`, dark `bg-input/30`, focus `ring-ring/50 ring-[3px]`

#### ERP Input (الگوی غالب)
`w-full bg-muted/30 border border-border rounded-xl pr-10 pl-4 py-2.5 text-[13px]`  
Focus: `focus:border-amber-500/40 focus:bg-muted/50`  
Icon داخل input: `absolute right-3 top-1/2 -translate-y-1/2 w-4 h-4 text-muted-foreground`  
فیلدهای LTR (email/password): `dir="ltr"`

#### shadcn `Textarea`
`min-h-16 rounded-md border px-3 py-2` — مشابه input

#### shadcn `Select`
Trigger `h-9 rounded-md border shadow-xs`; dropdown `bg-popover rounded-md border shadow-md` با animate-in/zoom-in-95

#### shadcn `Checkbox`
`size-4 rounded-[4px] border`, checked: `bg-primary border-primary`, icon `size-3.5`

### 4.4 مودال‌ها و پنل‌های کناری

#### shadcn `Dialog`
- Overlay: `bg-black/50`, fade in/out
- Content: `rounded-lg border p-6 shadow-lg max-w-lg`, zoom 95%, duration 200ms
- Close: `top-4 right-4 opacity-70 hover:opacity-100`

#### ERP Modals (quote/DN/customer form)
`motion` + `glass-strong rounded-2xl border border-border` — الگوی مشابه dialog با framer scale

#### CRM Panel (`customer-crm-panel.tsx`)
- Overlay: `fixed inset-0 z-[120]`, backdrop `bg-background/70 backdrop-blur-sm`
- Panel: `w-full max-w-md h-full glass-strong border-r border-border p-5`
- Animation: slide from right `x: "100%" → 0`, spring `damping: 28, stiffness: 320`

#### shadcn `Sheet`
slide از راست، `w-3/4 sm:max-w-sm`, duration open 500ms / close 300ms

### 4.5 Status Badge

**ساختار مشترک** (`StatusBadge`, `InvoiceStatusBadge`, `FinancialStatusBadge`):
inline-flex items-center gap-1.5 rounded-full border
{bg} {color} {border}
px-2 py-0.5 text-[11px] font-medium (sm: px-1.5 text-[10px])

dot: w-1.5 h-1.5 rounded-full {dot}

text

#### Shipment Status (`src/components/erp/types.tsx`)

| Status | رنگ متن | پس‌زمینه | border | dot |
|--------|---------|----------|--------|-----|
| DRAFT | slate-300 | slate-500/10 | slate-500/25 | slate-400 |
| CONFIRMED, BOOKED, LOADED, OUT_FOR_DELIVERY | amber-300 | amber-500/10 | amber-500/25 | amber-400 |
| IN_TRANSIT, RELEASED, DELIVERED | emerald-300 | emerald-500/10 | emerald-500/25 | emerald-400 |
| ARRIVED | yellow-300 | yellow-500/10 | yellow-500/25 | yellow-400 |
| CUSTOMS_HOLD | rose-300 | rose-500/10 | rose-500/25 | rose-400 |
| INVOICED | violet-300 | violet-400/10 | violet-400/25 | violet-400 |
| CLOSED | slate-300 | slate-500/10 | slate-500/25 | slate-400 |

#### Leg Status

| Status | palette |
|--------|---------|
| SCHEDULED | slate |
| IN_TRANSIT | emerald |
| ARRIVED | yellow |
| DELAYED | rose |

#### Invoice Status (`src/components/erp/finance/types.tsx`)

| Status | palette |
|--------|---------|
| DRAFT | slate |
| PENDING_APPROVAL | amber |
| APPROVED, PAID | emerald |
| ISSUED | sky |
| PARTIALLY_PAID | yellow |
| REJECTED | rose |

### 4.6 جدول‌ها

#### ERP Shipment Table
- Wrapper: `glass rounded-2xl`
- Header: `border-b border-border bg-muted/20 text-[11px] font-semibold text-muted-foreground uppercase`
- Row: `border-b border-border/50 hover:bg-white/[0.02] transition-colors`
- **Zebra striping:** ندارد
- Shipment no: `font-mono text-[12px] font-semibold text-amber-300`

#### shadcn `Table`
- Head: `h-10 font-medium`
- Row: `hover:bg-muted/50 border-b`
- Footer: `bg-muted/50 border-t`

### 4.7 نوار ناوبری

#### Header (`erp-shell.tsx`)
- `sticky top-0 z-[130] glass-strong border-b h-14`
- Logo: `w-9 h-9 rounded-xl bg-gradient-to-br from-amber-500/25 to-amber-600/5 border border-amber-500/30`
- Tenant chip: `px-3 py-1.5 rounded-lg bg-muted/30 border border-border`
- Avatar: `w-8 h-8 rounded-full bg-gradient-to-br from-amber-400/40 to-emerald-400/20 border border-amber-500/30 text-amber-300`
- Notification dot: `bg-rose-400 w-1.5 h-1.5`

#### Sidebar Nav
- `w-14 md:w-56 border-l border-border bg-muted/10`
- **Active:** `bg-amber-500/15 text-amber-300 border border-amber-500/25`
- **Inactive:** `text-muted-foreground hover:text-foreground hover:bg-muted/30`
- Icon: `w-4 h-4`; label: `hidden md:inline text-[13px]`

#### موبایل
- nav فقط آیکون (`w-14`) — label مخفی تا `md:`
- header brand text: `hidden sm:block`

### 4.8 Toast / Notification

**کامپوننت:** `@radix-ui/react-toast` via `src/components/ui/toast.tsx` + `<Toaster />` در layout

| Variant | ظاهر |
|---------|------|
| default | `border bg-background text-foreground rounded-md p-4 shadow-lg` |
| destructive | `border-destructive bg-destructive text-destructive-foreground` |

**Position:** mobile top (`slide-in-from-top-full`), desktop bottom-right (`sm:bottom-0 sm:right-0`, `slide-in-from-bottom-full`)  
**Animation:** fade-out-80, slide-out-to-right-full  
**Close icon:** Lucide `X` `h-4 w-4`

> ERP عمدتاً از inline error boxes استفاده می‌کند؛ toast در layout mount شده ولی استفاده محدود در ERP views.

### 4.9 Timeline (Shipment Tracking)

**مکان:** `shipment-detail.tsx` — بخش «خط زمانی وضعیت‌ها»

- Container: `space-y-3`
- Entry: `flex items-start gap-2`
- Dot: `w-2 h-2 rounded-full mt-1.5` + رنگ status `cfg.dot`
- Title: `text-[12px] font-medium`
- Meta: `text-[10px] text-muted-foreground`
- Guard note: `text-[10px] text-amber-400/80`
- Leg entries: dot sky fallback `bg-sky-400` اگر status نامشخص
- Empty: `text-[12px] text-muted-foreground` — «هنوز رویدادی ثبت نشده»

**Leg panel timeline** (`shipment-legs-panel.tsx`): sequence badge `w-7 h-7 rounded-full bg-amber-500/15 border border-amber-500/30 text-amber-300`

### 4.10 نمودارها

**کتابخانه:** `recharts` + wrapper `ChartContainer` (`src/components/ui/chart.tsx`)

#### Dashboard revenue chart
```tsx
config={{ amount: { label: "درآمد", color: "hsl(142 76% 45%)" } }}
// Area: stroke/fill var(--color-amount), fillOpacity={0.2}
// Grid: CartesianGrid strokeDasharray="3 3" vertical={false}
// Axis tick fontSize: 10
```

#### Chart tokens (globals)
`--chart-1` تا `--chart-5` — amber, emerald, rose, gold, teal

#### Shipment status bar (dashboard)
Track: `h-1.5 rounded-full bg-muted/40`  
Fill: `bg-sky-400/80`

#### ChartContainer styling
- Axis text: `fill-muted-foreground`
- Grid: `stroke-border/50`
- Tooltip: `rounded-lg border px-2.5 py-1.5 text-xs shadow-xl bg-background`

**انیمیشن recharts:** پیش‌فرض کتابخانه — override صریح در ERP dashboard نیست.

---

## 5) آیکون‌ها

### 5.1 کتابخانه

**`lucide-react`** — تمام آیکون‌های ERP و shadcn

### 5.2 سایزهای استاندارد

| سایز | کلاس | کاربرد |
|------|------|--------|
| 12px | `w-3 h-3` | chevron کوچک، meta |
| 14px | `w-3.5 h-3.5` | action buttons، tabs |
| 16px | `w-4 h-4` / `size-4` | **پیش‌فرض** nav، header، shadcn |
| 18px | `w-4.5 h-4.5` | logo header |
| 20px | `w-5 h-5` | عنوان صفحه، alert |
| 24px | `w-6 h-6` | loader بزرگ |
| 32px | `w-8 h-8` | login logo، empty state |

### 5.3 رنگ پیش‌فرض

| حالت | رنگ |
|------|-----|
| Neutral | `text-muted-foreground` |
| Accent / brand | `text-amber-400` |
| Success | `text-emerald-300/400` |
| Error | `text-rose-400` |
| Loading spin | `text-amber-400` |
| Empty faded | `text-muted-foreground/30` |

### 5.4 لوگو

**SVG سفارشی وجود ندارد.** لوگو = Lucide `Ship` داخل gradient box:

```tsx
<div className="w-9 h-9 rounded-xl bg-gradient-to-br from-amber-500/25 to-amber-600/5 border border-amber-500/30">
  <Ship className="w-4.5 h-4.5 text-amber-400" />
</div>
```

Login: `w-16 h-16` box + `Ship w-8 h-8 text-amber-400` + `shadow-glow-amber`

---

## 6) انیمیشن‌ها و Motion

### 6.1 CSS / Tailwind (`tw-animate-css`)

| الگو | کلاس‌ها | Duration |
|------|---------|----------|
| Fade in | `animate-in fade-in-0` | — |
| Fade out | `animate-out fade-out-0` / `fade-out-80` | — |
| Zoom | `zoom-in-95` / `zoom-out-95` | dialog/select |
| Slide | `slide-in-from-top-2`, `slide-in-from-right`, `slide-out-to-right-full` | sheet, toast |
| Dialog | `duration-200` | — |
| Sheet open/close | `duration-500` / `duration-300` | ease-in-out |
| Skeleton | `animate-pulse` | Tailwind default |
| Spinner | `animate-spin` | Loader2 |

### 6.2 Framer Motion (`framer-motion`)

| محل | پارامترها |
|-----|-----------|
| ERP view switch | `opacity 0→1`, `y: 10→0`, exit `y: -8`, `duration: 0.2` |
| Login card | `duration: 0.5`, `ease: [0.22, 1, 0.36, 1]` |
| Login logo | spring `stiffness: 200`, `delay: 0.2` |
| Login error | `height: 0 → auto`, opacity |
| CRM drawer | spring `damping: 28, stiffness: 320`, slide x |
| KPI cards | stagger `delay`, `y: 12→0` |
| Customer cards | `whileHover={{ y: -3 }}`, stagger `i * 0.05` |
| Finance variance | shake `x: [0,-10,10,-10,10,0]`, `duration: 0.6` |
| Finance card drag | `whileHover={{ x: ±3 }}` |
| Shipment list rows | `layout`, fade in/out |
| Delete modal | scale `0.96` |

### 6.3 Hover / Card

`.card-hover`: `transition transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease`  
`:hover { transform: translateY(-2px) }`

### 6.4 Skeleton / Shimmer

- shadcn Skeleton: `bg-accent animate-pulse rounded-md`
- KPI skeleton: glass card + `Skeleton h-4/h-8/h-3`
- `.shimmer`: gradient sweep `2.5s infinite`

### 6.5 prefers-reduced-motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
  .pulse-ring, .float-slow { animation: none !important; }
}
```

---

## 7) الگوهای RTL و فارسی‌سازی

### 7.1 RTL

```html
<html lang="fa" dir="rtl">
```

- Layout کلی RTL-native است
- **بدون** `tailwindcss-rtl` plugin — جهت از `dir="rtl"` مرورگر می‌آید
- Sidebar ERP: `border-l` (در RTL = سمت راست صفحه)
- CRM panel: `border-r`, slide از `x: "100%"` (از چپ viewport در RTL = از بیرون)

### 7.2 فیلدهای LTR

- Email/password login: `dir="ltr"` + icon در `right-3` (سمت راست فیلد در RTL = ابتدای متن LTR)

### 7.3 Flip آیکون‌ها

| آیکون | تغییر |
|-------|-------|
| `ArrowRight` (بازگشت) | `rotate-180` برای فلش «بازگشت به لیست» |
| `ArrowLeft` (drop zone) | بدون flip — «یک آیتم را اینجا بکشید» |
| `ChevronDown/Up` | بدون flip |

### 7.4 تاریخ و عدد

| تابع | فرمت |
|------|------|
| `formatDate()` | `toLocaleDateString("fa-IR", { year: "numeric", month: "short", day: "numeric" })` |
| `formatRelative()` | فارسی دستی: «X دقیقه/ساعت/روز پیش» |
| `formatMoney()` | `Intl.NumberFormat("fa-IR", { style: "currency", currency })` |
| `formatNumber()` | `Intl.NumberFormat("fa-IR")` |
| `formatCurrency()` | `en-US` number + symbol ارز |

**dayjs-jalali:** import مستقیم در `src/` **وجود ندارد** — تاریخ از `Intl` با locale `fa-IR` (تقویم شمسی مرورگر).

---

## 8) حالت‌های خاص (Empty / Error / Loading)

### 8.1 Loading

| الگو | ظاهر |
|------|------|
| **Spinner اصلی** | `Loader2 w-5/w-6/w-8 text-amber-400 animate-spin` centered |
| **Skeleton KPI** | 4 کارت glass + `Skeleton` blocks |
| **shadcn Skeleton** | `bg-accent animate-pulse rounded-md` |
| **CRM/Panel** | `Loader2 text-muted-foreground animate-spin` |

### 8.2 Empty State

| محل | آیکون | متن | CTA |
|-----|-------|-----|-----|
| Shipment list | `Package w-8 h-8 text-muted-foreground/30` | «محموله‌ای یافت نشد» | — |
| Invoice list | — | «فاکتوری صادر نشده» `text-[12px] text-muted-foreground` | — |
| Invoice detail | — | «یک فاکتور را برای مشاهده جزئیات انتخاب کنید» در glass card | — |
| Finance drop zone | `ArrowLeft w-5 h-5 text-muted-foreground/30` | «{label} — یک آیتم را اینجا بکشید» | — |
| Legs panel | — | `text-[12px] text-muted-foreground` data-testid=legs-empty | دکمه افزودن |
| Timeline | — | «هنوز رویدادی ثبت نشده» | — |

### 8.3 Error

| الگو | کلاس‌ها |
|------|---------|
| **Inline box (استاندارد ERP)** | `rounded-xl bg-rose-500/10 border border-rose-500/25 p-2.5 flex items-center gap-2 text-[12px] text-rose-300` + `AlertCircle w-3.5/w-4/w-5 text-rose-400` |
| Login error | همان + framer height animation |
| Transition error | `text-[11px] text-rose-300` |
| shadcn Alert destructive | `text-destructive bg-card` |

---

## 9) اسکرین‌شات مرجع

پوشه: [`design-reference/`](design-reference/)

اسکریپت گرفتن اسکرین‌شات: `design-reference/capture-screenshots.mjs`  
راهنما: [`design-reference/README.md`](design-reference/README.md)

| # | فایل | صفحه |
|---|------|------|
| 1 | `01-login-desktop.png` | `/login` |
| 2 | `02-dashboard-desktop.png` | داشبورد KPI |
| 3 | `03-shipments-list-desktop.png` | لیست محموله |
| 4 | `04-shipment-detail-desktop.png` | جزئیات + timeline |
| 5 | `05-shipment-legs-desktop.png` | پنل Leg |
| 6 | `06-finance-desktop.png` | تطبیق مالی |
| 7 | `07-invoices-desktop.png` | فاکتورها |
| 8 | `08-crm-desktop.png` | مشتریان |
| 9 | `09-dashboard-mobile.png` | داشبورد موبایل |
| 10 | `10-shipments-mobile.png` | محموله موبایل |

---

## 10) خلاصه Design Tokens قابل کپی

### CSS Custom Properties (`globals.css`)

```css
:root {
  --radius: 0.75rem;

  --background: oklch(0.16 0.012 250);
  --foreground: oklch(0.96 0.005 80);
  --card: oklch(0.205 0.014 250);
  --card-foreground: oklch(0.96 0.005 80);
  --popover: oklch(0.205 0.014 250);
  --popover-foreground: oklch(0.96 0.005 80);
  --primary: oklch(0.72 0.17 70);
  --primary-foreground: oklch(0.18 0.02 70);
  --secondary: oklch(0.26 0.015 250);
  --secondary-foreground: oklch(0.94 0.005 80);
  --muted: oklch(0.24 0.012 250);
  --muted-foreground: oklch(0.7 0.01 250);
  --accent: oklch(0.3 0.02 250);
  --accent-foreground: oklch(0.96 0.005 80);
  --destructive: oklch(0.65 0.22 22);
  --border: oklch(1 0 0 / 8%);
  --input: oklch(1 0 0 / 12%);
  --ring: oklch(0.72 0.17 70 / 50%);
  --chart-1: oklch(0.72 0.17 70);
  --chart-2: oklch(0.7 0.15 162);
  --chart-3: oklch(0.65 0.22 22);
  --chart-4: oklch(0.78 0.16 95);
  --chart-5: oklch(0.6 0.14 200);
  --sidebar: oklch(0.185 0.013 250);
  --sidebar-foreground: oklch(0.94 0.005 80);
  --sidebar-primary: oklch(0.72 0.17 70);
  --sidebar-primary-foreground: oklch(0.18 0.02 70);
  --sidebar-accent: oklch(0.26 0.015 250);
  --sidebar-accent-foreground: oklch(0.96 0.005 80);
  --sidebar-border: oklch(1 0 0 / 8%);
  --sidebar-ring: oklch(0.72 0.17 70 / 50%);
}

/* Tailwind v4 @theme inline */
@theme inline {
  --font-sans: var(--font-vazirmatn);
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}
```

### ERP Component Tokens (Tailwind classes)

```js
// erp-tokens.js — الگوهای پرتکرار برای paste در پروژه جدید
export const erpTokens = {
  glass: "glass rounded-2xl border border-border",
  glassStrong: "glass-strong rounded-2xl border border-border",
  pageTitle: "text-xl font-bold",
  pageTitleIcon: "w-5 h-5 text-amber-400",
  bodyText: "text-[12px]",
  metaText: "text-[11px] text-muted-foreground",
  monoAccent: "font-mono text-amber-300",
  input: "w-full bg-muted/30 border border-border rounded-xl px-3 py-2.5 text-[13px] outline-none focus:border-amber-500/40 focus:bg-muted/50 transition-colors",
  btnPrimary: "bg-amber-500/20 border border-amber-500/40 text-amber-200 hover:bg-amber-500/30 shadow-glow-amber rounded-xl text-[13px] font-semibold transition-colors disabled:opacity-60",
  btnSuccess: "bg-emerald-500/15 border border-emerald-500/30 text-emerald-200 hover:bg-emerald-500/25 rounded-lg text-[12px] transition-colors",
  btnDanger: "hover:text-rose-300 hover:bg-rose-500/10 rounded-lg transition-colors",
  navActive: "bg-amber-500/15 text-amber-300 border border-amber-500/25",
  navInactive: "text-muted-foreground hover:text-foreground hover:bg-muted/30 border border-transparent",
  errorBox: "rounded-xl bg-rose-500/10 border border-rose-500/25 p-2.5 flex items-center gap-2 text-[12px] text-rose-300",
  statusBadge: (bg, color, border, dot) =>
    `inline-flex items-center gap-1.5 rounded-full border ${bg} ${color} ${border} px-2 py-0.5 text-[11px] font-medium`,
};
```

### Status Color Map (برای بازسازی badge)

```js
export const statusPalettes = {
  slate:  { color: "text-slate-300",  bg: "bg-slate-500/10",  border: "border-slate-500/25",  dot: "bg-slate-400" },
  amber:  { color: "text-amber-300",  bg: "bg-amber-500/10",  border: "border-amber-500/25",  dot: "bg-amber-400" },
  emerald:{ color: "text-emerald-300",bg: "bg-emerald-500/10",border: "border-emerald-500/25",dot: "bg-emerald-400" },
  yellow: { color: "text-yellow-300", bg: "bg-yellow-500/10", border: "border-yellow-500/25", dot: "bg-yellow-400" },
  rose:   { color: "text-rose-300",   bg: "bg-rose-500/10",   border: "border-rose-500/25",   dot: "bg-rose-400" },
  sky:    { color: "text-sky-300",    bg: "bg-sky-500/10",    border: "border-sky-500/25",    dot: "bg-sky-400" },
  violet: { color: "text-violet-300", bg: "bg-violet-400/10", border: "border-violet-400/25", dot: "bg-violet-400" },
};
```

---

## چک‌لیست Validation

برای تأیید کامل بودن سند و بازسازی بصری، این ۱۰ مورد را بررسی کنید:

1. **پس‌زمینه صفحه** تیره slate با سه radial-gradient محیطی (amber/emerald/gold) و `background-attachment: fixed` نمایش داده می‌شود
2. **فونت Vazirmatn** از Google Fonts لود شده و تمام UI فارسی RTL با `dir="rtl"` و `lang="fa"` رندر می‌شود
3. **Header** شیشه‌ای (`glass-strong`) با ارتفاع 56px، لوگوی Ship در gradient amber، و nav sidebar با حالت فعال amber نمایش صحیح دارد
4. **کارت‌های glass** با `rounded-2xl`، border `border-border`، و hover `card-hover` (translateY -2px) مطابق ERP عمل می‌کنند
5. **Status badge** هر وضعیت (SCHEDULED, IN_TRANSIT, ARRIVED, DELAYED, PENDING_APPROVAL و غیره) رنگ dot/bg/border درست دارد
6. **دکمه primary ERP** شفاف amber (`bg-amber-500/20`) با `shadow-glow-amber` است — نه دکمه solid پررنگ
7. **Input ERP** با `bg-muted/30 rounded-xl` و focus `border-amber-500/40` — متفاوت از shadcn `rounded-md h-9`
8. **Timeline** با dot 8px رنگی، متن 12px/10px، و transition buttons emerald کار می‌کند
9. **انیمیشن‌ها** framer (view switch 0.2s، CRM spring) و `prefers-reduced-motion` رعایت شده‌اند
10. **اسکرین‌شات‌های** `design-reference/` با UI زنده مقایسه شده و ۱۰ صفحه کلیدی (لاگین، داشبورد، محموله، جزئیات، مالی، فاکتور، CRM — دسکتاپ و موبایل) تطابق بصری دارند

---

*پایان سند — منبع: `src/app/globals.css`, `src/app/layout.tsx`, `src/components/ui/*`, `src/components/erp/*`*
