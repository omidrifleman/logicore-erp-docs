# API Contracts: Shipments

**Feature**: 002-shipments | **Base URL**: `http://localhost:3001` (proxied as `/api`)

**Backend source**: `Shipping-Project-V2/production-ready/nestjs-backend/src/modules/shipment/`

**Auth**: All endpoints require `Authorization: Bearer {token}` (tenant-scoped via RLS).

---

## GET /api/shipments

**Query**:

| Param | Type | Notes |
|-------|------|-------|
| status | string? | Exact ShipmentStatus filter |
| search | string? | Case-insensitive: shipmentNo, customer.companyName, vesselName |

**Response**: `ShipmentListItem[]` (array — **no server pagination**)

```json
[
  {
    "id": "string",
    "shipmentNo": "string",
    "tenantId": "string",
    "customerId": "string",
    "parentShipmentId": "string|null",
    "status": "string",
    "mode": "string",
    "originPort": "string|null",
    "destinationPort": "string|null",
    "originCity": "string|null",
    "destinationCity": "string|null",
    "vesselName": "string|null",
    "voyageNo": "string|null",
    "eta": "string|null",
    "actualArrival": "string|null",
    "freeTimeStart": "string|null",
    "freeTimeDays": 0,
    "statusHistory": [],
    "createdAt": "string",
    "updatedAt": "string",
    "customer": { "id": "string", "companyName": "string" },
    "_count": { "containers": 0, "documents": 0 }
  }
]
```

> **T0 verified**: Prisma returns full row + includes. Frontend types may use a **UI subset**; extra fields are safe to ignore.

**Errors**: 401 → redirect `/login`

---

## POST /api/shipments

**Request** (MVP: out of UI scope — documented for completeness):

```json
{
  "mode": "SEA",
  "customerId": "string",
  "originPort": "string",
  "destinationPort": "string",
  "vesselName": "string",
  "containers": [{ "containerNo": "string", "type": "string" }]
}
```

**Response**: `ShipmentListItem` (status `DRAFT`)

---

## GET /api/shipments/:id

**Response**: `ShipmentDetail`

```json
{
  "id": "string",
  "shipmentNo": "string",
  "status": "string",
  "mode": "string",
  "originPort": "string|null",
  "destinationPort": "string|null",
  "vesselName": "string|null",
  "voyageNo": "string|null",
  "eta": "string|null",
  "freeTimeDays": 5,
  "actualArrival": "string|null",
  "customer": { "id": "string", "companyName": "string", "...": "..." },
  "statusHistory": [],
  "containers": [],
  "documents": [],
  "quotes": [],
  "shipmentLegs": [],
  "_count": { "containers": 0, "documents": 0 }
}
```

**Errors**: 404 `{ "message": "Shipment {id} not found" }`

---

## PATCH /api/shipments/:id

**Request** (partial):

```json
{
  "vesselName": "string",
  "voyageNo": "string",
  "originPort": "string",
  "destinationPort": "string",
  "eta": "string",
  "freeTimeDays": 0
}
```

**Response**: `ShipmentDetail`

**Errors**: 409 if status `CLOSED`

---

## DELETE /api/shipments/:id

**Response**:

```json
{ "deleted": true, "id": "string" }
```

**Errors**: 409 — only `DRAFT`; no documents/quotes

---

## GET /api/shipments/:id/timeline

**Response**:

```json
{
  "currentStatus": "string",
  "entries": [
    {
      "from": "string|null",
      "to": "string",
      "trigger": "string",
      "at": "string",
      "by": "string",
      "guard": "string|null"
    }
  ],
  "legEntries": [
    {
      "legId": "string",
      "sequence": 1,
      "mode": "string",
      "origin": "string",
      "destination": "string",
      "status": "string",
      "carrierName": "string|null",
      "departureAt": "string|null",
      "arrivalAt": "string|null",
      "at": "string",
      "trigger": "string"
    }
  ],
  "activeLeg": {
    "sequence": 1,
    "mode": "string",
    "status": "string",
    "carrierName": "string|null",
    "origin": "string",
    "destination": "string"
  },
  "availableTransitions": [
    {
      "to": "string",
      "label": "string",
      "trigger": "string",
      "guard": "string|null"
    }
  ]
}
```

---

## POST /api/shipments/:id/transition

**Request**:

```json
{
  "newStatus": "CONFIRMED",
  "trigger": "customer_confirmed",
  "guard": "string|null"
}
```

**Response**: `ShipmentDetail`

**Errors**: 400 invalid transition; 400 guard failure

---

## GET /api/shipments/:id/legs

**Response**: `ShipmentLegItem[]`

```json
[
  {
    "id": "string",
    "sequence": 1,
    "mode": "SEA|LAND|RAIL|AIR",
    "origin": "string",
    "destination": "string",
    "carrierId": "string|null",
    "carrier": { "id": "string", "name": "string", "code": "string" },
    "departureAt": "string|null",
    "arrivalAt": "string|null",
    "status": "SCHEDULED|IN_TRANSIT|ARRIVED|DELAYED",
    "createdAt": "string",
    "updatedAt": "string"
  }
]
```

---

## POST /api/shipments/:id/legs

**Request**:

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

**Errors**: 409 if shipment `CLOSED`

---

## PATCH /api/shipments/:id/legs/:legId

**Request**: partial fields from CreateShipmentLegDto

**Response**: `ShipmentLegItem`

---

## PATCH /api/shipments/:id/legs/:legId/status

**Request**:

```json
{ "status": "IN_TRANSIT" }
```

**Response**: `ShipmentLegItem` — may sync parent shipment status

---

## DELETE /api/shipments/:id/legs/:legId

**Response**:

```json
{ "deleted": true, "id": "string" }
```

---

## UI Contract (testids)

| Element | data-testid |
|---------|-------------|
| Shipment list root | `shipment-list-view` |
| Create shipment CTA | `shipment-create-btn` |
| View row | `shipment-view-{id}` |
| Delete row | `shipment-delete-{id}` |
| Detail root | `shipment-detail-view` |
| Shipment number | `shipment-detail-no` |
| Timeline container | `shipment-timeline` |
| Status entry | `timeline-entry` |
| Leg entry in timeline | `timeline-leg-entry` |
| Transition button | `transition-btn-{to}` |
| Legs panel | `shipment-legs-panel` |
| Legs empty | `legs-empty` |
| Add leg | `leg-create-btn` |
| Leg list | `legs-list` |
| Leg item | `leg-item-{sequence}` |
| Leg form | `leg-form` |
| Leg form submit | `leg-form-submit` |

---

## Frontend Pagination Note

`page` / `limit` در **ShipmentListQuery** فقط client-side اعمال می‌شوند — backend pagination ندارد (see `research.md` R0).
