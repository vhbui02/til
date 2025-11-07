# Hatchet Learning

## Overview

Modern distributed task queue for Python, TypeScript and Go.

**Old way:** manage your own task queue or pub/sub system
**Now:** use Hatchet to distribute functions between a set of workers with minimal configs and infras.

Another shiny tool in your tech stack, but does it necessary?

Client requests an API endpoint that required running a very long and resource-extensive task. Instead of making multiple clients waiting, off-load those tasks to a task queue and return a simple message that get displayed on user-facing UI. Clients will then poll the result to receive data. There are many benefits when doing this.

## Cheatsheet

### [SEVEN best practices when deploying Hatchet Worker](https://docs.hatchet.run/home/workers#:~:text=Best%20Practices%20for%20Managing%20Workers)

1. Reliability: workers need to be run in a stable environment (GNU/Linux distro such as Debian/Ubuntu, least number of software running) with sufficient resources to avoid _resource contention_.

2. Monitor + Logging: track worker health, performance, task execution status

3. Lifecycle Management: automatic restarts on critical failures, include graceful shutdown procedure.

4. Robust Error Handling: worker handle errors, report execution failures, retry tasks based on policy.

5. Secure communcation: skip if your network infra is private/isolated.

6. Scalability: design your system to add/remove workers based on-demand easily.

7. Update: Update Hatchet SDK and Self-Hosted Hatchet Docker Compose regularly, ensure API compatibility with Hatchet engine version.

### Network/Socket client initialization

