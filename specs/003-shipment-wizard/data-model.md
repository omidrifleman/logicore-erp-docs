# Data Model: Shipment Creation Wizard

**Feature**: 003-shipment-wizard | **Date**: 2026-07-06

**Extends**: [002-shipments data-model](../002-shipments/data-model.md) — فیلدهای Shipment/Leg همان تعاریف 002.

## CreateShipmentPayload (Step 1 → POST)

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| customerId | string (uuid) | **yes** | از `GET /api/customers` |
| mode | TransportMode | **yes** | API enum: `SEA`, `AIR`, `ROAD`, `RAIL`, `MULTIMODAL` |
| originPort | string? | conditional | حداقل یکی از origin* |
| originCity | string? | conditional | |
| destinationPort | string? | conditional | حداقل یکی از destination* |
| destinationCity | string? | conditional | |

**Response**: `ShipmentListItem` — `status` همیشه `DRAFT`؛ `shipmentNo` تولید سرور.

### TransportMode (API)

```
SEA | AIR | ROAD | RAIL | MULTIMODAL
```

**UI mapping**: اگر picker «زمینی» با کلید `LAND` نمایش داده شود → ارسال `ROAD` به API.

## UpdateShipmentPayload (Step 2 → PATCH)

| Field | Type | Required | Mode rule |
|-------|------|----------|-----------|
| vesselName | string? | SEA: **yes** | برچسب AIR: «پرواز / هواپیما» |
| voyageNo | string? | no | فقط SEA visible |
| eta | string? (ISO date) | no | همه modes |
| freeTimeDays | number? | no | 0–60؛ عمدتاً SEA |
| originPort / originCity / destinationPort / destinationCity | string? | no | اگر گام ۱ ویرایش شد |

همچنین امکان PATCH `mode` و `customerId` در بازگشت به گام ۱ پس از create.

## CreateShipmentLegPayload (Step 3 → POST, optional)

مطابق 002 — `CreateShipmentLegDto`:

| Field | Type | Required |
|-------|------|----------|
| mode | LegMode | yes (if form open) |
| origin | string | yes |
| destination | string | yes |
| carrierId | string? | recommended |
| departureAt | string? (ISO) | no |
| arrivalAt | string? (ISO) | no |
| status | LegStatus | default `SCHEDULED` |
| sequence | number | default `1` |

## CustomerPickerItem

| Field | Type | Notes |
|-------|------|-------|
| id | string | value select |
| companyName | string | label نمایشی |

API ممکن است فیلدهای CRM بیشتر برگرداند — UI subset.

## WizardState (client-only)

| Field | Type | Notes |
|-------|------|-------|
| currentStep | 1 \| 2 \| 3 \| 4 | |
| shipmentId | string? | set after step 1 POST |
| shipmentNo | string? | برای review display |
| step1 | CreateShipmentPayload snapshot | |
| step2 | UpdateShipmentPayload snapshot | |
| legCreated | boolean | step 3 |
| legSummary | ShipmentLeg? | optional |

**Persistence**: in-memory React state — refresh صفحه wizard از نو (v1؛ resume خارج از scope).

## Validation Rules

### Step 1 (client, before POST)

```text
customerId present
mode present
(originPort OR originCity) present
(destinationPort OR destinationCity) present
```

### Step 2 (client, before PATCH)

```text
IF mode === SEA → vesselName non-empty trim
IF freeTimeDays set → 0 <= n <= 60
eta: valid date if provided
```

### Step 3 (client, if leg form visible)

```text
mode, origin, destination required
carrierId recommended — warn if empty, block if spec requires (optional v1)
```

### Step 4

Read-only review — no new validation.

## State Machine (Shipment)

**Initial status**: `DRAFT` — از `shipment-transitions.ts` backend:

```
created trigger → status DRAFT
```

Wizard **does not** call transition API — user confirms از detail page (002).

## Relationships

```
Customer 1──* Shipment (via customerId)
Shipment 1──0..1 ShipmentLeg (wizard v1 max 1 leg in step 3)
```

## Client Aggregates

پس از POST گام ۱:
- invalidate `useShipments` list
- DRAFT در لیست با فیلتر «همه» یا `DRAFT` visible

## Abandon Semantics

| Action | Backend state |
|--------|---------------|
| User leaves wizard after step 1 | DRAFT shipment exists |
| User leaves before step 1 submit | nothing created |
| Delete abandoned DRAFT | manual از list (002 delete — DRAFT only) |
