# MySQL Primary Key Best Practices

## Terminology

- **Clustered Index (or _primary index_):** the index determines the physical order (specifically, actual bits stored on the disk) in which data rows are stored. The data can only be stored in ONE order (duh) => a table can only have one clustered index, usually `PRIMARY KEY`.
- **Non-clustered Index (or _secondary index_):** the index that has pointers to the physical rows. Unique index and compound index are classified as non-clustered index.

**CAUTION:**

- If a table has a non-sequential clustered index (e.g. `UUID`), after each write operation, data rearrangment occurs.
- More indexes improve read performance but also increase the amount of data written to disk, resulting in additional time and storage overhead for write operations.

---

- **Page splitting:** rows in DBMS are stored inside fixed-side block called a _page_. When this page gets full, and you try to add a non-sequential data (like writting a sentence in the middle of an essay that has already passed the 300-words limit), the engine splits the data into 2, move the latter into a new page, and write the new data at the end of the existing page.

**CAUTION:** This frequently occurs when inserting non-sequential data such as `UUID`, resulting in numerous partially filled pages and significant space inefficiency. 5**0% page filling for sequential data compares to 94% for sequential data.**

## Auto-incrementing Integer

Choices:

- `INT`: 32 bits/value, ~ 2 billion rows/table.
- `BIGINT`: 64 bits/value, ~ 9.2 quintillion rows/table.

Pros:

- Simple, built-in. No need for stored procedures, functions, application code's 3rd-party libraries.
- The clustered index has a small space overhead, and secondary indexes as well.

Cons:

- Less uniquity.
- Reveal the count of records inside tables.
- Collision can happen when merging rows among distributed systems.

## UUID

Choices:

- `BINARY(16)`: 128 bits/value. Machine-readable only.
- `CHAR(36)`: 288 bits/value. Human-readable.

Pros:

- Large unique space, very low chance of collision.
- Time-based, ordered variant such as `UUIDv6` and `UUIDv7` can both guarantee uniqueness and keep values as close to sequential as possible. Avoid use `UUIDv1` since `time_low`, the least significant portion of the timestamp is put first.

  ```
  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  |        |    |    |    |
  |        |    |    |    |-- node (MAC address or random)
  |        |    |    |-- clock_seq (sequence number)
  |        |    |-- time_hi_and_version
  |        |-- time_mid
  |-- time_low
  ```

Cons:

- Long. Take a lot of disk space.
- Bad UX in browser since the browsers interpret a UUID as 5 different words, therefore engineers can't select them by double-click.
- `BINARY(16)` requires using built-in functions + swap flag to reorder the timestamp portion more sequential: `UUID_TO_BIN(<uuid>, 1)` and `BIN_TO_UUID(<bin>)`. Error-prone if user forget to add the transformation when writing queries.
- If mishandled, write operations can suffer from performance overhead.

Better choices: `nanoid`, `ULID`, ...

## NanoID with `BIGINT`

Pros:

- Shorter than UUID
- Easy to select with double clicking.
- Low chance of collision (measurable with [Zelark's NanoID collision tool](https://zelark.github.io/nano-id-cc/))
- Easy to be generated in multiple programming languages.

## References

- [Mike Coutermarsh, PlanetScale's Blog, "Why we chose NanoIDs for PlanetScale’s API"](https://planetscale.com/blog/why-we-chose-nanoids-for-planetscales-api)
- [Brian Morrison, PlanetScale's Blog, "The Problem with Using a UUID Primary Key in MySQL", 2024-03-19](https://planetscale.com/blog/the-problem-with-using-a-uuid-primary-key-in-mysql#use-the-binary-data-type)

<!-- TODO: read more about PlanetScale's Engineering Blog -->
