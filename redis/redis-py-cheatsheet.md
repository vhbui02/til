---
description: My redis-py cheatsheet for learning.
applyTo: "**/*.py"
---

# `redis-py` Cheatsheet

## `set/get` and their variants

NOTE: all of the APIs supported both sync and async.

```py
import redis

r = redis.Redis(decode_responses=True)
r.ping()

r.set("full_name", "john wick")  # True
r.exists("full_name")  # 1
print(r.get("full_name"))  # "john wick"

# ============================================================================ #

# THREE ways to add TTL to a key
r.set("sensitive_key", "sensitive_value")
r.expire("sensitive_key", 100)  # set TTL in seconds

# or set with expiration in same call
r.set("sensitive_key", "sensitive_value", ex=100)

# or use setex(name, seconds, value)
r.setex("sensitive_key", 100, "sensitive_value")  # True

print(r.ttl("sensitive_key"))  # 100

# ============================================================================ #

# set multiple key-value pairs at once
r.mset({
  "first_name": "John",
  "last_name": "Wick",
  "job_title": "Underworld Assassin",
})
print(r.mget("first_name", "last_name", "job_title"))  # ['John', 'Wick', 'Underworld Assassin']

# list operations: prepend (LPUSH) and append (RPUSH)
r.lpush("list_1", "one")
r.lpush("list_2", "four", "three", "two")

r.rpush("list_3", "one")
r.rpush("list_4", "two", "three", "four")

# insert BEFORE or AFTER an existing pivot value
r.linsert("list_4", "BEFORE", "four", "three_and_a_half")
r.linsert("list_4", "AFTER", "four", "five")

# hash (map) operations
r.hset("user:1", mapping={
  "first_name": "John",
  "last_name": "Wick",
})f
print(r.hgetall("user:1"))  # {'first_name': 'John', 'last_name': 'Wick'}
print(r.hget("user:1", "first_name"))  # 'John'
print(r.hmget("user:1", "first_name", "last_name"))  # ['John', 'Wick']

# set
r.sadd("set_1", "one", "two", "three", "one")
r.zadd("sset_1", {"last": 100, "first": 1, "middle": 50})

# sorted set, ...
```

`get_or_set` design pattern:

```py
async def get_or_set(client, key, delegate):
  # NOTE: this function will specifically handle Hash value
  val = await client.hgetall(key)

  if val:
    return val

  val = await delegate() # fetch value from DB, file, ...
  await client.hset(key, mapping=val)
  return val
```

## Connect/Disconnect

```py
import redis
import redis.asyncio as redis
import ssl

# ================================================================== #
# Connection
# ================================================================== #

# connect to a Redis instance
# an interal ConnectionPool is created
client = redis.Redis(
  host="localhost", # or 127.0.0.1
  port=6379,
  username="foo",
  password="bar",
  encoding='utf-8', # def='utf-8' also
  decode_responses=True, # def=binary response
  # if the specified data is byte, Redis don't need conversion step
  # if the specified data is string, Redis will need to convert it into byte using `encoding` value, then encoded value will be writtin into Redis
  # for Hash type, even the name of the field is stored in bytes
  ssl=True, # external connections between client and server, False if internal connections between microservices or using self-signed SSL
  ssl_check_hostname=True, # same as `ssl`
  ssl_min_version=ssl.TLSVersion.TLSv1_3,
  ssl_cert_reqs="none",
  protocol=3, # better for newer project due to its simplicity when compared to RESP 2
  health_check_interval=2,
)
print(f"Ping: {await client.ping()}")

# TODO: learn about AWS Secret Manager when deploying in production

# TCP socket connection
client_tcp = redis.from_url("redis://localhost:6379?ssl_cert_reqs=none&decode_responses=True&health_check_interval=2&protocol=3")

# NOTE: double 'ss' => 'diss'
# SSL wrapped TCL socket connection
client_tcp_ssl = redis.from_url("rediss://localhost:6379?...")

# Unix Domain connection
client_unix = redis.from_url("unix://...")

# explicit close single connection
# CAUTION: also close internal connection pool!
await client.aclose()

# ================================================================== #
# Connection Pool
# ================================================================== #


# Method 1: Pool initialization via constructor
pool = redis.ConnectionPool(
  host="localhost",
  port=6379,
  # **config
  max_connections=992, # https://stackoverflow.com/a/63402062/9122512
  health_check_interval=2,
  socket_timeout=2.5,  # balance network latency sensitivity (lower) or long blocking commands (higher)
  socket_connect_timeout=1.0,  # must be shorter than `socket_timeout`, fail fast on connection attempt helps orchestration detech unhealthy nodes
  socket_keepalive=True,  # best practices is to let OS detect dead peers
  retry_on_timeout=False,  # True if operations are idempotent and network blips are common
)

# Method 2: Pool initialization via static method
pool2 = redis.ConnectionPool.from_url(
  "redis://localhost:6379?...",
  connection_class=redis.SSLConnection,
)

# not recommended: `client` instance now take ownership of the pool
# if it's closed, the pool is also being disconnected implicitly
client = redis.Redis.from_pool(pool)
await client_from_pool.aclose()

# recommended: share connection pool with multiple clients
# Redis instance closed, but not the pool
client1 = redis.Redis(connection_pool=pool2)
client2 = redis.Redis(connection_pool=pool2)
#clientX = redis.StrictRedis(connection_pool=pool) # StrictRedis for backward-compatibility
await client1.aclose()
await client2.aclose()

# after than, explicit closed the pool
await pool.aclose()
```

