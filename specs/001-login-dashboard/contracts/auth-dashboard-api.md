# API Contracts: Auth + Dashboard

**Feature**: 001-login-dashboard | **Base URL**: `http://localhost:3001` (proxied as `/api`)

## POST /api/auth/login

**Request**:
```json
{ "email": "string", "password": "string" }
```

**Response A — single workspace**:
```json
{
  "type": "direct",
  "token": "string",
  "user": { "id": "string", "email": "string", "fullName": "string" },
  "tenant": { "id": "string", "legalName": "string", "tradeName": "string|null", "country": "string" },
  "role": "string"
}
```

**Response B — workspace select**:
```json
{
  "type": "workspace_select",
  "tempToken": "string",
  "user": { "id": "string", "email": "string", "fullName": "string" },
  "tenants": [{ "id": "string", "legalName": "string", "tradeName": "string|null", "country": "string" }]
}
```

**Errors**: 401 `{ "message": "string" }` — نمایش در error box فارسی

---

## POST /api/auth/select-workspace

**Headers**: `Authorization: Bearer {tempToken}`

**Request**:
```json
{ "tenantId": "string" }
```

**Response**:
```json
{
  "token": "string",
  "tenant": { "id": "string", "legalName": "string", "tradeName": "string|null", "country": "string" },
  "role": "string"
}
```

---

## GET /api/dashboard/summary

**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "activeShipments": 0,
  "revenueTotal": 0,
  "revenueCurrency": "IRR",
  "periodLabel": "string",
  "statusBreakdown": [{ "status": "string", "count": 0 }],
  "revenueSeries": [{ "date": "string", "amount": 0 }]
}
```

**Errors**: 401 → redirect login; 5xx → dashboard error state

---

## UI Contract (testids)

| Element | data-testid |
|---------|-------------|
| Login email input | `login-email` |
| Login password input | `login-password` |
| Dashboard root | `dashboard-view` |
| KPI active shipments | `kpi-active-shipments` |
