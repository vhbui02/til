# Python Concurrency Models

<!-- tl;dr starts -->

While trying to find the best way to maximize Apify free tier's resource utilization without touching its limit, I discovered 1 of 3 Python Concurrency Model: Asynchronous IO (short for "Async IO"). There are 3 different concurrency models in Python: Multiprocessing, Threading and Asynchronous IO.

<!-- tl;dr ends -->

## Cheatsheet

### Synchronous

- For synchronous WSGI server such as `uWSGI`, use workers/threads or background workers to off-load blocking operations. It requires carefully sizing thread pools and DB max conections, also more memory and CPU. Therefore it's considered inferior to proper async library with ASGI server.

- Implementing a Python thread pool and a DB connection pool can make your code error-prone, make your code more complex, have memory issues when scale. but it's the simpliest way to keep most of your synchronous stack intact. Beside, it's still bettern than using barebone synchronous operations.

- Tweak the number of possible DB connections:

`processes * Python thread pool size (MySQL connection pool size - the number of DB connections to keep) <= max_connections`

- Use `ThreadPoolExecutor` + MySQL queue-based connection pool so threads can re-use MySQL connection instead of a new one for every request. HOWEVER, this pattern is only for `SELECT` queries, since connections will be re-used, you don't know when it will be closed nor when to commit, to finish complete the transaction.

```py
app = Flask(__name__)

DB_CONF = {
  # host: ...
  # user: ...
  # password: ...
  # db: ...
  # cursorclass: ...
  # autocommit: True, #
}
```

### Asynchronous

- Flask version >=2 support async view functions, therefore can run async operations (run DB query, send HTTP request, read/write file inside filesystem, ...) with the support of native Python event loop integration by `asyncio`, under ASGI server (e.g. `uvicorn`, `hypercorn`, ...).

- Asynchronous database operations on synchronous frameworks (e.g. Flask) required async driver (e.g. `aiomysql`/`asyncmy`).

```py
import asyncio
import aiomysql
from typing import Any

# Constants
DB_HOST = "127.0.0.1"
DB_PORT = 3306
DB_USER = "root"
DB_PASSWORD = ""
DB_NAME = "mysql"
POOL_MIN_SIZE = 1
POOL_MAX_SIZE = 5
AUTOCOMMIT = False

async def run_query() -> Any:
    pool = await aiomysql.create_pool(
        host=DB_HOST,
        port=DB_PORT,
        user=DB_USER,
        password=DB_PASSWORD,
        db=DB_NAME,
        minsize=POOL_MIN_SIZE,
        maxsize=POOL_MAX_SIZE,
        autocommit=AUTOCOMMIT,
    )
    async with pool.acquire() as conn:
        async with conn.cursor() as cur:
            try:
                await cur.execute("SELECT 10")
                result = await cur.fetchone()
                await conn.commit()
                return result
            except Exception as exc:
                await conn.rollback()
                raise exc
    pool.close()
    await pool.wait_closed()

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    result = loop.run_until_complete(run_query())
    print(result)
```

## Overview

![python concurrency model diagram](concurrency-model.svg)

- **Parallelism**: performing multiple operations at the same time.

- **Concurrency**: broader than _parallelism_, suggests that multiple tasks run in an overlapping manner. Concurrency does not imply parallelism, but the vice versa is true.

<!-- prettier-ignore -->
| **Concurrency models** | Multiprocessing | Threading | Asynchronous IO (Async IO) |
| ---------------------- | --------------- | --------- | -------------------------- |
| **CPU** | Many | One | One |
| **Multitasking Strategy** | Preemptive | Preemptive | Cooperative |
| **Switching Decision** | The processes all run at the same time on different processors (CPU cores) | The OS decides when to switch tasks that are external to Python | The tasks decide when to give up control |
| **Use Cases/Examples** | **CPU-bound tasks** (number-crunching, ...) | **IO-bound tasks** (file operations, database queries, ...) | **High-volume IO-bound tasks** (web servers handle 1000s of WebSocket connections, ...) |
| **Synchronization Coordinator** | IPC | Locks/Semaphore | No locks, required developers write non-blocking code |
| **Limitation by Global Interpreter Lock (GIL)** | No, each process run on its own Python Interpreter | Yes, threads are forced to run in sync mode, and No with *free threading* introduced since Python 3.13 | No, single-threaded |
| **Scalability Limitation** | Number of CPU cores | GIL (suitable for few threads) | All awaited tasks are long-running ones |
| **Code Complexity** | **High** (IPC, memory separation, ...) | **Moderate** (race condition, OS-level management, GIL limitation...) | **Low/Moderate** (non-blocking code required) |
| **Package** | `multiprocessing` (low-level), `concurrent.futures` (high-level) | `threading` (low-level), `concurrent.futures` (high-level) | `asyncio` |
| **Performance Optimization** | Avoids GIL, uses as much cores as possible | Only blocking libraries are available | Non-blocking libraries are available as well |
| **General** | **Multiple system processes** run multiple ToTs at the same time | **Single process**, **multiple threads** can only run 1 ToT, but cleverly take turns to speed up the overall process | **Single process**, **single-threaded** can only run 1 ToT, but cleverly take turns to speed up the overall process |

## Introduction: What is Concurrency?

Dictionary definition: **simultaneous occurrence**. So, what are the things that are "occuring simultaneously"?

- Process
- Thread
- Task

At a high-level, they're the same and all refers to "a sequence of instructions that run in order".

**Analogy**: a series of **trains of thought** (now refers to as ToT). Each one can be stopped at certain points, the brain can switch to a different one. The state of each train is saved so it can be restored where it was interrupted.

But at a much lower-level, they represent slightly different things:

- Process: a collection of resources including memory, file descriptors, ... that's unshareable between other processes (that's why Inter-Process Communication (IPC) existed). Overall, a completely different program.
  - In Python context, each process runs in its own Python Interpreter.
  - Each ToT can run on a separated CPU core.

## Multitasking strategies

- **Preemptive Multitasking:** OS-level management, OS knows about each process/thread and interrupts it _at any time_ to start calling a different process/thread.
  - Both easy and hard, depends on whether or not the code need to implement context switch from just a trivial `x = x + 1`.
- **Cooperative Multitasking:** No OS-level management. Tasks cooperate with each other by announcing when they're ready to be switched out:
  - Much easier to implement than Preemptive Multitasking. Know where task will be swapped out => Easier to read the execution flow.

## 2 problems that Concurrency addresses

- IO-bound: cause program to slowdown because it needs to wait for input or output from external resource, things that are much slower than CPU. E.g. File system, Network connection, ...

![io bound diagram](https://files.realpython.com/media/IOBound.4810a888b457.png)

=> Solution: overlap the times spend waiting by doing something else in the meantime.

- CPU-bound: the resource limiting the speed of the program is the CPU.

![cpu bound diagram](https://files.realpython.com/media/CPUBound.d2d32cb2626c.png)

=> Solution: do more computations in the same amount of time.

## Python Concurrency Models

### Multiprocessing

Utilizes 2 Python packages: `concurrent.futures` (high-level) and `threading` (low-level).

```py
threading.local() #
```

### Threading

### Asynchronous IO (Async IO)

## References

- [Brad Solomon's "Async IO in Python: A Complete Walkthrough", 2019-01-16, RealPython](https://realpython.com/async-io-python/#a-full-program-asynchronous-requests)
- [Jim Anderson's "Speed Up Your Python Program With Concurrency", 2024-11-25, RealPython](https://realpython.com/python-concurrency/)
