---
description: My MySQL Connector/Python learning cheatsheet.
applyTo: "**/*.py"
---

# MySQL Connector Python Cheatsheet

## Asynchronous library `mysql.connector.aio`

### Basic syntax

Single connection database query:

```py
from mysql.connector.aio import connect, MySQLConnection, CMySQLConnection


async def main():
  conn = await connect(user="abc", password="abc123", database="abc_db")
  # or conn = MySQLConnection(user="abc", password="abc123", database="abc_db")
  # or conn = CMySQLConnection(user="abc", password="abc123", database="abc_db") # if C extension is installed
  cur = await conn.cursor()

  await cur.execute("SELECT * FROM users;")
  # NOTE: if you're running DML, you don't have to call cur.fetch*() API
  results = await cur.fetchall()
  print(results)

  # CAUTION: remember to explicitly closed them
  await cur.close()
  await conn.close()

# more recommended

# separate configuration
config = {
  "host": "127.0.0.1",
  "port": 3306,
  "user": "abc",
  "password": "abc123",
  "database": "abc_db",
  "use_pure": True, # C extension mode, faster
  "pool_name": "pool_abc",
  "pool_size": 20, # def=5
  "pool_reset_session": True
}

# passing connection instance
async def get_users(conn):
  # running directly without cursor
  await conn.cmd_query("SELECT * FROM users;")
  results = await conn.get_rows()
  print(results)

# use context manager
async def main2():
  async with await connect(**config) as conn:
    async with await conn.cursor() as cur:
      await cur.execute("SELECT @@versions;")
      results = await cur.fetchall()
      print(results)

      users = await get_users(conn)
      print(users)

      # no need to explicitly close cur or conn here

if __name__ == "__main__":
  asyncio.run(main2())
```

The same database connection can be used to execute multiple _sequential_ queries, but not for multiple async _concurrent_ queries. Reason is that one query can receive the result of the other quety.

They must be executed by MULTIPLE database connections, but the cost of establishing and closing a database connection is too much when scaled. Therefore, modern applications use an async-safe connection implementation (hard) or a connection pool that reuse database connections (easy) to reduce this overhead.

As of `2025-03-10`, connection pooling was not supported in `mysql.connector.aio` [Booboo's answer on SOF](https://stackoverflow.com/a/79498669/9122512). However, by `2025-09-02`, official support for connection pooling was added and verified in my tests. This eliminates the need to implement a custom connection pool.

```py
async def main():
  conn = await connect(pool_name="test", pool_size = 3, **config)
  cur = await conn.cursor()
  # ...
```

## References

- [Bill Karwin's answer on SOF](https://stackoverflow.com/a/65169821/9122512)
- [MySQL Official Documentation: MySQL Connector/Python Asynchronous](https://dev.mysql.com/doc/connector-python/en/connector-python-asyncio.html)
