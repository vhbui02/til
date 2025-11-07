# Python Asynchronous Cheatsheet

## `asyncio` API (Python v3.12)

### `asyncio.run()`

Create a new event loop to run the Coroutine (a.k.a _the return value of an `async def` function_), the Coroutine yield the control to the event loop so it can run other Coroutines.

**NOTE:** ASGI server initialize and maintain a global event loop throughout the lifespan of the application instance.

```py
import asyncio

async def main():
  await asyncio.sleep(5)

if __name__ == "__main__":
  asyncio.run(main()) # input: a Coroutine
```

### `asyncio.to_thread()`

Run synchronous _IO-bound operations_ non-blocking by creating a seperate worker/thread and run the code inside it, allowing the event loop (which only runs on the main thread) to be responsive.

> CAUTION: Differentiate between Python Thread and CPU Thread, Python Thread.

The result from the seperate thread is "seemingly" coming from the event loop, but it's actually not. The event loop only organize the execution flow:

- The event loop schedules the thread and await its result.

- The sync function runs outside the main thread.

- If other async operations in the main thread finishes, but the sync operation is still working on seperate thread, the coroutine must wait (or be blocked)

- When the thread finishes and return the result, the event loop resumes your coroutine with the result

```py
import asyncio
from time import sleep
from concurrent.futures import ThreadPoolExecutor

def run_io_blocking_code(x):
  sleep(x)
  print(x)

async def run_io_blocking_code_async_modern():
  # modern, Python v>=3.9
  # NOTE: deep down, it's a wrapper of `ThreadPoolExecutor` and `loop.run_in_executor()`
  await asyncio.to_thread(run_io_blocking_code, 5)

# legacy
async def run_io_blocking_code_async_legacy():
  loop = asyncio.get_running_loop()
  with ThreadPoolExecutor() as pool:
    loop.run_in_executor(pool, run_io_blocking_code, 5)

if __name__ == "__main__":
  asyncio.run(run_io_blocking_code_async_modern())
  asyncio.run(run_io_blocking_code_async_legacy())
```

You have to be careful to not run _CPU-bound operations_ inside `asyncio.to_thread` or `with ThreadPoolExecutor() as pool` since Python will trigger **Global Interpreter Lock (GIL)**.

Your Python process consists of the main thread (where the event loop runs) and a separate worker thread. Due to GIL, only one thread can execute Python bytecode at a time, so one thread must wait while the other completes its task.

Instead, you should use `ProcessPoolExecutor`:

```py
import asyncio
from concurrent.futures import ThreadPoolExecutor

def run_cpu_bound_task(x):
  return x**x

async def run_cpu_bound_task_non_blocking():
  loop = asyncio.get_running_loop()
  with ProcessPoolExecutor() as pool:
    result = await loop.run_in_executor(pool, run_cpu_bound_task, 15)
    return result

if __name__ == "__main__":
  asyncio.run(run_cpu_bound_task_non_blocking())
```

### `asyncio.as_completed()` versus `asyncio.gather()`

Use `asyncio.as_completed()` if:

- You have a list of awaitable objects (Coroutines, Futures (depreciated)).
- You want to run them concurrently.
- You want to process their results as soon as possible, rather than waiting for all tasks to complete or processing them in the order they were started.
- Use cases: streaming, early exists, progressive UI updates.

Use `asyncio.gather()` if:

- You have a list of awaitable objects (Coroutines, Futures (depreciated)).
- You want to run them concurrently.
- You want to process their results all at once, in the order the coroutines was provided.
- Use cases: batch processing

```py
import asyncio

async def fetch_data(x):
  await asyncio.sleep(x)
  return f"Done: {x}"

async def fetch_completed():
  tasks = [fetch_data(3), fetch_data(2), fetch_data(1)]
  for coro in asyncio.as_completed(tasks): # input: an array (or in general, an iterable), consists of multiple awaitables
    result = await coro
    print(result)

async def fetch_gather()
  tasks = [fetch_data(3), fetch_data(2), fetch_data(1)]
  results = await asyncio.gather(*tasks) # input: not an array, multiple awaitables
  print(f"gather: {results}")

if __name__ == "__main__":
  asyncio.run(fetch_completed())
  asyncio.run(fetch_gather())
```

### `asyncio.create_task()` (legacy API: `asyncio.ensure_future()`)

There are two ways to run background jobs.

If you're reading code from the early days of `asyncio` (pre-v3.5) where Coroutines weren't around, `asyncio.ensure_future()` is used to run background tasks/jobs.

However, when Coroutines comes out in v3.5, along with implementing a dedicated API called `asyncio.create_task()` to run background tasks, Python programming language engineers also want to add Coroutines support to `asyncio.ensure_future()`. This turns out to be a bad design decision and now `ensure_future()` is _scheduled_ to be depreciated (hah!)

`asyncio.ensure_future()`:

Pre-v3.5:

- Input: an `asyncio.Future`-like object (including the `asyncio.Task` object, which is the subclass of `asyncio.Future`)
- Output: an `asyncio.Task` object.

Post-v3.5:

- Input: A Coroutine or an `asyncio.Future`-like object.
- Output: An `asyncio.Task`
- Logic: if given a Coroutine, it will be wrapped inside an `asyncio.Task` and calls `asyncio.create_task()` internally.

---

`asyncio.create_task()`: available post-v3.5.

- Input: only a Coroutine.
- Output: an `asyncio.Task`.
- Requires running inside an event loop.

=> Modern way to schedule a Coroutine to a Task.

## Database

To deliver the best possible performance when working with IO-bound or CPU-bound operations, modern Python applications leverages either 3 types of strategies:

- Utilize native Python support for asynchronous functionality (i.e. stable `asyncio` library from Python version 3.5 or more), often requires a compatible web framework (e.g., `Flask` version 2.0 or higher, `FastAPI`, ...) and a pure-ASGI (`uvicorn`, `hypercorn`, `daphne`, ...) or hybrid WSGI-ASGI production server (`uWSGI`, `gunicorn`, ...). In some cases, an asynchronous database driver is needed (for `Flask`, it's `aiomysql` or `asyncmy`)

- Offloading the blocking synchronous operation into a new thread using either:

* `ThreadPoolExecutor`
* `run_in_executor()`

- `asyncio.to_thread()`

* Find the optimal size of the connection pool, given factors such as infrastructure resources, the asynctech stack and the application logic.
* Analyze the connection usage

## References

- [Python's `asyncio` ](https://realpython.com/async-io-python/)
