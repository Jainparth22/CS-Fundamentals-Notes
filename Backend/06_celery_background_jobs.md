# Celery and Background Jobs: Interview Notes

## 1. Why background jobs
Move slow, unreliable or scheduled work out of the request path: emails, PDF reports, image/video processing, ML inference, webhooks, data imports, reminders, cleanups. The API enqueues a job and returns quickly (often `202 Accepted` with a job id).

## 2. Celery architecture
```
Web app (producer) -> Broker (Redis / RabbitMQ) -> Worker(s) -> Result backend (Redis / DB, optional)
                                     ^
                              Celery Beat (scheduler)
```
- **Broker:** queue transport. RabbitMQ gives stronger delivery guarantees; Redis is simpler and faster.
- **Worker:** processes that run tasks (prefork pool by default; also gevent/eventlet/threads, solo).
- **Result backend:** stores return values/state; skip it if you do not need results.
- **Beat:** triggers periodic tasks (cron-like).

```python
# celery_app.py
from celery import Celery
from celery.schedules import crontab
celery = Celery("app", broker="redis://localhost:6379/0", backend="redis://localhost:6379/1")
celery.conf.update(task_acks_late=True, worker_prefetch_multiplier=1, task_time_limit=300,
                   timezone="Asia/Kolkata",
                   beat_schedule={"hold-expiry": {"task": "tasks.expire_holds", "schedule": 60.0},
                                  "monthly-report": {"task": "tasks.monthly_report", "schedule": crontab(day_of_month=1, hour=2)}})

@celery.task(bind=True, autoretry_for=(ConnectionError,), retry_backoff=True, retry_jitter=True, max_retries=5)
def send_email(self, to, subject, body):
    ...

# caller
send_email.delay("a@b.com", "Hi", "Body")
send_email.apply_async(args=[...], countdown=30, queue="emails")
```
Run: `celery -A celery_app.celery worker -l info` and `celery -A celery_app.celery beat -l info`.

## 3. Reliability concepts
| Concept | Meaning |
|---|---|
| Acknowledgement | worker confirms completion; `acks_late=True` acks after the task finishes so a crash re-delivers it (at-least-once) |
| At-least-once vs at-most-once | default Celery with late ack = at-least-once (tasks may run twice); early ack = at-most-once (may be lost) |
| Idempotency | a task must be safe to run twice (check state, use idempotency keys, upsert) |
| Retries | exponential backoff with jitter; cap retries; send to a dead-letter store after the cap |
| Visibility timeout (Redis) | if a task runs longer than it, Redis re-delivers it to another worker (set it above your longest task) |
| Prefetch | `worker_prefetch_multiplier=1` for long tasks so workers do not hoard messages |
| Time limits | `soft_time_limit` raises an exception in the task; `time_limit` kills it |
| Routing | separate queues per workload (`emails`, `reports`, `ml`) and dedicated workers |
| Priorities | RabbitMQ supports; with Redis use separate queues |
| Rate limits | `rate_limit="10/m"` per task |

## 4. Patterns
- **Transactional outbox:** write the event/email row in the same DB transaction as the business change; a sweeper task publishes it and marks it sent. Avoids "DB committed but enqueue failed". Your Healthcare project uses an email outbox with retries and a dead state.
- **Enqueue after commit:** never enqueue inside a transaction that may roll back (use `on_commit` hooks or the outbox).
- **Idempotent consumers:** deduplicate by key or check current state first.
- **Canvas (workflows):** `chain`, `group`, `chord` (group then callback), `chunks`.
- **Delayed jobs:** `countdown`/`eta`; expiry of holds is a scheduled sweep or a per-hold delayed task.
- **Dead-letter queue:** park permanently failing messages for inspection.
- **Fallback to synchronous execution** if the broker is down (graceful degradation), as in your Placement Portal; with `CELERY_TASK_ALWAYS_EAGER=true` in development/tests.
- **Backpressure:** bound queue length, scale workers on queue depth, reject or degrade when overloaded.

## 5. Using Celery with Flask / FastAPI
- Flask: create the Celery app from the Flask app and wrap tasks in `app.app_context()`.
- FastAPI: enqueue from endpoints; do not call `.get()` on results inside async endpoints (it blocks); return a job id and expose `GET /jobs/{id}`.
- Keep task arguments small and serialisable (ids, not objects); fetch fresh data inside the task.

## 6. Monitoring and ops
Flower for monitoring, Prometheus exporters, logs with task id and request id, alert on queue depth and failure rate, graceful shutdown (`SIGTERM` finishes running tasks), autoscale workers, separate Beat (run exactly one instance; two Beats duplicate schedules).

## 7. Alternatives
| Tool | Notes |
|---|---|
| RQ | simple Redis queue |
| Arq | asyncio Redis queue |
| Dramatiq | reliable, simpler than Celery |
| Huey, APScheduler | light schedulers |
| Kafka / RabbitMQ consumers | event-streaming or advanced routing |
| Cloud queues (SQS, Pub/Sub) + workers | managed reliability |
| Temporal / Airflow / Prefect | long workflows and data pipelines |

## 8. When NOT to use a queue
Short synchronous work, when the caller needs the result immediately, or tiny scale where a thread or `BackgroundTasks` is enough. Queues add operational cost.

## Quick interview Q&A
- **How does Celery work end to end?** Producer sends a message to the broker; a worker pulls it, runs the task, optionally stores the result.
- **What happens if a worker crashes mid-task?** With `acks_late` the task is redelivered (so tasks must be idempotent); with early ack it is lost.
- **How do you guarantee a task runs exactly once?** You cannot; aim for at-least-once delivery with idempotent handling.
- **How do you retry?** `autoretry_for`, `retry_backoff`, `max_retries`; then dead-letter.
- **Redis vs RabbitMQ as a broker?** Redis: simple, fast, weaker guarantees; RabbitMQ: acknowledgements, routing, priorities, durability.
- **What is Celery Beat?** The periodic task scheduler; run a single instance.
- **How do you avoid losing an email when the DB commits but enqueue fails?** Outbox pattern.
- **How do you scale?** More worker processes/nodes, queue per workload, autoscaling on queue length.
- **BackgroundTasks vs Celery?** In-process and non-durable vs distributed and durable.
- **How do you return job status to users?** Store state in DB/Redis; poll `GET /jobs/{id}`, or push by WebSocket/webhook.
