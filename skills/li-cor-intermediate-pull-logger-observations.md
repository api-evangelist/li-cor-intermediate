---
name: Pull HOBO logger observations
description: Retrieve time-series observations for one or more HOBO data loggers from LI-COR Cloud over a UTC time range, handling truncation and rate limits correctly.
api: openapi/li-cor-intermediate-hobolink-openapi.json
base_url: https://api.licor.cloud
operations:
  - "GET /v1/data"
overlay_operation_ids:
  - getOrganizationData
generated: '2026-07-19'
method: generated
---

# Pull HOBO logger observations

The HOBOLINK External API is **read-only**. There is nothing in this skill that
mutates provider state.

> Note: the upstream OpenAPI declares no `operationId` values. Steps below are
> grounded in the real method + path published in the spec. The `operationId`s in
> the frontmatter are the ones `overlays/li-cor-intermediate-hobolink-overlay.yaml`
> assigns; do not expect them in the provider's own spec.

## 1. Authenticate

Send a JWT bearer token on every request:

```
Authorization: Bearer $HOBOLINK_API_TOKEN
```

A missing or expired token returns **401 or 403 with code `ATH-001` (Invalid
token)**. Re-issue the token from LI-COR Cloud; do not retry with the same token.

## 2. Build the request

`GET /v1/data` requires all three query parameters:

| Parameter | Required | Format |
|---|---|---|
| `loggers` | yes | comma-separated logger serial numbers, e.g. `10665731,10665732` |
| `start_date_time` | yes | UTC, `yyyy-MM-dd HH:mm:ss` |
| `end_date_time` | yes | UTC, `yyyy-MM-dd HH:mm:ss` |

Both timestamps are **UTC** — convert local time before calling. Omitting or
malforming any parameter returns **400** with a code in the `VAL-001`–`VAL-008`
range.

```
GET /v1/data?loggers=10665731,10665732&start_date_time=2023-11-27%2013:45:00&end_date_time=2023-11-27%2014:45:00
```

## 3. Read the response

```json
{ "message": "...", "max_results": false, "data": [ { "logger_sn": "...", "sensor_sn": "...", "timestamp": "...", "data_type": "...", "data_type_id": "...", "value": 0, "unit": "...", "sensor_measurement_type": "..." } ] }
```

Each element of `data[]` is one timestamped reading, carrying both its
`logger_sn` and `sensor_sn`.

## 4. Handle truncation — there is no pagination

This API has **no page, cursor, limit, or offset parameter**. If
`max_results` is `true` the result set was truncated. Do not look for a next
page — instead **split the time range** and re-request the narrower windows,
then concatenate. Repeat until every window returns `max_results: false`.

## 5. Respect the rate limit

The limit is **60 requests per 60 seconds**, advertised on every response:

```
RateLimit-Policy: 60;w=60
RateLimit-Limit: 60
RateLimit-Remaining: <n>
RateLimit-Reset: <seconds>
```

Because step 4 turns one large query into many small ones, this limit is easy to
hit. Read `RateLimit-Remaining` and pace requests; when it nears zero, sleep for
`RateLimit-Reset` seconds. Exceeding the limit returns **429 `SYS-002` (Too many
requests)** — back off and retry rather than looping.

## 6. Handle server-side errors

| Status | Code | Meaning | Action |
|---|---|---|---|
| 400 | `VAL-001`–`VAL-008` | Invalid parameters | Fix the request; do not retry unchanged |
| 401 / 403 | `ATH-001` | Invalid token | Re-issue the token |
| 429 | `SYS-002` | Too many requests | Back off, retry |
| 509 | `SYS-001` | System is busy | Retry with exponential backoff |

Full registry: `errors/li-cor-intermediate-error-codes.yml`.
