# Data Model: Login + Dashboard

**Feature**: 001-login-dashboard | **Date**: 2026-07-05

## User

| Field | Type | Notes |
|-------|------|-------|
| id | string (uuid) | From backend |
| email | string | Login identifier, LTR display |
| fullName | string | فارسی display name |
| avatarUrl | string? | Optional header avatar |

## Tenant (Workspace)

| Field | Type | Notes |
|-------|------|-------|
| id | string (uuid) | Selected workspace |
| legalName | string | نمایش در tenant chip |
| tradeName | string? | Optional |
| country | string | ISO or Persian label |

## AuthSession (client-side)

| Field | Storage | Notes |
|-------|---------|-------|
| token | cookie `logicore-token` + localStorage | Bearer for API |
| tenantId | cookie `logicore-tenant` | Active tenant |
| tenantInfo | localStorage `logicore-tenant-info` | JSON Tenant |
| user | localStorage `logicore-user` | JSON User |
| role | localStorage `logicore-role` | RBAC hint |

### State Transitions

```
ANONYMOUS → (login success, single tenant) → AUTHENTICATED
ANONYMOUS → (login success, multi tenant) → WORKSPACE_SELECT → AUTHENTICATED
AUTHENTICATED → (logout) → ANONYMOUS
AUTHENTICATED → (token expired / 401) → ANONYMOUS (redirect /login)
```

## DashboardSummary

| Field | Type | Notes |
|-------|------|-------|
| activeShipments | number | KPI — `kpi-active-shipments` |
| revenueTotal | number | KPI amber/emerald tone |
| revenueCurrency | string | e.g. IRR |
| periodLabel | string | «بازه: ...» فارسی |
| statusBreakdown | { status: string; count: number }[] | Bar chart segments |
| revenueSeries | { date: string; amount: number }[] | Area chart |

### Validation Rules

- Login: email format required; password min 6 chars (backend enforces truth)
- Dashboard: all KPI numbers ≥ 0; empty series → show empty chart state not crash

## Relationships

- User → many Tenants (workspace_select response)
- AuthSession → one active Tenant
- DashboardSummary → scoped to active Tenant (backend enforces)
