# SQL Query Optimization

Resources:
* [SCaLE - Query Optimization 101 in MySQL](https://www.youtube.com/watch?v=3pu7hoR1HbU)

## Indexes

Always use indexes. They allow faster look up of columns. The lookups can be done with a b-tree rather than a full scan leading to O(logn) instead of O(n).

## Primary Keys

A way to uniquely identify a row. It must be unique, and there can only be one primary key per table. It can consist of multiple columns.

All primary keys are indexes.

## EXPLAIN

`EXPLAIN` can be used to describe the query optimization plan.

[Ref](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

A few key things to look out for:

`type`

The type is important for determining the access. Some common ones are `ref`, specifying index lookups. `all` specifying full scan (normally bad). `range` specifying range query

### EXPLAIN FORMAT=TREE

Shows better sequence of query.

### EXPLAIN ANALYZE

Executes query and reports back statistics.


