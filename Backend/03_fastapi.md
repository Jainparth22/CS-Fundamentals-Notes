# FastAPI: Interview Notes

## 1. What FastAPI is
An **ASGI** web framework built on **Starlette** (web) and **Pydantic** (validation). It uses Python type hints to validate input, serialize output and generate **OpenAPI** docs (`/docs` Swagger UI, `/redoc`) automatically. Supports async and sync endpoints.

```python
from fastapi import FastAPI, HTTPException, Depends, Query, status
from pydantic import BaseModel, Field

app = FastAPI(title="Booking API")

class BookingIn(BaseModel):
    slot_id: int
    note: str | None = Field(None, max_length=200)

class BookingOut(BaseModel):
    id: int
    slot_id: int
    model_config = {"from_attributes": True}      # Pydantic v2 (orm_mode in v1)

@app.post("/bookings", response_model=BookingOut, status_code=status.HTTP_201_CREATED)
async def create_booking(body: BookingIn, user=Depends(get_current_user)):
    ...
    return booking

@app.get("/bookings")
def list_bookings(limit: int = Query(20, le=100), cursor: str | None = None): ...
```
Run: `uvicorn app.main:app --reload` (dev), production `gunicorn -k uvicorn.workers.UvicornWorker` or `uvicorn --workers N` behind a proxy.

## 2. ASGI vs WSGI
- **WSGI:** synchronous, one request per worker at a time (Flask, Django classic).
- **ASGI:** async-capable, supports WebSockets, long-lived connections and HTTP/2; one event loop handles many concurrent requests.

## 3. Request parsing
- Path params: `/users/{user_id}` with `user_id: int`.
- Query params: function args with defaults; `Query()` for validation.
- Body: Pydantic model; `Body()`, `Form()`, `File()` / `UploadFile`.
- Headers/cookies: `Header()`, `Cookie()`.
- Invalid input returns **422** with a structured error automatically.

## 4. Pydantic (validation and serialization)
```python
class User(BaseModel):
    email: EmailStr
    age: int = Field(ge=0, le=120)
    @field_validator("email")
    @classmethod
    def lower(cls, v): return v.lower()
```
- Pydantic v2 core is written in Rust and is much faster. Use separate **input** and **output** models (never return password hashes); `response_model` filters fields.
- `pydantic-settings` for configuration from environment variables.

## 5. Dependency injection (`Depends`)
```python
def get_db():
    db = SessionLocal()
    try: yield db            # cleanup after the response
    finally: db.close()

def get_current_user(token: str = Depends(oauth2_scheme), db=Depends(get_db)): ...
@app.get("/me")
def me(user=Depends(get_current_user)): return user
```
Dependencies are composable, cached per request, and easily overridden in tests (`app.dependency_overrides`). Use them for DB sessions, auth, pagination params, rate limits.

## 6. async def vs def
- `async def` endpoints run on the event loop: use only with async libraries (`asyncpg`, `httpx`, async SQLAlchemy). **A blocking call inside `async def` blocks every request.**
- Plain `def` endpoints run in a thread pool, so blocking code (sync SQLAlchemy, `requests`) is safe there.
- CPU-heavy work: offload to a process pool or a Celery worker.

## 7. Background tasks and lifespan
```python
from fastapi import BackgroundTasks
@app.post("/signup")
def signup(u: UserIn, bg: BackgroundTasks):
    bg.add_task(send_welcome_email, u.email)      # runs after the response, in-process
    return {"ok": True}
```
`BackgroundTasks` is fine for light work; use **Celery/RQ/Arq** for durable, retryable jobs. Startup/shutdown via the `lifespan` context manager (create pools, load models once).

## 8. Auth
- `OAuth2PasswordBearer`, JWT with `python-jose`/`pyjwt`, password hashing with `passlib[bcrypt]` or `argon2`.
- API keys via `APIKeyHeader`. Role checks as dependencies.
- See `Backend/04_auth_jwt_security.md`.

## 9. Middleware, CORS, routers, exceptions
```python
app.add_middleware(CORSMiddleware, allow_origins=["https://site.com"], allow_methods=["*"], allow_headers=["*"])
router = APIRouter(prefix="/bookings", tags=["bookings"]); app.include_router(router)

@app.exception_handler(SlotTaken)
async def slot_taken(request, exc): return JSONResponse(status_code=409, content={"error": "SLOT_TAKEN"})
```
Custom middleware for request IDs, timing and logging (`@app.middleware("http")`).

## 10. WebSockets
```python
@app.websocket("/ws/{room}")
async def ws(ws: WebSocket, room: str):
    await ws.accept()
    try:
        while True:
            msg = await ws.receive_text(); await ws.send_text(msg)
    except WebSocketDisconnect: ...
```
For multiple server instances, fan out through Redis Pub/Sub (your chat project).

## 11. Database access
SQLAlchemy 2.0 (sync or async) + Alembic migrations; session per request via dependency; avoid sharing sessions across threads; pool tuning; repository/service layers for testability.

## 12. Testing
```python
from fastapi.testclient import TestClient
client = TestClient(app)
def test_health(): assert client.get("/health").status_code == 200
app.dependency_overrides[get_db] = override_db
```
For async tests use `httpx.AsyncClient` with `pytest-asyncio`.

## 13. Performance and deployment
Uvicorn workers = CPU cores; keep handlers non-blocking; use connection pools; cache with Redis; gzip middleware; Docker image; health/readiness endpoints; behind Nginx or a load balancer; observability (structured logs, Prometheus metrics, OpenTelemetry).

## 14. FastAPI vs Flask
Type-driven validation, automatic docs, native async and dependency injection vs flexible, huge ecosystem, simpler mental model. FastAPI is the better default for new JSON APIs; Flask fine for small apps and server-rendered pages.

## Quick interview Q&A
- **Why is FastAPI fast?** ASGI/Starlette, async I/O, Pydantic v2 in Rust.
- **How does FastAPI generate docs?** From type hints and Pydantic models into an OpenAPI schema.
- **`def` vs `async def`?** Thread-pooled vs event-loop; never block inside `async def`.
- **What is `Depends`?** Dependency injection for shared logic (DB, auth).
- **How do you validate input?** Pydantic models, `Field`, `Query`, custom validators; automatic 422.
- **BackgroundTasks vs Celery?** In-process, non-durable vs distributed, durable, retryable.
- **How do you handle auth?** OAuth2 password flow or bearer JWT dependency; hash passwords with bcrypt/argon2.
- **How do you scale it?** Multiple uvicorn workers/instances behind a load balancer, stateless design, Redis, DB pooling, Celery.
- **How do you share state across workers?** You do not use in-process globals; use Redis or the DB.
