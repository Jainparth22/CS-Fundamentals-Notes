# Authentication, Authorization and Web Security: Interview Notes

## 1. Authentication vs authorization
- **Authentication (AuthN):** who are you? **Authorization (AuthZ):** what may you do?
- Models: **RBAC** (roles), **ABAC** (attributes/policies), ACLs, ownership checks (user can only edit their own rows).

## 2. Password storage
- Never store plain text or fast hashes (MD5, SHA-256 alone).
- Use a slow, salted, adaptive hash: **argon2id**, **bcrypt**, or scrypt. Per-user random salt (built in), optional pepper from a secret store.
- Enforce rate limiting and lockout on login; use constant-time comparison.

## 3. Sessions vs tokens
| | Server session (cookie) | JWT / token |
|---|---|---|
| State | stored on server (Redis/DB) | self-contained, stateless |
| Revocation | easy (delete session) | hard; needs a blacklist or short expiry |
| Scaling | needs shared session store | any server can verify |
| Best for | server-rendered web apps | APIs, SPAs, mobile, microservices |

## 4. JWT in depth
A JWT is `header.payload.signature` (Base64URL). Payload is **not encrypted**, only signed: never put secrets in it.
```json
{ "sub": "42", "role": "recruiter", "exp": 1767225600, "iat": 1767222000, "jti": "b3a1..." }
```
- **Signing:** HS256 (shared secret) or RS256/ES256 (private key signs, public key verifies; better for multiple services).
- **Validate:** signature, `exp`, `nbf`, `iss`, `aud`, algorithm (pin the algorithm; reject `alg: none`).
- **Access token (short, 5 to 15 min) + refresh token (long, stored securely, rotated)**. Refresh-token rotation with reuse detection limits theft damage.
- **Revocation / logout:** keep a **JTI blacklist** in Redis with TTL = remaining lifetime (what your Placement Portal does), or keep tokens short-lived and revoke refresh tokens.
- **Where to store in browsers:** `HttpOnly; Secure; SameSite` cookie resists XSS theft but needs CSRF protection; `localStorage` is readable by any injected script.

## 5. OAuth 2.0 and OpenID Connect
- **OAuth 2.0:** delegated authorization (let an app access your Google Calendar without your password). **OIDC** adds identity (ID token) on top.
- **Authorization Code flow with PKCE** for web/mobile apps: user logs in at the provider, app receives a code, exchanges it for tokens.
- Store provider refresh tokens encrypted (Fernet/KMS), as in your Healthcare Google Calendar integration.
- Roles: resource owner, client, authorization server, resource server; scopes limit access.

## 6. Common web vulnerabilities (OWASP)
| Attack | What | Defence |
|---|---|---|
| SQL injection | untrusted input becomes SQL | parameterised queries / ORM; never string-format SQL |
| XSS | injected script runs in the victim's browser | output escaping (template autoescape), CSP, sanitise HTML, HttpOnly cookies |
| CSRF | a site makes the browser send an authenticated request | CSRF tokens, `SameSite` cookies, check `Origin`; not needed for pure bearer-token APIs |
| IDOR / broken access control | change an ID to read others' data | check ownership and role on every request |
| SSRF | server fetches attacker-chosen URLs | allow-lists, block internal IPs, no redirects |
| Mass assignment | client sets fields like `role` | whitelist fields via input schemas |
| Insecure deserialization | untrusted pickle/YAML | never unpickle untrusted data |
| Brute force / credential stuffing | repeated logins | rate limits, lockout, MFA, breached-password checks |
| Sensitive data exposure | secrets in code, logs, JWT | env vars/secret manager, redact logs, TLS everywhere |
| Dependency vulnerabilities | vulnerable packages | `pip-audit`, Dependabot, pin versions |
| Clickjacking | UI framing | `X-Frame-Options` / CSP `frame-ancestors` |

## 7. Transport and headers
HTTPS/TLS everywhere, HSTS, `Secure` cookies, security headers (`Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy`). Verify webhooks with HMAC signatures and timestamp windows.

## 8. Rate limiting and abuse protection
Token bucket / sliding window per IP, user or API key; Redis-backed counters (`INCR` + `EXPIRE`, or Lua for atomicity); return `429` and `Retry-After`; protect login, signup, OTP and expensive endpoints. See the rate limiter in your HLD portfolio.

## 9. Secrets and configuration
Environment variables or a secrets manager; never commit `.env`; rotate keys; separate keys per environment; least privilege for DB users and cloud roles.

## 10. Auth flow design (login with refresh and logout)
1. `POST /auth/login` -> verify password hash -> issue access JWT (15 min) + refresh token (stored hashed server-side).
2. Client sends `Authorization: Bearer <access>`.
3. On expiry, `POST /auth/refresh` with the refresh token -> new pair; invalidate the old refresh token (rotation).
4. `POST /auth/logout` -> add access `jti` to Redis blacklist until `exp`; delete the refresh token.
5. Middleware/dependency: verify signature, `exp`, blacklist, load roles; enforce authorization per route.

## 11. Multi-tenant and RBAC design tips
Row-level ownership (`tenant_id` on every table, filtered in a base query), role-permission tables, deny by default, audit logs of sensitive actions.

## Quick interview Q&A
- **JWT vs session?** See section 3.
- **Is a JWT encrypted?** No, only signed (JWE is the encrypted variant).
- **How do you log out a stateless JWT?** Short expiry plus a JTI blacklist or refresh-token revocation.
- **What does `HttpOnly` / `Secure` / `SameSite` do?** Hide cookie from JS / HTTPS only / limit cross-site sending.
- **How do you prevent SQL injection?** Parameterised queries.
- **CSRF vs XSS?** Abusing the browser's automatic credentials vs running attacker script in your page.
- **How do you store passwords?** argon2id/bcrypt with salt.
- **What is PKCE?** Extension to the code flow that stops intercepted authorization codes being reused.
- **How do you secure service-to-service calls?** mTLS or signed tokens (JWT with audience), network policies.
- **What is least privilege?** Give each user, service and token only the permissions it needs.
