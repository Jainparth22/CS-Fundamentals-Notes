# Flask (10) and FastAPI (10): Most-Asked Questions

## Flask

**Q1. What is Flask and when choose it over Django?**
A lightweight WSGI micro-framework (Werkzeug + Jinja2). Choose it for small to medium APIs or when you want to pick your own components; Django suits full applications needing an ORM, admin and auth out of the box.

**Q2. What is WSGI and how does it relate to Flask?**
WSGI is the standard sync interface between Python web apps and servers. A Flask app object is a WSGI application, so Gunicorn or uWSGI can run it behind Nginx.

**Q3. Application context vs request context?**
The application context (`current_app`, `g`) is tied to the app and available in CLI and background code once pushed; the request context (`request`, `session`) exists for one HTTP request. "Working outside of application context" means you need `with app.app_context():`.

**Q4. How do `request` and `current_app` work as globals without breaking concurrency?**
They are proxies resolved through context-local storage (context variables), so each thread/greenlet sees its own request.

**Q5. What is the application factory pattern?**
A `create_app(config)` function that builds and configures the app (extensions, blueprints). It supports multiple configs, tests with fresh apps, and avoids circular imports.

**Q6. What are blueprints?**
Modular groups of routes, templates and static files registered on the app with a URL prefix, used to organise large applications by feature.

**Q7. How does the Flask session and `SECRET_KEY` work?**
The default session is a signed cookie; `SECRET_KEY` signs it so tampering is detected (content is readable by the client). Use server-side sessions (Redis) for sensitive data.

**Q8. How do you protect a Flask app against CSRF and XSS?**
CSRF tokens (Flask-WTF) and SameSite cookies for cookie-auth; Jinja2 autoescaping for XSS (turned off by `|safe`, so sanitise); parameterised queries via the ORM for SQL injection.

**Q9. How do you run background tasks and deploy Flask?**
Offload slow work to Celery/RQ workers. Deploy with Gunicorn workers behind Nginx, `debug=False`, config from environment variables, logging, health endpoint, Docker.

**Q10. How do you avoid the N+1 query problem in SQLAlchemy?**
Use eager loading (`joinedload`, `selectinload`) or a join so related rows load in one or two queries instead of one per parent row; confirm with query logging.

## FastAPI

**Q11. What is FastAPI and what is it built on?**
An ASGI framework built on Starlette (web) and Pydantic (validation) that uses type hints to validate input, serialise output and generate OpenAPI docs (`/docs`).

**Q12. ASGI vs WSGI?**
WSGI is synchronous, one request per worker at a time; ASGI supports async, WebSockets and long-lived connections on an event loop. Run FastAPI with Uvicorn (or Gunicorn + Uvicorn workers).

**Q13. `async def` vs `def` endpoints, and what happens if you block in `async def`?**
`async def` runs on the event loop; `def` runs in a thread pool. A blocking call (sync DB driver, `requests`, `time.sleep`) inside `async def` freezes every request. Use async libraries, `def`, or offload to a thread/process/worker.

**Q14. How does dependency injection (`Depends`) work?**
Functions declared as dependencies are called per request, can have sub-dependencies, are cached per request, can use `yield` for cleanup (DB sessions), and can be replaced in tests with `app.dependency_overrides`.

**Q15. How does Pydantic validation work and what does `response_model` do?**
Pydantic models validate and coerce request data (invalid input returns 422 automatically). `response_model` filters and validates the output, so secrets (like password hashes) are never returned.

**Q16. Middleware vs dependency?**
Middleware runs for every request (CORS, request IDs, timing, logging); dependencies run only for endpoints that declare them (auth, DB session, pagination).

**Q17. How do you handle errors globally?**
Raise `HTTPException` for expected errors; register `@app.exception_handler` for custom exceptions and validation errors so every error returns a consistent JSON shape.

**Q18. `BackgroundTasks` vs Celery?**
`BackgroundTasks` runs after the response in the same process (fine for light work, not durable or retryable). Celery/Arq are distributed, durable and retryable for important jobs.

**Q19. How do you manage DB sessions, config and lifespan?**
A dependency that yields a session and closes it; settings via `pydantic-settings` from environment variables; create pools and load models once in the `lifespan` handler.

**Q20. Your API is slow under load. How do you debug it?**
Measure latency percentiles and traces, find blocking calls in async code, check DB queries and pool exhaustion, add caching and indexes, run enough workers, move CPU work to workers, and load-test before and after.