- Initialize clients during worker startup (process-level) at Quart's `app.before_serving()`, [Gunicorn's](https://docs.gunicorn.org/en/stable/settings.html#server-hooks) `worker_int()`, ...
- In each task, acquire a connection from the pool, use it for short operation then release it immediately.
- If the task is a long-running process, only open the connection at the end of the process.

### Retry policy

- No retry, fail-fast: `ValueError`, auth failures (401), client errors (4xx) which's not related to rate-limited.
- Retry once: Parse error (`KeyError`, `TypeError`). If `KeyError` occurs, that means your code lack defensive programming and should be converted to `ValueError`.
- Retry with exponential backoff: rate-limited responses, `aiohttp.ClientError` (aiohttp), `asyncio.TimeoutError` (asyncio), 5xx response, `TimeoutError`, `ConnectionError`, `RuntimeError`, ... `Exception` in general.

**Practice:**

- Track the number of retry attempts via `ctx.retry_count`.
- Raise `NonRetryableException` to prevent retry, let it fail. This is when manual intervention from engineer is required.
- Extremely careful to non-idempotent operations (e.g. non-GET requests).
- Add a circuit breaker if operation is repeatedly failing.

### Hatchet-Lite

```yml
# cre: https://docs.hatchet.run/self-hosting/hatchet-lite

name: hatchet-lite
services:
  postgres:
    image: postgres:18-bookworm
    command: postgres -c 'max_connections=200'
    restart: always
    environment:
      - POSTGRES_USER=hatchet
      - POSTGRES_PASSWORD=hatchet
      - POSTGRES_DB=hatchet
    volumes:
      - hatchet_lite_postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d hatchet -U hatchet"]
      interval: 10s
      timeout: 10s
      retries: 5
      start_period: 10s
  hatchet-lite:
    image: ghcr.io/hatchet-dev/hatchet/hatchet-lite:latest
    ports:
      - "8888:8888"
      - "7077:7077"
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      # Refer to https://docs.hatchet.run/self-hosting/configuration-options
      # for a list of all supported environment variables
      DATABASE_URL: "postgresql://hatchet:hatchet@postgres:5432/hatchet?sslmode=disable"
      SERVER_AUTH_COOKIE_DOMAIN: localhost
      SERVER_AUTH_COOKIE_INSECURE: "t"
      SERVER_GRPC_BIND_ADDRESS: "0.0.0.0"
      SERVER_GRPC_INSECURE: "t"
      SERVER_GRPC_BROADCAST_ADDRESS: localhost:7077
      SERVER_GRPC_PORT: "7077"
      SERVER_URL: http://localhost:8888
      SERVER_AUTH_SET_EMAIL_VERIFIED: "t"
      SERVER_DEFAULT_ENGINE_VERSION: "V1"
      SERVER_INTERNAL_CLIENT_INTERNAL_GRPC_BROADCAST_ADDRESS: localhost:7077
    volumes:
      - "hatchet_lite_config:/config"

volumes:
  hatchet_lite_postgres_data:
  hatchet_lite_config:
```

Create an API key:

```sh
docker compose -f docker-compose.hatchet.yml exec hatchet-lite /hatchet-admin token create --config /config --tenant-id 707d0855-80ab-4e1f-a156-f1c4546cbf52 | xargs
```

Add client env vars:

```sh
export HATCHET_CLIENT_TOKEN="eyJhbGciOiJFUzI1NiIsImtpZCI6IlRLR0g3USJ9.eyJhdWQiOiJodHRwOi8vbG9jYWxob3N0Ojg4ODgiLCJleHAiOjE3NjgxNTEwNDYsImdycGNfYnJvYWRjYXN0X2FkZHJlc3MiOiJsb2NhbGhvc3Q6NzA3NyIsImlhdCI6MTc2MDM3NTA0NywiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4ODg4Iiwic2VydmVyX3VybCI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODg4OCIsInN1YiI6IjcwN2QwODU1LTgwYWItNGUxZi1hMTU2LWYxYzQ1NDZjYmY1MiIsInRva2VuX2lkIjoiZDNlY2MxMjgtZGFjYi00MjcyLWFiNTQtMWFmMDg0ZTIzNDIwIn0.xeipFQ3XNYiuTowgPo6lUUgN3pkwx12galSD-sZg_Wvf4xBpm1ZpGhwYZK-r4pvJgC7KgIQSSugFZ6jlC3fVuA"
export HATCHET_CLIENT_HOST_PORT=8889
export HATCHET_CLIENT_TLS_STRATEGY=none # tls, mtls, none
export HATCHET_CLIENT_API_URL=localhost:8889
export HATCHET_CLIENT_SERVER_URL=localhost:8889
export HATCHET_CLIENT_NAMESPACE=dev
```

### FIVE ways to run a task

1. Enqueue, then wait for results.
2. Enqueue, no wait (a.k.a fire-and-forget pattern).
3. Schedule a task at a specific time in the future.
4. Schedule a task with a cron schedule.
5. Event-driven.

[**Enqueue, then wait for results:**](https://docs.hatchet.run/home/run-with-results) best for:

- running multiple concurrent tasks AND require waiting (a.k.a fan-out pattern).
- running long task AND critical task AND require waiting.

[**Enqueue, no wait (fire-and-forget)**](https://docs.hatchet.run/home/run-no-wait):

- long-running, non-urgent tasks that should be notified on the user interface that it's running, but without requiring immediate results.

[**Schedule a task at a specific time in the future**](https://docs.hatchet.run/home/scheduled-runs):

- Define a timestamp in your task definition to trigger the task. This is done manually by developers, useful if you know the specific time.
- Tasks required programmatically/dynamically.

[**Schedule a task using cron**](https://docs.hatchet.run/home/cron-runs):

- Define a cron expression in your task definition to trigger the task. This is done manually by developers, useful if you know the specific time.

- Dynamically/Programmatically set the cron schedule of a task. This is what I used the most.

[**Cron Best Practices:**](https://docs.hatchet.run/home/cron-runs#:~:text=Cron%20Considerations,-When)

1. Cron schedules are ALWAYS UTC.
2. Execution time: even the enqueue time is not guaranteed, let alone execution moment. Every task is executed with best1.effort strategy.
3. Missed schedule: if missed, it will NOT run again.
4. Overlapping schedule: if a task is still running, but the next scheduled time is due, it will start a new instance of the task, or respect the concurrency policy (one of the three concurrency strategies).

[**Event-driven**](https://docs.hatchet.run/home/run-on-event)

- Run a task when an ephemeral event is received.
- Run multiple independent tasks in response to a single event. (e.g. after signup successfully, you will want to run a lot of task)

```py
# ============================================================================ #
# workflow.py                                                                  #
# ============================================================================ #

EVENT_KEY = "user:create"
SECONDARY_KEY = "foobarbaz"
WILDCARD_KEY = "subscription:*"

class EventWorkflowInput(BaseModel):
    should_skip: bool

event_workflow = hatchet.workflow(
    name="EventWorkflow",
    on_events=[EVENT_KEY, SECONDARY_KEY, WILDCARD_KEY],
    input_validator=EventWorkflowInput,
)

# ============================================================================ #
# runner.py                                                                    #
# ============================================================================ #

hatchet.event.push("user:create", {"should_skip": False})
```

### SEVEN key concepts

1. Queueing:

- Enqueue tasks, send them to workers
- Control the sending rate that workers can handle.
- Less requests kept waiting on web servers === less hardware resources being wasted to keep their states while waiting.

2. Orchestration:

- Fan-out Fan-in pattern: a workflow running multiple parallel tasks. Each of the task can be run independently, via different workers hosted on different machines.

**Orchestation mechanism:**:

- **DAG:** _workflow_ - a collection of tasks where multiple tasks can be run concurrently, prior tasks' output is subsequent tasks' input.

- **Durable tasks:** tasks are stored inside a durable storage (up to a retention period). In case of a network partition between web servers and worker servers, tasks aren't lost but being kept safely. Not only it proceeds to retry/replay the task automatically but also send alert to engineers.

> **NOTE: retryable tasks should be idempotent to prevent errors causing by multiple identical tasks get replayed**

3. **Flow Control:** or Concurrency Limit on a per-user, per-tenent or per-queue basis.

- Fairness for all clients.
- Hardware resource management.
- Avoid race condition when working on shared resources.
- Compliance with external service limits
- Even after employ fairness, spike can occur when you simply have too many users trigger an event.

E.g. max 5 tasks/s, max 20 tasks/60s

**Concept of Slots:** number of concurrent _task_ runs that a worker can execute, configured via `slots=` argument in `hatchet.worker()`.

If the number of slots are all occupied, later tasks will wait in queue before picking up.

**Concept of `max_runs=`:** number of concurrent _tasks_ from a specific workflow that can execute among all workers that registered the workflow. If the number of Slots combined from all workers that registered the workflow exceeds the `max_runs=` numbers, the remaining Slots are distributed to remaining workflows, or idle (don't let it idle).

**CRITICAL:** slot-level concurrency only helps when it's not bottlenecked by CPU, memory or network bandwidth, that's why sometimes increase your slots won't increase your throughput.

[**THREE strategies:**](https://docs.hatchet.run/home/concurrency)

- Round Robin: Distribute task instances across available slots using `key` function.
- Cancel In Progress: Cancel currently running task instances with the same concurrency key.
- Cancel Newest: Cancel newest task instance with the same concurrency key.

4. **One-time scheduling and cron scheduling:**

- Cron schedules are ideal for data pipeline, batch processing and notification systems.
- Durable sleep: pause execution of a task for a specific duration.

5. **Task routing:** the default Hatchet behevior is impliment a FIFO queue.

- Sticky assignment: `SOFT` mode allows spawned tasks to prefer execution on the same worker, but can be scheduled to run on another worker if that same worker is busy. `HARD` mode require execution on the same worker.
- Worker affinity: ranks workers to discover which is best suited to handle a given task.

6. **Event triggers and listeners:** support event-based architectures

- Event listening: internal tasks and workflows can pause execution while waiting for a specific external event.
- Event triggering: external events can trigger new workflows or a step in a workflow.

7. **Observability:**

- Tasks, workflows, queues can be monitored in a real-time built-in web dashboard.
- Built-in logging supported with colorful message levels.
- Real-time alerting via Slack or email with adjustable window.

## Deployment: Self-hosted versus Official SaaS Cloud

I choose to self-host. Why spending money when you have time and on-premises resources to setup?

**FIVE** components in a self-host Hatchet:

1. API server: REST + gRPC APIs for workflow management
2. Engine: Workflow orchestration + Task scheduling
3. Database: PostgreSQL, store workflow state + metadata
4. Message Queue (optional): RabbitMQ for inter-service communication, high throughput real-time updates.
5. Dashboard: Web UI for monitoring workflows + debugging.

Workers (or worker processes) are OS processes that execute workflow steps. It's not inside the self-host Hatchet. It connects to the self-hosted control plane.

## Hatchet vs Library-based queues (Celery, BullMQ) backed by Redis Pub/Sub, Redis Stream, RabbitMQ, ...

Unlike Hatchet, they're not a dedicated background task management platform. One of the major drawbacks is that they don't have built-in real-time dashboard for logging and monitoring and often required a 3rd-party solution (e.g. Celery Flower)

If the tasks are simple, they might be a better choice due to simplicity. But when the tasks become more complex, these queues become difficult to debug, log or monitor the unexpected error.
