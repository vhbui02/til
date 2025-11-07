# MySQL Isolation Level

## Overview

In 1980, Andreas Reuter and Theo Harder coined the term "ACID" - a set of properties regarding a database transaction to keep the data stored reliably and with integrity.

## ACID compliance

- **Atomic:** a number of SQL statements must be executed as a single unit of work (a.k.a transaction). If any of the statement fails, the changes made by prior statements must be roll backed.

- **Consistency:** database can be confidently be moved from one state to another (constraints, cascading effects, triggers, ...)

- **Isolation:** transactions are executed independently and in a controlled and ordered way since many clients are connected to a single database and execute transactions concurrently. _MySQL isolation levels_ relate directly to this property.

- **Durability:** Once a transaction completes, the data must survive system failures and outages by storing the data on persistent storage.

## Isolation violations (or _violations_)

Violations are "read phenomena" that occur when transactions are executed concurrently, and these behaviors are what isolation levels seek to prevent.

Depending on the selected MySQL isolation level, one or more violations may be permitted, introducing query inconsistency in the exchange for performance.

## THREE common violations

### 1. Dirty Read

A query within one transaction returns the new version of a piece of data that has been made by another concurrent transaction but has not committed to the database yet.

Reading uncommited data can lead to unexpected behavior.

E.g. 2 customers buying the last CD. One customer visit the produce page and made the purchase request. The system will conduct a series of operations on the database inside a transaction, one write operation will be:

"Make the quantity of that product in-stock equal to 0 (either via decrease the quantity by 1 / set the quantity by 0)"

At the same time, an another customer also view the product page. Now the system can see both of the following cases:

- Return ONE unit. This happens when this another customer visit the page **BEFORE** the operation is executed.
- Return ZERO unit. This happens when this another customer visit tha page **AFTER** the operation is executed.

**The latter case is undesirable.** What if the transaction failed and a rollback operation is executed? The system should have displayed this unit of product to this another customer but instead it displayed 0 => lose potential customer => database admin/engineer's career on the line.

### 2. Non-repeatable Reads

Occur when a transaction executes multiple same `SELECT` query pointing to the same row yet the data of the row retured is different. Because in the meantime between two `SELECT` queries, another transaction has modified the data that matched the `SELECT` criteria..

E.g. the store owner is looking to read the number of units available for a given CD across two `SELECT` queries inside a single transaction, but a customer happens to buy a CD during that time.

### 3. Phantom Reads

Occur when a transaction executes multiple same `SELECT` query yet _different rows are being selected_. Another transaction has inserted/deleted rows before the first can complete.

