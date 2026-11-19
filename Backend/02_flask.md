# Flask: Interview Notes

## 1. What Flask is
A lightweight WSGI **micro-framework** (Werkzeug for routing/WSGI, Jinja2 for templates). It provides routing, request/response, templating and sessions; everything else (ORM, auth, forms, migrations) comes from extensions.

```python
from flask import Flask, jsonify, request, abort

app = Flask(__name__)

@app.get("/health")
def health(): return {"status": "ok"}

@app.post("/api/items")
def create_item():
    data = request.get_json(silent=True) or {}
    if "name" not in data: abort(400, "name required")
    return jsonify(id=1, name=data["name"]), 201

if __name__ == "__main__":
    app.run(debug=True)        # dev server only; use gunicorn in production
```

## 2. Core concepts
- **WSGI:** synchronous Python web server interface. Production: `gunicorn -w 4 app:app` behind Nginx. Workers = processes; each handles one request at a time (use threads or gevent for more concurrency).
- **Request context and application context:** `request`, `session`, `g` (per-request scratch space) and `current_app` are proxies bound to the current context via context variables, not true globals.
- **Routing:** `@app.route("/users/<int:id>", methods=["GET"])`; converters `int`, `string`, `uuid`, `path`.
- **`url_for('endpoint', id=1)`** builds URLs.
- **Hooks:** `before_request`, `after_request`, `teardown_request`, `errorhandler`.
- **Config:** `app.config.from_object/from_envvar`, `.env` via python-dotenv; separate dev/test/prod configs; secrets only from the environment.
- **Templates (Jinja2):** inheritance (`{% extends %}`), autoescaping (XSS protection), filters, macros.
- **Sessions:** signed cookie (client-side) using `SECRET_KEY`; use server-side sessions (Redis) for sensitive data.
- **Flask `g` vs `session`:** `g` lives for one request; `session` persists across requests.

## 3. Application factory and blueprints (how real apps are structured)
```python
# app/__init__.py
def create_app(config="app.config.DevConfig"):
    app = Flask(__name__)
    app.config.from_object(config)
    db.init_app(app); migrate.init_app(app, db); jwt.init_app(app)
    from .auth import bp as auth_bp
    from .bookings import bp as bookings_bp
    app.register_blueprint(auth_bp, url_prefix="/api/auth")
    app.register_blueprint(bookings_bp, url_prefix="/api/bookings")
    return app
```
- **Blueprints** group routes, templates and static files per feature; the **factory** avoids circular imports and lets tests create fresh apps with test config.
- Your Placement Portal and VPA/HMA/HSA projects use this pattern.

## 4. Extensions you should know
| Need | Extension |
|---|---|
| ORM | Flask-SQLAlchemy |
| Migrations | Flask-Migrate (Alembic) |
| Auth | Flask-Login (sessions), Flask-JWT-Extended (tokens) |
| Forms / CSRF | Flask-WTF |
| CORS | Flask-CORS |
| Caching | Flask-Caching (Redis) |
| Rate limiting | Flask-Limiter |
| Email | Flask-Mail |
| Serialization | Marshmallow / Pydantic |
| Background jobs | Celery (with app context) |

## 5. SQLAlchemy basics
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(255), unique=True, nullable=False)
    posts = db.relationship("Post", back_populates="author", lazy="select")

u = db.session.get(User, 1)
q = db.session.execute(db.select(User).where(User.email == e)).scalars().first()
db.session.add(obj); db.session.commit()
```
- Unit of work: changes are flushed on `commit`. Use `try/except` with `rollback`.
- **N+1 problem:** lazily loading a relationship in a loop; fix with `joinedload` / `selectinload`.
- Use migrations, not `db.create_all()`, in production.

## 6. Authentication patterns in Flask
- **Session cookie (Flask-Login):** good for server-rendered apps; needs CSRF protection.
- **JWT (Flask-JWT-Extended):** stateless tokens for SPAs and APIs; add a **blacklist (JTI)** if you need logout/revocation (stored in Redis with TTL equal to token expiry). See `Backend/04_auth_jwt_security.md`.
- Role checks via decorators:
```python
def role_required(*roles):
    def deco(fn):
        @functools.wraps(fn)
        @jwt_required()
        def wrapper(*a, **kw):
            if get_jwt().get("role") not in roles: abort(403)
            return fn(*a, **kw)
        return wrapper
    return deco
```

## 7. Error handling and validation
```python
@app.errorhandler(404)
def nf(e): return jsonify(error="not found"), 404
@app.errorhandler(Exception)
def boom(e): app.logger.exception(e); return jsonify(error="internal error"), 500
```
Validate with Marshmallow/Pydantic or manually; return 400/422 with field errors.

## 8. Testing
```python
def test_health(client): assert client.get("/health").json == {"status": "ok"}
@pytest.fixture
def client(): app = create_app("app.config.TestConfig"); return app.test_client()
```
Use an in-memory SQLite or a test Postgres, fixtures to seed data, `pytest`.

## 9. Production checklist
`gunicorn` + Nginx, `debug=False`, secret from env, HTTPS, connection pooling, DB migrations, logging, health endpoint, Sentry, rate limiting, CORS configured, static files via CDN/Nginx, Docker.

## 10. Flask vs Django vs FastAPI
| | Flask | Django | FastAPI |
|---|---|---|---|
| Style | micro, flexible | batteries included (ORM, admin, auth) | async, type-hint driven |
| Server | WSGI | WSGI/ASGI | ASGI |
| Docs | manual | manual | automatic OpenAPI |
| Choose when | small/medium APIs, control | full web apps, admin | high-concurrency APIs, validation, docs |

## Quick interview Q&A
- **What is WSGI?** Standard sync interface between Python web apps and servers.
- **What is the application factory pattern and why use it?** A function that builds the app; supports multiple configs, testing, avoids circular imports.
- **What are blueprints?** Modular groups of routes and resources.
- **What is `g`?** Per-request storage for sharing data (e.g. DB connection, current user).
- **How do you handle background tasks?** Celery (or RQ) workers with Redis/RabbitMQ; do not block requests.
- **How does the Flask session work?** Signed cookie holding session data; tamper-proof but readable by the client unless you use server-side sessions.
- **How do you prevent CSRF/XSS/SQL injection in Flask?** CSRF tokens (Flask-WTF) for cookie auth; Jinja autoescape; parameterised queries via ORM.
- **How do you scale a Flask app?** Stateless workers behind a load balancer, Redis for cache/sessions, DB pooling and replicas, Celery for async work.
- **Is Flask thread-safe?** The request context is per-thread/greenlet; avoid mutable module-level state.