## Transactions and Pipelining

To achieve atomicity, engineers create a transaction by specifying a combination of `WATCH`, `MULTI` and `EXEC` commands + optimistic locking handle strategies (return error or retry).

From the perspective of a CLI user, the concept differs when applied to a programming language and library such as `redis-py`. Enter _Pipelining_, which batches the commands and send them in one go.

**The pipeline is executed as a transaction by default**, hence `transaction=True` by default.

- If `transaction=True`, wrapping `pipe.multi()` or `pipe.exec()` is **unnecessary in Python code** since **all commands will be wrapped with a `MULTI` and `EXEC` on Redis instance**. While being inside a transaction, Redis will defer commands from other clients to run this sequence of commands firsthand.

- If `transaction=False`, the commands will still be batched up and sent in one go, **but they won't be wrapped between `MULTI` and `EXEC` on Redis**. If not inside a transaction, Redis will not defer commands from other clients and therefore can run commands in-between the commands in sequence.

Now, about the `watch()` method. It turns a transaction into a _conditional transaction_: if the watched keys aren't having their value changed by other clients between the execution of the `watch()` method and `exec()` method, the transaction is deemed successfully, otherwise, it failed.

**NOTE:** `watch()` is only implemented in `client.pipeline()` instance, there is no implementation on `watch()` method in `Redis` instance (a warning will be displayed if somebody try to run).

Q: If you called `watch()` method, you will have to call BOTH `multi()` and `exec()` explicitly and set `transaction=False` optionally ?

A: Yes. Since if `watch()` is called inside the context manager of the pipeline with `transaction=True`, it means the `watch()` will be called **INSIDE** the `MULTI-EXEC` block on Redis instance, after the `MULTI` command. **This is incorrect because `watch()` has to be run BEFORE the `MULTI` command**. After calling `watch()` method, and if there is no `multi()`, all commands that are called on the pipe (getting keys, setting keys) are **immediately** sent to Redis instance, they aren't waited for `exec()` to run. **The commands are no longer batched**. Calling `exec()` now is like sending an empty queue.

```py
# Incorrect
async with r.pipeline(transaction=True) as pipe:
  await pipe.watch("keyWatch") # breaks the batching
  #await pipe.multi() => missing
  await pipe.incr("keyWatch") # get sent immediatly to Redis! but no multi() has been run before => watch() deemed this operation affect an un-existed transaction
  await pipe.execute() # there's no multi() => there is no queue => you send nothing

# what happens on Redis instance
# > redis-cli monitor
# OK
# [127.0.0.1:46368] "GET" "keyWatch"
# [127.0.0.1:46368] "WATCH" "keyWatch"
# [127.0.0.1:46368] "INCRBY" "keyWatch" "1" => no prior multi() => shoot your own foot
# [127.0.0.1:46368] "MULTI"
# [127.0.0.1:46368] "EXEC" => sending empty queue
```

