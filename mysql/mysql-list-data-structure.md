# MySQL List Data Structure

There are many ways to add list data structure to the column of MySQL table:

1. Normalized 1NF table
2. `ENUM` type
3. `JSON` type

Each of them has their own pros and cons.

## Normalized 1NF table (lookup/referenced table)

Pros:

- SSOT.
- Standardized. It's the de-facto when storing list data structure in database.

Cons:

- Verbose. `JOIN` is required when fetching data from these normalized tables.

- Sometimes it's much more efficient to use `N + 1` strategy when joining a very large number of rows in a parent (referencing) table with a very small number of rows in a normalized table.

## ENUM

Pros:

- Simple.
- Storage optimized (MySQL store a set of incrementing numbers and map them to the corresponding string value)

Cons:

- Violate Single Source of Truth (SSOT). If more than one table contains columns sharing the same set of list elements, those columns must be kept synchronized (this can be mitigated with triggers but still adds complexity and maintainance).

## JSON

Pros:

- Robust and flexible. Support object/dictionary-like data structure, such as array.

Cons:

- Introduced in MySQL v5.7 but it's not feature perfect until v8.0.17, to this day there are still new fixes regarding this type.

- Break the standards that made up the foundation of Relational Database.

- Uses 2-3x more space as storing the same data in normal rows and columns.

- Using elements of JSON arrays/objects in SQL query can lead to unoptimized execution plan and add to performance overhead

- Multi-valued index is required to optimized `SELECT` query regarding searching JSON arrays for a particular element.

## References

- [Bill Karwin's answer on SOF](https://stackoverflow.com/a/75675676/9122512)
