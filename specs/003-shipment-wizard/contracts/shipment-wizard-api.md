# API Contracts: Shipment Wizard

**Feature**: 003-shipment-wizard | **Base URL**: `http://localhost:3001` (proxied `/api`)

**Extends**: [002-shipments contracts](../002-shipments/contracts/shipments-api.md) — همان auth و error handling.

---

## GET /api/customers

**Auth**: Bearer token (tenant-scoped)

**Response**: `CustomerListItem[]`

```json
[
  {
    "id": "string",
    "companyName": "string",
    "contactName": "string|null",
    "email": "string|null",
    "phone": "string|null",
    "status": "string"
  }
]
```

> Wizard UI uses `id` + `companyName` only. Extra fields safe to ignore.

**Errors**: 401 → redirect `/login`

**Source**: `customer.controller.ts` `@Get() findAll`

---

## POST /api/shipments

**Request** — Step 1 wizard:

```json
{
  "mode": "SEA",
  "customerId": "string",
  "originPort": "string",
  "destinationPort": "string",
  "originCity": "string",
  "destinationCity": "string"
}
```

| Field | Required |
|-------|----------|
| mode | yes |
| customerId | yes |
| origin/destination fields | optional in DTO — **wizard validates** at least one origin + one destination |

**Response**: `ShipmentListItem` with `status: "DRAFT"`

**Errors**:
- 400 validation
- 401 unauthorized

---

## PATCH /api/shipments/:id

**Request** — Step 2 wizard (partial):

```json
{
  "vesselName": "string",
  "voyageNo": "string",
  "eta": "2026-08-01T00:00:00.000Z",
  "freeTimeDays": 5
}
```

**Response**: `ShipmentDetail`

**Errors**: 404 not found; 409 if CLOSED

---

## POST /api/shipments/:id/legs

**Request** — Step 3 wizard (optional):

```json
{
  "mode": "LAND",
  "origin": "string",
  "destination": "string",
  "carrierId": "string",
  "departureAt": "string",
  "arrivalAt": "string",
  "status": "SCHEDULED",
  "sequence": 1
}
```

**Response**: `ShipmentLegItem`

**Errors**: 409 if shipment CLOSED

---

## GET /api/carriers

Used in step 3 — see 002 contracts / `fetchCarriers`.

---

## UI Contract (data-testid)

| Element | data-testid |
|---------|-------------|
| Wizard root | `shipment-wizard` |
| Step container | `wizard-step-{n}` (n = 1..4) |
| Progress indicator | `wizard-progress` |
| Next | `wizard-next-btn` |
| Back | `wizard-back-btn` |
| Submit (step 4) | `wizard-submit-btn` |
| Cancel / back to list | `wizard-cancel-btn` |
| Customer select | `wizard-customer-select` |
| Mode select | `wizard-mode-select` |
| Origin | `wizard-origin` |
| Destination | `wizard-destination` |
| Vessel | `wizard-vessel` |
| Voyage | `wizard-voyage` |
| ETA | `wizard-eta` |
| Free time | `wizard-free-time` |
| Leg form | `wizard-leg-form` |
| Review summary | `wizard-review-summary` |

**Modified from 002**:

| Element | Change |
|---------|--------|
| `shipment-create-btn` | enabled → `Link` to `/shipments/new` |

---

## Frontend Helpers (to implement)

| Function | File |
|----------|------|
| `fetchCustomers()` | `src/lib/customers.ts` |
| `createShipment(data)` | `src/lib/shipments.ts` |
| `updateShipment(id, data)` | existing |
| `createShipmentLeg(shipmentId, data)` | existing |

---

## Query Invalidation

| Mutation | Invalidate |
|----------|------------|
| createShipment | `['shipments']` |
| updateShipment | `['shipments']`, `['shipment-detail', id]` |
| createShipmentLeg | `['shipment-legs', id]`, `['shipment-timeline', id]` |