An exception is thrown:

```py
raise WatchError("Watched variable changed.")
redis.exceptions.WatchError: Watched variable changed.
```

```py
# Correct
async with r.pipeline(transaction=True) as pipe:
  await pipe.watch("keyWatch") # breaks the batching
  await pipe.multi()
  await pipe.incr("keyWatch") # now get batched
  await pipe.execute()

# what happens on Redis instance
# > redis-cli monitor
# OK
# [127.0.0.1:46342] "GET" "keyWatch"
# [127.0.0.1:46342] "WATCH" "keyWatch"
# [127.0.0.1:46342] "MULTI"
# [127.0.0.1:46342] "INCRBY" "keyWatch" "1"
# [127.0.0.1:46342] "EXEC"
# [127.0.0.1:46342] "GET" "keyWatch"
```

**NOTE:** since no batching is made, round-trip time (RTT) after the execution of a command between `multi()-execute()` block can negatively impacting performance.

```py
import redis.asyncio as redis

redis_client = await redis.from_url("redis://localhost")

async with redis_client.pipeline(transaction=True) as pipe: # type: aioredis.Pipeline
  # buffer all commands in-memory and compile into batches
  # each commands (i.e. get, set, ...) will return the aioredis.Pipeline, allowing commands chaining
  # the commands will be reflected in Redis until `execute()` is called
  ok1, ok2 = await pipe.set("key1", "value1").set("key2", "value2").execute()

assert ok1
assert ok2
```

Prevent race condition by optimistic locking and retry:

```py
await pipe.watch("user:123:balance") # WATCH, prevent race condition

# NOTE: get() must be run before multi()
# NOTE: required if you need immediate data for client-side validation, long-running IO-bound operations, conditionally computed value...
# NOTE: if get() run after multi(), then it's just being queued (not executed immediately) and cannot be used for immeidate decision making.
current_balance = await pipe.get("user:123:balance") # GET

# WATCH + GET ("read-then-decide") === optimistic lock

# CAUTION: long-running IO-bound operation will increase the duration between watch() and execute(), increase the chance of value in watched keys changed.
# CAUTION: However, it's not really a problem if the modification boundary is wrapped around one user with the possibility of using multiple devices (race condition)
current_balance = await get_or_set(pipe, "user:123:balance", fetch_user_balance_from_db)
current_balance = int(current_balance or 0)

if current_balance < 100:
  raise ValueError("Insufficient balance.")

# enter transaction mode
await pipe.multi()
await pipe.set("user:123:balance", current_balance - 100)
await pipe.lpush("transactions", "debit:100:user:123")

# execute atomically
# if any of the WATCH-ed keys are modified by anoother client between watch()
# command and exec() command, the entire transaction is aborted
await pipe.execute()
```

## Redis Sentinel

Redis Sentinel is the master-slave functionality that increase Redis's high availability.

```py
import asyncio
from redis.asyncio.sentinel import Sentinel

# hostname + port
sentinel = Sentinel([("localhost", 26379), ("sentinel2", 26379)], socket_timeout=0.1)

# return current master node's credentials
host, port = sentinel.discover_master("service_name")

# connect to current master node, handle failover to ensure connection reliability
redis_client = sentinel.master_for("service_name")

ok = await redis_client.set("key", "value")
assert ok

val = await redis_client.get("key")
assert val == b"value" # NOTE: by default, Redis return byte type data
```

## References

- [`redis-py` Official Documentation's "Examples"](https://redis.readthedocs.io/en/stable/examples)
- [Duong's Blog "Kết nối với Redis bằng aioredis và Python"](https://duongnt.com/aioredis-vie/)
- [Peter Parker's comment on SOF](https://stackoverflow.com/a/77585776/9122512)
