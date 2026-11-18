# HTTP, HTTPS & TLS

---

## 1. HTTP Methods

| Method | Idempotent? | Safe? | Use |
|--------|------------|-------|-----|
| GET | ✅ | ✅ | Retrieve resource |
| HEAD | ✅ | ✅ | GET without body (check headers) |
| POST | ❌ | ❌ | Create resource |
| PUT | ✅ | ❌ | Replace resource entirely |
| PATCH | ❌ | ❌ | Partial update |
| DELETE | ✅ | ❌ | Delete resource |
| OPTIONS | ✅ | ✅ | CORS preflight, list allowed methods |

**Idempotent:** Calling N times has same effect as 1 time. **Safe:** No side effects.

---

## 2. HTTP Status Codes

| Range | Category | Key Codes |
|-------|----------|-----------|
| 1xx | Informational | 100 Continue, 101 Switching Protocols (WebSocket) |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| 5xx | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

---

## 3. HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Year | 1997 | 2015 | 2022 |
| Transport | TCP | TCP | QUIC (over UDP) |
| Multiplexing | ❌ (Head-of-line blocking) | ✅ (binary frames) | ✅ |
| Header compression | ❌ | ✅ HPACK | ✅ QPACK |
| Server push | ❌ | ✅ | ✅ |
| HOL blocking | At request level | At TCP level | ✅ eliminated |
| Connection setup | 3-way handshake | Same | 0-RTT or 1-RTT |
| TLS | Optional | Effectively required | Built into QUIC |

**HTTP/1.1 workarounds:** Pipelining (rarely used), domain sharding (multiple connections), concatenating files, spriting images. HTTP/2 makes all of these unnecessary.

---

## 4. HTTP/2 Internals

```
Stream: bidirectional sequence of frames
Frame: smallest unit of HTTP/2 (header frame, data frame, etc.)

Connection carries multiple streams simultaneously:
Stream 1: GET /index.html
Stream 3: GET /style.css    ← concurrent, no waiting
Stream 5: GET /app.js
```

---

## 5. TLS (Transport Layer Security) Handshake

TLS 1.2 (older):
```
Client → Server: ClientHello (supported cipher suites, TLS version, random)
Server → Client: ServerHello (chosen cipher, certificate, random)
Client:          Verify cert (chain to trusted CA)
Client → Server: Pre-master secret (encrypted with server's public key)
Both:            Derive session keys from randoms + pre-master secret
Client → Server: Finished (encrypted)
Server → Client: Finished (encrypted)
--- Encrypted communication begins ---
```

TLS 1.3 (current, faster):
```
Client → Server: ClientHello + key_share (Diffie-Hellman params)
Server → Client: ServerHello + key_share + Certificate + Finished
Client → Server: Finished
--- Encrypted communication begins (1-RTT vs 2-RTT in TLS 1.2) ---
```

**0-RTT Resumption (TLS 1.3):** Client sends data immediately using PSK from previous session. Risk: replay attacks (use only for safe/idempotent requests).

---

## 6. Certificate Chain of Trust

```
Root CA (self-signed, pre-installed in OS/browser)
    └── Intermediate CA (signed by Root CA)
            └── Server Certificate (signed by Intermediate CA, contains server's public key)
```

Browser verifies: server cert signed by intermediate, intermediate signed by trusted root → chain of trust.

**Certificate fields:** Subject (domain), Issuer, Validity period, Public key, SANs (Subject Alternative Names), Signature.

---

## 7. HTTPS vs HTTP

| | HTTP | HTTPS |
|---|------|-------|
| Port | 80 | 443 |
| Encryption | ❌ | ✅ TLS |
| Authentication | ❌ | ✅ certificate |
| Integrity | ❌ | ✅ MAC in TLS |
| Performance | Faster (no TLS) | Slightly slower (TLS setup), but HTTP/2 compensates |

**HSTS (HTTP Strict Transport Security):** Browser always uses HTTPS after first visit:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

---

## 8. CORS (Cross-Origin Resource Sharing)

```
Origin: scheme + domain + port
Same origin: https://api.example.com and https://api.example.com
Cross origin: https://frontend.app and https://api.backend.com
```

**CORS flow:**
```
1. Browser checks if request is "simple" (GET/HEAD/POST with safe content-type)
2. For non-simple requests: preflight OPTIONS request
   Browser → Server: OPTIONS /api/data with Origin and Access-Control-Request-Method
   Server → Browser: Access-Control-Allow-Origin: * (or specific origin)
3. Actual request proceeds if allowed
```

```python
# FastAPI CORS middleware
from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(CORSMiddleware, allow_origins=["https://frontend.app"],
                   allow_methods=["*"], allow_headers=["*"])
```

---

## 9. Quick Reference

| Question | Answer |
|----------|--------|
| Difference between 401 and 403? | 401: not authenticated (no credentials); 403: authenticated but not authorized |
| What is REST? | Architectural style: stateless, resources via URLs, standard methods |
| What is a cookie vs session? | Cookie: stored client-side; session: stored server-side, session ID in cookie |
| What is JWT? | JSON Web Token: self-contained, signed token; no server lookup needed |
| What is WebSocket? | Persistent bidirectional connection over TCP; starts as HTTP, upgrades (101) |
