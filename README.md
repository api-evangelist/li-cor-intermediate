# LI-COR Intermediate

LI-COR Intermediate, Inc. is the Battery Ventures-backed holding entity for LI-COR, a Lincoln, Nebraska maker of environmental and life-science instrumentation. LI-COR Environmental ([licor.com](https://www.licor.com/)) builds greenhouse-gas analyzers, eddy-covariance flux systems, photosynthesis and soil-gas instruments, and the HOBO data logger line; LICORbio ([licorbio.com](https://www.licorbio.com/)) builds Western blot imaging systems and infrared fluorescent reagents.

Backed by: battery-ventures

## API

**HOBOLINK External API** — `https://api.licor.cloud` — [reference](https://api.licor.cloud/v1/docs)

A read-only REST API for LI-COR Cloud (formerly HOBOlink) returning time-series observations from HOBO data loggers and their sensors. Two published operations, JWT bearer auth, 60 requests per minute.

- `GET /v1/data` — observations for a list of logger serial numbers over a UTC range
- `GET /v1/newa/{device-serial-number}` — sensor-grouped observations for one device

## Artifacts

| Artifact | Method |
|---|---|
| `openapi/` | searched (extracted from the live Swagger UI) |
| `agentic-access/`, `authentication/`, `conformance/`, `conventions/`, `data-model/`, `errors/`, `lifecycle/`, `mcp/` | derived |
| `rate-limits/`, `security/` (domain security) | probed |
| `overlays/`, `llms/`, `skills/` | generated |
| `packages/`, `well-known/` | searched (negative results recorded) |

## Not published

No `/llms.txt`, no `/.well-known/` documents or `security.txt`, no status page, no dated changelog, no public SDKs, no CLI, no webhook or AsyncAPI event surface, no sandbox, no OAuth scopes, no trust center or vulnerability-disclosure program.
