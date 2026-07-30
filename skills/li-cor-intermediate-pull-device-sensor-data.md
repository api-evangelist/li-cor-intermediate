---
name: Pull sensor-grouped data for one device
description: Retrieve observations for a single HOBO device from the LI-COR Cloud NEWA endpoint, grouped by sensor with units and metadata, over a UTC time range.
api: openapi/li-cor-intermediate-hobolink-openapi.json
base_url: https://api.licor.cloud
operations:
  - "GET /v1/newa/{device-serial-number}"
overlay_operation_ids:
  - getDeviceSensorData
generated: '2026-07-19'
method: generated
---

# Pull sensor-grouped data for one device

Use this when you want one device's readings **organized by sensor** (with each
sensor's unit, measurement type, and metadata) rather than as the flat
observation list returned by `GET /v1/data`. Read-only.

> Note: the upstream OpenAPI declares no `operationId` values. Steps are grounded
> in the real method + path. The `operationId` in the frontmatter is assigned by
> `overlays/li-cor-intermediate-hobolink-overlay.yaml`.

## 1. Authenticate

```
Authorization: Bearer $HOBOLINK_API_TOKEN
```

401/403 → `ATH-001` (Invalid token).

## 2. Call the endpoint

| Parameter | In | Required | Format |
|---|---|---|---|
| `device-serial-number` | path | yes | single device serial number |
| `start_date_time` | query | yes | UTC, `yyyy-MM-dd HH:mm:ss` |
| `end_date_time` | query | yes | UTC, `yyyy-MM-dd HH:mm:ss` |

```
GET /v1/newa/10665731?start_date_time=2023-11-27%2013:45:00&end_date_time=2023-11-27%2014:45:00
```

Unlike `/v1/data`, this endpoint takes **exactly one** device — it does not accept
a comma-separated list. To cover several devices, loop and pace against the rate
limit.

## 3. Read the response

```json
{ "more_data": false, "sensors": [ { "measurement_type": "...", "sensor_sn": "...", "unit": "...", "meta_data": "...", "observations": [ { "value": 0, "timestamp": "..." } ] } ] }
```

Observations are **nested inside each sensor**, and `unit` /
`measurement_type` live on the sensor rather than on each reading — read units
from the sensor level when converting or charting.

## 4. Handle truncation

If `more_data` is `true` the result was truncated. There is no cursor: narrow
`start_date_time`/`end_date_time` and re-request, repeating until `more_data` is
`false`.

## 5. Rate limit

60 requests per 60 seconds (`RateLimit-Policy: 60;w=60`). Watch
`RateLimit-Remaining` and sleep for `RateLimit-Reset` when it approaches zero.
429 → `SYS-002`.

## 6. Errors specific to this endpoint

| Status | Code | Meaning |
|---|---|---|
| 400 | `VAL-001`, `VAL-002`, `VAL-006`, `VAL-008`, `VAL-034` | Invalid request |
| 401 / 403 | `ATH-001` | Invalid token |
| 429 | `SYS-002` | Too many requests |
| 500 | `DW-010` | Failed to retrieve observations from data warehouse |
| 500 | `IOE-005` / `IOE-006` | Unit conversion function failed to execute / load |
| 509 | `SYS-001` | System is busy |

`DW-010`, `IOE-005`, and `IOE-006` are server-side and transient-looking — retry
with backoff, and if `IOE-*` persists for a specific sensor, report it to
envsupport@licor.com rather than retrying indefinitely. Full registry:
`errors/li-cor-intermediate-error-codes.yml`.
