# Task Queue

## Overview

<!-- TODO: write overview -->

## Terminologies

> [!CAUTION]
>
> All terminology is presented in the context of working with Python.

- Server process (sometimes also called "worker"): OS processes spawned by a program typically running a web framework that're hosted on one or more web servers/nodes. The OS process receives API requests, enqueues tasks to a message broker, and returns responses to clients; it may run on the same or a different host as worker processes.
- Thread: in the context of Python, a Python process contains many Python threads (CAUTION: not to be confused with OS threads) that can run non-blocking concurrently. They're suitable for IO-bound tasks yet vulnerable to CPU-bound tasks due to Global Interpreter Lock triggering when a CPU-bound task is run on one of the threads. If so, at a single point of time, only one thread is allowed to run.
- Worker process: OS processes spawned by a program typically running a background task management platform that're hosted on one or more cron servers/nodes. It continuously pulls tasks/jobs from a message broker queue and executes the actual work; it can run on the same or a different host as the server process.
- Task/Job: a discrete unit of work, typically represented as a function or a method call with arguments, that is scheduled for asynchronous execution and may require significant hardware resources (CPU, memory, disk, ...) and take a lot of time to finish.
- Broker/Message Broker: middleware (software-in-the-middle) that reliably queues, stores, and routes tasks from server processes to worker processes.

## Why you need distributed task queue?

Here is a simple workflow when you're running long-processing logic directly on an WSGI server:

- Multiple clients send multiple requests.
- If a WSGI/ASGI server is used, one server process taken from a pool of many pre-started server processes take in the request.
- If it's an I/O-bound task: for a WSGI server, a new thread is spawned and runs inside a `ThreadPoolExecutor`. For an ASGI server, only one thread is used; the server process yields control to the event loop and enqueues the task.
- If it's a CPU-bound task, a new process is spawned and runs inside a `ProcessPoolExecutor` to avoid blocking other threads or the event loop, ensuring that the `Global Interpreter Lock (GIL)` does not hinder concurrent execution.
- Hardware resources are being occupied.
- Results are made and server proces return a response.
- Clients display the result on the UI.

**Now "worker" here can be:**

- A new OS process spawn by Apache server to run PHP code in old LAMP stack. This option costs too much resources.
- A serverless (e.g. AWS Lambda) spin up a new virtual machine/container to execute your code in AWS's infrastructure. This option has cold start problems
- An existing OS process spawned prior which happened to be available at that time if you're running Python modern dedicated WSGI server (e.g. `gunicorn`). **This option is the most viable and most popular, but the availble workers will soon run-out and everything slow downs.**

**Problems with the asynchronous methods:**

- CPU lock up due to CPU-bound operation clicked in.
- Deferred I/O-bound operations (such as network requests, database queries, or filesystem read/write) consume minimal CPU and memory. However, as their number increases, your server will quickly run out of resources and become unable to handle new requests efficiently.

**"Queueing theory":** There are 3 results you need to know:

- As utilization approaches 100%, wait times = infinity.
- If tasks comes faster than processing speed, the queue will grow.
- If the queue is full, start dropping requests.

If vendor goes down, server workers become busy, yet requests keep coming and start queueing (deep down, occupying your memory) without the ability to clear them (drop client's requests === saying: my app is incompetent, I have to cancel your request). 

=> You will need: task queue, and, deferred result UI.

## Task queue

Here is a simple workflow when you're running long-processing logic inside task queue

- Multiple clients send multiple requests.
- If a WSGI/ASGI server is used, one server process taken from a pool of many pre-started server processes take in the request.
- If it's an I/O-bound task: for a WSGI server, a new thread is spawned and runs inside a `ThreadPoolExecutor`. For an ASGI server, only one thread is used; the server process yields control to the event loop and enqueues the task.
- If it's a CPU-bound task, a new process is spawned and runs inside a `ProcessPoolExecutor` to avoid blocking other threads or the event loop, ensuring that the `Global Interpreter Lock (GIL)` does not hinder concurrent execution.
- Hardware resources are being occupied.
- Server process off-loading the long running task to message broker.
- Message broker queue the tasks and sent it to worker process.
- Worker process pick up work from the queue and start processing.
- Server returns response work-in-progress message.
- Client receives work-in-progress message, start polling for results.
- Server returns the pending status of the task.
- Task worker finish the task, write results to a database, 3rd-party API, ...
- Server returns the success status of the task + result.

This is a distributed system, so each component described can run on multiple hosts or nodes, within multiple containers, and across multiple clusters.

### Benefits

- Respond to user requests quickly, do the long task later.
- Also, long task doesn't have to run at one place, it can be scaled horizontally (not on the same system will you, it's like put all of the eggs in one basket with your shoes unties)
- Set timeout of long tasks.
- High reliability and availability via storing pending tasks inside durable storage (power outage, disaster, ...) and automatically reassigned to other workers (if they are still alive, hopefully). If all workers are dead, tasks can be replay later without losing a single task.
- Buffering to handle spike demands with limited resources.
- Retry mechanism for partial (some parts of a system or operation fail, while others succeed, such as API rate limit) and intermittent (unexpected error happens infrequent, succeed if retry) failures. Setup exponential backoff for partial failures, limit number of retries, log bad tasks so engineers can fix them ASAP.
- Observability: seamless integration with robust logging and monitoring systems (ELK, Grafana+Loki+OpenTelemetry, ...)
- Flexible strategy: Exactly-once, At-least once (a.k.a best effort), ...
- Message Broker and/or Worker process are all Serverless: it's like tiger with wings.

  + Message Broker such as SQS, SNS, ... tasks can be processed asynchronously by default (no thread). Retrying, buffering, batching, scaling, ... are simplified.
  + Server process calling AWS Lambda directly will lose the benefits of decoupling. The maximum number of concurrent Lambda invocations the client can initiate is limited by async operation counts/thread count (if sync), in general, system resources. Therefore, it's not recommended. 

> However, task must be idempotent. Exponential backoff may execute the same task multiple times if faiilure occur.


