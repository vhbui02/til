# Redis Strategies

## You don't need cache

You don't need cache if your system is purely write-intensive.

But if you do, here are some combination strategies:

### 1. Cache-aside and Write-around strategies

Read operation: app fetch data from cache:

- if it's a cache hit, cache system return the data.
- if it's a cache miss, app query the database database, database system return the data, then app write into cache.

Write operation: app write data into database. On success, the app can:

- write new key-value into cache. Highest performance, but need to handle race condition.
- invalidate key in cache, cache warming will be done ad-hoc/on-demand. Easiest to implement, but less performant.
- do nothing with the cache, rely on its TTL => data inconsistency, don't do this.

**Pros:**

- Save cache storage: cache is warmed on-demand, ad-hoc. Cache automatically be removed after TTL.
- High availability: if cache down, app can query database. If database down, app can read previously cached data (might be staled)
- Cache structure can be different from database schema.
- Suitable for **read-intensive, write-light** systems: social media's avatar, post, friends ...

**Cons:**

- Cache-warming latency: first retrieval hits the DB.
- Consistency & ordering: carefully order DB write vs cache invalidation/update to avoid stale reads.
- Race conditions and stampedes: concurrent misses can overload the DB — mitigate with per-key locks, singleflight, or request coalescing.
- Not suitable for **write-intensive, read-light** use cases: stock price, social media's reaction count, comment count.
- Not appropriate for **read-intensive, write-intensive** use cases also: game online leaderboard.
- Not suitable for systems requiring **strict transactional consistency** system (e.g. banking) without additional mechanisms.
- Not appropriate for use cases that **won't tolerate First Read latency**: trending news posts, trending vidoes, ...
- Requires TTLs, eviction policy, monitoring, and possibly pub/sub invalidation in distributed deployments.

**Implementation:** manual via library (`redis-py` + `mysql-connector-python`, ...).

### 2. Read-through and Write-through

Read operation: app read from cache system:

- cache hit => cache system return data
- cache miss => cache system fetch data from database/object storage/no-sql/... update into cache then return data to app.

Write operations:

1. App write into cache system.
2. Cache system check if the data is different

- If not, return success.
- If yes, sync the changed the data to database **synchronously**/immediately. The Redis server is blocked until a response from the database is received.

3. If database return success, cache system update the cache.
4. If cache system return success, return success to the app.

If either of the writing operations failed, the transaction is rollbacked.

**Pros:**

- Simple: cache system handles the communcation with database.
- Strong data consistency is upheld, suitable for banking, e-commerce, ...
- Transaction logic is now divided into 2 places: app and cache.
- Suitable for **read-intensive** use cases: CDNs, ...

**Cons:**

- Single Point of Failure: If cache dies, the data is loss, the app dies also.
- Database schema and cache must have the same data structure (I wonder if I can workaround it via Lua)
- Not suitable for **write-intensive** systems: each write operation requires updating both the cache and the database, which can make the cache system a bottleneck.
- Not suitable for **writing transient data**: e.g. user session data on theme/language
- Not suitable for **writing computed data**. (I wonder if I can work around it via Lua)

**Implementation:** RedisGears, a self-hosted serverless engine for transaction, batch and event-driven data processing, allowing users to write and run their own functions on data stored in Redis.

### 3. Write-back/Write-behind

Write a lot into cache singularly, then write into database storage in batch.

**Pros:**

- Less write operations on database.
- No need to wait for database to be updated.
- In case of database failure, data is still stored in cache.
- Suitable for **write-in-bulk** use cases: SSD, Log (buffered first, write into file later), Message system

**Cons:**

- Single Point of Failure: If cache dies, the data is loss, the app dies also.
- Eventual Consistency: there will be inconsistency between data in cache and data in database during a small amount of time.
- Not suitable for **strict data consistency** system that won't tolerate Eventual Consistency
- Data consistency management when updating database in bulk is very complex.

## Microservices

- 2PC.
- Saga.
- Transactional Outbox.
