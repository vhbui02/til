# Why you should not run background tasks directly on server?

You can see these APIs when reading Python asynchronous code:

- `asyncio.create_task(callback, *args)`
- `quart_app.add_background_task(callback, *args)`
- ...

They're, half-measures. You missed the benefits of decoupling when processing the tasks  (technically speacking, the coroutines) using the same event loop that handle the request/response.

In the case of synchronous programming, Python threads are spawned and run inside `ThreadPoolExecutor` to achieve non-blocking, with the cost of hardware resources.

If the process crashes, the task is lost. If the task is failed, no retry mechanism. You have integrate your own logging and monitoring. 

Therefore the best use cases are local, ephemeral background work such as pushing client's behavior metrics, or cleanup logic.

=> Use a dedicated distributed task queues such as Hatchet, Celery, ... for long-running, resource-intensive tasks.
