# HTTP, REST and API Design: Interview Notes

## 1. HTTP essentials
- **Request:** method, URL, headers, body. **Response:** status code, headers, body.
- Stateless: each request carries what the server needs; state lives in tokens, DB or cache.

### Methods
| Method | Purpose | Safe | Idempotent |
|---|---|---|---|
| GET | read | yes | yes |
| HEAD | headers only | yes | yes |
| POST | create / action | no | no |
| PUT | replace a resource | no | yes |
| PATCH | partial update | no | not guaranteed |
| DELETE | remove | no | yes |
| OPTIONS | capabilities / CORS preflight | yes | yes |

**Safe** = no state change. **Idempotent** = repeating gives the same server state. POST is not idempotent, so use **idempotency keys** (a client-generated UUID in `Idempotency-Key`; the server stores the first result and returns it for replays).

### Status codes
| Range | Meaning | Common ones |
|---|---|---|
| 2xx | success | 200 OK, 201 Created, 202 Accepted (async job), 204 No Content |
| 3xx | redirect | 301, 302, 304 Not Modified |
| 4xx | client error | 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (not allowed), 404, 405, 409 Conflict (e.g. double booking), 410, 415, 422 Unprocessable (validation), 429 Too Many Requests |
| 5xx | server error | 500, 502 Bad Gateway, 503 Unavailable, 504 Gateway Timeout |

### Important headers
`Content-Type`, `Accept`, `Authorization`, `Cache-Control`, `ETag` / `If-None-Match` (conditional GET), `Location`, `Retry-After`, `X-Request-ID` (tracing), `Set-Cookie`, CORS headers (`Access-Control-Allow-Origin`).

### HTTP versions
HTTP/1.1 (keep-alive, head-of-line blocking), HTTP/2 (multiplexing, header compression), HTTP/3 (QUIC over UDP). HTTPS = HTTP over TLS (see `CN/02_http_and_https.md`).

## 2. REST principles
Client-server, stateless, cacheable, uniform interface, layered system. Resources are nouns; verbs are HTTP methods.
```
GET    /api/v1/bookings          list (with filters, pagination)
POST   /api/v1/bookings          create  -> 201 + Location
GET    /api/v1/bookings/42       read
PATCH  /api/v1/bookings/42       partial update
DELETE /api/v1/bookings/42       delete -> 204
POST   /api/v1/bookings/42/cancel   action as a sub-resource when not CRUD
```
- Plural nouns, lower-case, hyphens; no verbs in paths.
- **Versioning:** URL (`/v1`), header, or media type. Do not break existing clients.
- **Filtering, sorting, pagination:** `?status=paid&sort=-created_at&limit=20&cursor=...`. Prefer cursor (keyset) pagination for large or changing data; offset pagination is simple but slows and shifts.
- **Consistent error shape:**
```json
{ "error": { "code": "SLOT_TAKEN", "message": "Slot already booked", "details": {} }, "request_id": "..." }
```
- **PUT vs PATCH:** PUT replaces the whole resource; PATCH changes some fields.
- **HATEOAS** (links in responses) is part of strict REST but rarely used.

## 3. API design checklist
1. Validate input (types, ranges, size limits); return 422/400 with field errors.
2. Authenticate (who) then authorize (what they may do), on the server, for every request.
3. Idempotency for POSTs that create or charge.
4. Pagination and rate limits on lists and expensive endpoints; return `429` with `Retry-After`.
5. Timeouts, retries with exponential backoff and jitter for outgoing calls; circuit breakers.
6. Return the right status code and a stable error body; never leak stack traces.
7. Document with OpenAPI/Swagger (FastAPI generates it); keep examples.
8. Log with a request ID; expose `/health` (liveness) and `/readyz` (readiness).
9. Use ETags / `Cache-Control` for cacheable GETs.
10. Plan for backward compatibility (add fields, do not remove or rename).

## 4. REST vs GraphQL vs gRPC vs WebSocket
| | REST | GraphQL | gRPC | WebSocket |
|---|---|---|---|---|
| Format | JSON over HTTP | single endpoint, client picks fields | Protobuf over HTTP/2 | persistent bidirectional |
| Strength | simple, cacheable | no over/under-fetch | fast, typed, streaming, service-to-service | real-time push (chat, live feeds) |
| Weakness | over-fetching, many calls | complexity, caching, N+1 | browser support, debugging | stateful scaling needs a pub/sub backplane |

Other real-time options: Server-Sent Events (one-way), long polling.

## 5. CORS in one paragraph
Browsers block cross-origin requests unless the server sends `Access-Control-Allow-Origin` (and for non-simple requests answers the `OPTIONS` preflight with allowed methods and headers). CORS is enforced by the browser, not the server; it does not protect your API from non-browser clients.

## 6. Webhooks
The server calls your URL when an event happens (payments, GitHub). Verify the signature (HMAC), respond 2xx quickly, process asynchronously, make handlers idempotent because deliveries are retried.

## 7. Caching on the HTTP layer
`Cache-Control: max-age=60, public`, `ETag` revalidation, CDN for static and public API responses, `Vary` header. Do not cache per-user data publicly.

## Quick interview Q&A
- **PUT vs PATCH vs POST?** Replace, partial update, create/action.
- **401 vs 403?** Not authenticated vs authenticated but not allowed.
- **What does idempotent mean? Which methods are?** Same effect on repeats: GET, PUT, DELETE, HEAD, OPTIONS.
- **How do you make POST idempotent?** Idempotency key stored with the response.
- **What is statelessness and why does it help?** Any server can handle any request, so you scale horizontally.
- **Offset vs cursor pagination?** Cursor is stable and fast on big data.
- **What is rate limiting and where do you apply it?** Limit requests per key; at the gateway and per endpoint; token bucket or sliding window.
- **When use WebSockets instead of REST?** Server push and low-latency two-way updates.