NOTE: this is different from with [Non-repeatable reads](#2-non-repeatable-reads):

- Non-repeatable Reads: same query, same row (via id column) with different value (non-id columns).
- Phantom Reads: same query, different number of rows (new rows appear, old rows disappear)

## Locks

Locks can be enforced on a single row/table or a specific set of rows/tables.

### 1. Shared Lock

- Current transaction can both read and write the data.
- Other transactions **read the data without waiting, but must wait if they want to write.**

To apply this lock, use `SELECT ... FOR SHARE` syntax. `FOR SHARE` is exclusive to DQL `SELECT`.

### 2. Exclusive Lock

- Current transaction can both read and write the data.
- Another transaction **must wait if they want to either read or write it.**

This lock is used on the rows which are expected to be updated after being read and prevent an anoother transaction from reading old data.

To apply this lock, use `SELECT ... FOR UPDATE` syntax.

> `FOR UPDATE` syntax is exclusive to DQL `SELECT`.

> [!TIP]
>
> Your app will suffer from performance impact for data that's modifiable by multiple clients, but not so much for user-specific data.

When **reading and updating is made by ONE single transaction**, use exclusive Lock (a.k.a _row-level locking_):

```sql
-- same transaction
SELECT ... FOR UPDATE;
UPDATE ... SET ...;
```

When **reading and updating is made by TWO seperate transactions**, use _optimistic concurrency check_ pattern instead:

```sql
-- 1st transaction
SELECT version_number/etag/updated_at, ... FROM ...;

-- 2nd transaction
UPDATe ... SET ... WHERE version_number/etag/updated_at = ?
```

### 3. Gap Locking

InnoDB called it _next-key lock_, or technically, lock the existing records and lock **the gap** between the existing records.

Gap Locking can be used in conjunction with other locks ([Shared lock](#1-shared-lock), [Exclusive lock](#2-exclusive-lock))

This lock **prevents insertion** into the gap between existing indexed records. Since they're made when using `SELECT` with [Shared lock](#1-shared-lock) or [Exclusive lock](#2-exclusive-lock), it's easily to be mistaken that Gap Locking blocks updates/deletes of existing records within the gap, but it's actually the [Shared lock](#1-shared-lock) or [Exclusive lock](#2-exclusive-lock) who do that.

- Record lock: if `WHERE` clause specify equality lookup that return single row.
- Gap lock: if `WHERE` clause specify range lookup that returns multiple rows, and there are non-existance row whose criteria is matching the `WHERE` clause, thus creating the "gap".

It's also made when you `UPDATE` or `DELETE`. During the update or delete operations affecting a range of rows with multiple gap, you can't insert new row into the gap before the update or delete completed.

E.g. Suppose you have a table with indexed values: `1, 3, 5`.

```sql
START TRANSACTION
SELECT * FROM table WHERE id > 1 AND id < 5 FOR UPDATE;
-- ...
COMMIT
```

- MySQL places gap locks between 1 and 3, and between 3 and 5.
- New record with `id = 2, 4` must wait before the commit took place.
- `UPDATE` and `DELETE` can be done on `id = 1, 3, 5` but only exclusive for the current transaction. Other transactions must wait.

**NOTE:** since the scanning is made on indexed column, the gap is precisely located. But if the scanning is made on non-indexed column, that means a _full-table scan (FTS)_ is conducting and the lock will span across **all rows in the table**, results in **no row can be inserted** in the mean time.

Prevent [Phantom Reads](#3-phantom-reads) - rows from being inserted that may alter the query if the read query runs the second time.

## FOUR MySQL isolation levels

Each level instruct the database engine on what violations are allowed/denied during multiple transactions concurrent execution.

### `READ UNCOMMITED`

It's the lowest level of isolation, allowing all violations.

**A transaction can read the uncommited data from other uncommited transactions.**

Those uncommited transactions can rollback at a certain point of time later, but the current transaction has already read it, and every operations utilize this data are deemed incorrect.

This data is "transient" - accurate only at a single point in time. This approach is well-suited for use cases that permit transient data to trade strict accuracy for improved performance.

#### Pros

- Very fast, no need to wait for those uncommited transactions to commit since the read data is allowed to be transient.

- **Use case:** performant, non-critical UX data such as reading the approximation of the reaction count of a post on social media.

#### Cons

- Not appropriate for some use cases that assume the other committed transactions can fail, and read should fetch the pre-commit data.

E.g. a customer is in the middle of buying the last CD, the transaction has already decrease the quantity of that CD to 0, but it's still unsure that transaction will be commited since that customer can suffer failure from banking system, the e-commerce system itself, ... That last CD should be visible to other customers as well (of course they might not be able to buy it, but who knows, what if the previous customer suffer from an error or manual cancel the buy, you don't want to lose other customers right?)

### `READ COMMITED`

- You only see data that has already been committed from another transactions.
- You will never see changes made by other uncommited transactions. If there is an on-going uncommited transaction, you will read the previous version of the modified data.
- Of cource, your own uncommited changes will always be visible to you.
- Every time you run a SELECT, you get the latest commited data at that moment, be that it's from your uncommited transactions or another commited transaction.
- If someone else changes a row and commits, your next SELECT will show the new value.

#### Pros

- Avoid [**dirty read**](#1-dirty-read), always read the latest commited data (p/s: now late buyer/customer can still take their chance!)

#### Cons (also a problem to [`READ UNCOMMITED`](#read-uncommited))

**Lost Update problem due to _read-then-write_ pattern:** If two transactions requiring reading the same data into app, update it there, then write it back, they can overwrite each other's change (a.k.a **Race Condition**).

E.g.

Transaction A reads 5, plans to increase by 1 by setting it to 6.
Transaction B reads 5, plans to increase by 2 by setting it to 7.

```sql
-- first and second transaction
-- compute new `count` in app
UPDATE reactions SET count = ? WHERE id = ?
```

Now the final result can be either **6 or 7**, but the correct value is **8**.

=> Prevension methods:

1. Check if the current use case implementing `read-then-write` pattern can convert into **atomic SQL operation** since MySQL's InnoDB serializes the concurrent increments correctly by **acquiring an exclusive row-level lock on this row, two UPDATEs will not step on each other foot, one will wait for the other and then apply its increment.**

```sql
-- atomic increment

-- first transaction
UPDATE reactions SET count = count + 1 WHERE id = ?

-- second transaction
UPDATE reactions SET count = count + 2 WHERE id = ?
```

2. However, if `read-then-write` (`SELECT` then `UPDATE`) is a must with app-computed value then:

- Set up an [exclusive lock](#2-exclusive-lock):

```sql
START TRANSACTION
SELECT count FROM reactions WHERE id = ? FOR UPDATE; -- exclusive lock
-- compute new `count` in app
UPDATE reactions SET count = ? WHERE id = ?;
COMMIT;
```

- Optimistic checking using `WHERE` clause:

```sql
-- Optimistic: only update if stored value equals old_value; check affected rows
UPDATE reactions
SET count = ?
WHERE id = ? AND count = ?;
```

If the affected row is zero, it means the value has been modified in the meantime, your client application can let it fail, or implement auto retry with new value (_statement replay_).

**NOTE:** ORM patterns rely heavily on this `read-then-write` pattern, they are the usual source of lost updates.

3. Use [`SERIALIZABLE`](#serializable) isolation level.

### `REPEATABLE READ`

It's the **default MySQL isolation level**, unless configured otherwise.

MySQL's modern and latest storage engine - `InnoDB`, uses _Multi Version Concurrency Control (MVCC)_ to take _transaction-level snapshot_ of a row or multiple rows at the start of the transaction (now called _row versions_).

_Statement-level snapshot_: the snapshot behavior when user running `READ COMMITED`, where each subsequent statements creates a new snapshot.

_Transaction-level snapshot_ is different from _statement-level snapshot_: using MVCC, writers can create a new row version and keep the old version in undo logs, in case of rollback in the future. Other readers can also see the old version (remember, writers' transactions haven't been commited yet) and subsequent read will return this old version for the rest of the transaction. Readers can only work on the old version until the reader commit (remember, even if writer commit and reader read it again, the returned data is still old version)

Transaction-level snapshot applies to plain `SELECT`. `SELECT` with locks

**Pros:** Avoid all 3 violations.

**Cons:**

- Long-running connections will build up a lot of old versions (a.k.a _MVCC retention_), results in rapid increasing in transient data.
- To prevent the other transactions from modifying the rows you read (or even prevent the reading of those rows), you still have to use [Shared Lock](#1-shared-lock) and [Exclusive Lock](#2-exclusive-lock).

=> **At the end, all isolation level do is prevent the 3 violations.** There are much more problems.

### Serializable

**Pros:** Prevent ALL violations
**Cons:** has the most performance impact. Works exactly like `REPEATABLE READ` but create a [Shared Lock](#1-shared-lock) implicitly on ALL `SELECT` statements, regardless of whether `FOR SHARE` is specified or not.

=> Serializable: "While I'm reading, other can read as well, but other must wait if they want to write, until I'm commiting, even though I just read and do no modification."

## References

- [PlanetScale's Blog, "MySQL isolation levels and how they work", Brian Morrison II, 2024-01-08](https://planetscale.com/blog/mysql-isolation-levels-and-how-they-work)
