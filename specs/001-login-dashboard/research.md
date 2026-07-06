# Research: Login + Dashboard

**Feature**: 001-login-dashboard | **Date**: 2026-07-05

## R1: Authentication Flow (multi-tenant)

**Decision**: Email/password login → optional workspace select → store session in cookie +
localStorage keys matching `capture-screenshots.mjs`.

**Rationale**: Existing backend contract and screenshot script already use this pattern;
minimizes integration risk.

**Alternatives considered**:
- NextAuth.js — rejected for MVP (adds dependency; backend already owns auth)
- JWT-only memory — rejected (breaks SSR and existing cookie names)

## R2: Dashboard Data Fetching

**Decision**: TanStack Query `useQuery` for `/api/dashboard/summary` with skeleton loading,
rose error box, and `data-testid` markers for visual QA.

**Rationale**: SPEC.md mandates TanStack Query; aligns with ERP async state rules.

**Alternatives considered**:
- Server Components only — rejected for MVP (auth cookie handling simpler client-side first)
- SWR — rejected (not in constitution stack)

## R3: shadcn Component Acquisition

**Decision**: Use shadcn MCP (`npx shadcn@latest mcp`) to add Button, Input, Card, Skeleton,
Toast into `src/components/ui/`, then apply DESIGN-SYSTEM token overrides in `globals.css`.

**Rationale**: Constitution IV requires MCP/CLI, not hand-written primitives.

**Alternatives considered**:
- Copy-paste from shadcn docs — rejected per constitution
- Radix direct — rejected (loses shadcn consistency)

## R4: Visual QA Strategy

**Decision**: Extend `capture-screenshots.mjs` logic in `visual-qa.mjs` using pixelmatch +
Playwright programmatic API (not MCP-only) for deterministic CI-style comparison.

**Rationale**: MCP good for interactive debug; scripted comparison needed for regression gate.

**Alternatives considered**:
- Percy/Chromatic — rejected (external service, not requested)
- Manual-only — rejected (constitution visual QA gate)

## R5: Layout & Navigation

**Decision**: Single `erp-shell.tsx` wraps dashboard; login page standalone (no shell).
Middleware redirects unauthenticated users from `/` to `/login`.

**Rationale**: Matches design-reference: login full-page card; ERP views use glass header/nav.

**Alternatives considered**:
- Shared layout for login — rejected (design shows isolated login card)
