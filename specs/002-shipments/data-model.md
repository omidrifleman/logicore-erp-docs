# Data Model: Shipments Management

**Feature**: 002-shipments | **Date**: 2026-07-06

**Source of truth**: NestJS Prisma schema + `Shipping-Project-V2/src/lib/api.ts` — mapped to spec entities.

## ShipmentListItem

| Field | Type | Notes |
|-------|------|-------|
| id | string (uuid) | Primary key |
| shipmentNo | string | نمایش mono amber — e.g. `SHP-000037` |
| status | ShipmentStatus | badge via `shipmentStatusPalette` |
| mode | TransportMode | `SEA` \| `LAND` \| `AIR` \| `RAIL` \| … |
| originPort | string? | مسیر — اولویت با port |
| destinationPort | string? | |
| originCity | string? | fallback مسیر |
| destinationCity | string? | |
| vesselName | string? | ستون کشتی |
| voyageNo | string? | optional در list |
| eta | string? (ISO) | `formatDate` fa-IR |
| actualArrival | string? (ISO) | live API برمی‌گرداند — UI در detail |
| freeTimeStart | string? (ISO) | live API — optional |
| freeTimeDays | number? | |
| statusHistory | StatusHistoryEntry[] | **live list هم embed دارد** — timeline API ترجیح داده شود |
| createdAt | string (ISO) | sort پیش‌فرض desc |
| updatedAt | string (ISO) | live API — optional در UI |
| tenantId | string | live API — ignore در UI |
| customerId | string | live API — ignore در UI |
| parentShipmentId | string? | live API — multimodal link |
| customer | `{ id, companyName }` | |
| _count | `{ containers, documents }` | آیکون Package/FileText |

### ShipmentStatus (enum)

`DRAFT` | `CONFIRMED` | `BOOKED` | `LOADED` | `IN_TRANSIT` | `ARRIVED` | `CUSTOMS_HOLD` | `RELEASED` | `OUT_FOR_DELIVERY` | `DELIVERED` | `INVOICED` | `CLOSED`

Labels: `shipmentStatusLabels` در `src/components/erp/types.tsx`

## ShipmentDetail

extends **ShipmentListItem** plus:

| Field | Type | Notes |
|-------|------|-------|
| voyageNo | string? | |
| freeTimeDays | number? | |
| actualArrival | string? (ISO) | |
| statusHistory | StatusHistoryEntry[] | در list و detail — برای timeline از `GET .../timeline` استفاده شود |
| containers | ShipmentContainer[] | تب تحویل — MVP shell only |
| documents | `{ id, type, docNumber }[]` | تب اسناد — MVP shell only |
| quotes | `{ id, status }[]` | |
| shipmentLegs | ShipmentLeg[]? | alias backend `shipmentLegs` |

## ShipmentTimelineEvent (StatusHistoryEntry)

| Field | Type | Notes |
|-------|------|-------|
| from | string? | وضعیت قبلی |
| to | string | وضعیت جدید — label فارسی از map |
| trigger | string | e.g. `created`, `customer_confirmed` |
| at | string (ISO) | timeline meta |
| by | string | userId |
| guard | string? | guard note amber |

**Leg timeline entries** (`ShipmentLegTimelineEntry`): legId, sequence, mode, origin, destination, status, carrierName, departureAt, arrivalAt, at, trigger — رندر جدا در timeline با dot sky.

## ShipmentTransitionOption

| Field | Type | Notes |
|-------|------|-------|
| to | ShipmentStatus | target |
| label | string | فارسی — دکمه emerald |
| trigger | string | POST body |
| guard | string? | optional guard name |

## ShipmentLeg

| Field | Type | Notes |
|-------|------|-------|
| id | string (uuid) | |
| sequence | number | badge amber circle |
| mode | LegMode | `SEA` \| `LAND` \| `RAIL` \| `AIR` |
| origin | string | |
| destination | string | |
| carrierId | string? | |
| carrier | `{ id, name, code }?` | |
| departureAt | string? (ISO) | |
| arrivalAt | string? (ISO) | |
| status | LegStatus | badge leg palette |
| createdAt | string | |
| updatedAt | string | |

### LegStatus (enum)

`SCHEDULED` | `IN_TRANSIT` | `ARRIVED` | `DELAYED`

## ShipmentListQuery (client)

| Field | Type | Notes |
|-------|------|-------|
| status | string? | maps to `?status=` — omit = all |
| search | string? | maps to `?search=` — debounced |
| page | number | **client-only** — slice after fetch |
| limit | number | **client-only** — default 25 |

### Validation Rules

- Delete: only `status === DRAFT'` and no documents/quotes (backend enforces)
- Transition: must be in `availableTransitions` from timeline API
- Leg: cannot mutate when shipment `CLOSED`
- Search: min 0 chars (empty = no filter)

## Relationships

```
Tenant 1──* Shipment
Customer 1──* Shipment
Shipment 1──* ShipmentLeg
Shipment 1──* ShipmentContainer
Shipment 1──* Document
Shipment statusHistory (JSON array on Shipment row)
```

## State Transitions (Shipment)

Key edges (full map in `shipment-transitions.ts`):

```
DRAFT → CONFIRMED → BOOKED → LOADED → IN_TRANSIT → ARRIVED → … → CLOSED
```

Leg sync may auto-advance shipment status when all legs ARRIVED.

## Client-Side Aggregates (List Header)

Computed from fetched array (not separate API):

| Stat | Rule |
|------|------|
| total | `items.length` |
| inTransit | `status === IN_TRANSIT` count |
| needsAttention | e.g. `CUSTOMS_HOLD` + overdue heuristic (match V2) |
