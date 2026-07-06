# Implementation Plan: Login + Dashboard

**Branch**: `001-login-dashboard` | **Date**: 2026-07-05 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-login-dashboard/spec.md`

## Summary

پیاده‌سازی صفحه لاگین فارسی RTL و داشبورد KPI مدیریتی LogiCore ERP با Next.js App Router،
اتصال به API احراز هویت و dashboard summary، layout مشترک `erp-shell`، و تطابق بصری با
`design-reference/01-login-desktop.png`, `02-dashboard-desktop.png`, `09-dashboard-mobile.png`.

## Technical Context

**Language/Version**: TypeScript 5.x, Node.js 22 LTS, React 19.x, Next.js 16.x (App Router)

**Primary Dependencies**: Tailwind CSS v4, shadcn/ui, framer-motion, lucide-react, Vazirmatn
(`next/font/google`), TanStack Query 5 (server state), recharts (dashboard chart)

**Storage**: Cookie + localStorage for auth session (`logicore-token`, `logicore-tenant`, etc.)

**Testing**: Playwright visual QA (`design-reference/visual-qa.mjs`), manual RTL check

**Target Platform**: Web PWA — desktop 1280×800 and mobile Pixel 5

**Project Type**: Web application (frontend Next.js + backend NestJS on :3001)

**Performance Goals**: Dashboard KPI visible <5s; API summary P95 <2s per SPEC.md

**Constraints**: DESIGN-SYSTEM.md only; no light mode; framer-motion view transitions 0.2s;
`prefers-reduced-motion` respected

**Scale/Scope**: 2 pages (login, dashboard) + shared shell; MVP internal users only

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Tech Stack | ✅ PASS | Next.js + Tailwind v4 + shadcn + framer-motion + Vazirmatn |
| II. RTL Persian | ✅ PASS | `lang="fa" dir="rtl"`, LTR email/password fields |
| III. Design System | ✅ PASS | glass, erpTokens, amber primary — reference screenshots |
| IV. Component Reuse | ✅ PASS | shadcn via MCP → `src/components/ui/`; ERP in `src/components/erp/` |
| V. Async States | ✅ PASS | Loading/Error/Empty on login + dashboard |
| VI. Simplicity | ✅ PASS | MVP scope: login + dashboard only, no extra modules |

**Post-design re-check**: ✅ All gates pass. No violations requiring Complexity Tracking.

## Project Structure

### Documentation (this feature)

```text
specs/001-login-dashboard/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── auth-dashboard-api.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── app/
│   ├── layout.tsx              # Vazirmatn, RTL, globals
│   ├── globals.css             # Design tokens, glass utilities
│   ├── login/
│   │   └── page.tsx            # Login page
│   └── page.tsx                # Dashboard (ERP home)
├── components/
│   ├── ui/                     # shadcn primitives (via MCP)
│   ├── shared/                 # ErrorBox, loaders
│   └── erp/
│       ├── erp-shell.tsx       # Header + sidebar layout
│       ├── management-dashboard.tsx
│       └── types.tsx           # statusPalettes
└── lib/
    ├── erp-tokens.ts
    ├── format.ts               # fa-IR date/money/number
    └── auth.ts                 # session helpers

design-reference/
├── capture-screenshots.mjs
└── visual-qa.mjs               # Compare vs reference PNGs

.cursor/
├── mcp.json                    # shadcn + playwright MCP
└── rules/                      # design-system, rtl, component-structure, ponytail
```

**Structure Decision**: Single Next.js frontend at repo root `src/` (greenfield scaffold).
Backend assumed at `production-ready/nestjs-backend` or `:3001` proxy via `next.config`.

## Complexity Tracking

> No constitution violations. Table intentionally empty.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
