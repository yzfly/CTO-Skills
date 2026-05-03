# PostgreSQL Anti-Patterns — "Don't Do This"

> Source: https://wiki.postgresql.org/wiki/Don%27t_Do_This

## Database Encoding

- **Don't use SQL_ASCII.** Treats bytes without conversion, resulting in mixed encodings with no recovery. Use UTF-8.

## SQL Constructs

- **Don't use NOT IN.** Returns zero rows if NULL exists in list. Use `NOT EXISTS (SELECT...)` instead.
- **Don't use BETWEEN for timestamps.** Includes both endpoints; midnight timestamps get double-counted. Use `>= start AND < end`.
- **Don't use uppercase table/column names.** Requires double-quoting. Use lowercase with underscores.

## Date/Time

- **Don't use timestamp without time zone.** Stores picture of clock, not point in time. Use `timestamptz`.
- **Don't store UTC in timestamp without time zone.** Database can't know UTC is intended. Use `timestamptz`.
- **Don't use timetz.** Only exists for SQL compliance. Use `timestamptz`.
- **Don't use CURRENT_TIME.** Returns `timetz`. Use `CURRENT_TIMESTAMP` or `LOCALTIMESTAMP`.
- **Don't use timestamp(0) or timestamptz(0).** Rounds instead of truncating. Use `date_trunc('second', value)`.

## Text Storage

- **Don't use char(n).** Pads with spaces, wastes storage, breaks comparisons. Use `text`.
- **Don't use varchar(n) by default.** Arbitrary limits risk production errors. Use `text` with `CHECK` constraints if needed.

## Data Types

- **Don't use money.** Fixed-point, locale-dependent, wrong rounding. Use `numeric` with separate currency column.
- **Don't use serial.** Weird dependency behavior. Use identity columns (PostgreSQL 10+).

## Authentication

- **Don't use trust over TCP/IP.** Anyone claiming any username is accepted. Use `scram-sha-256`.

## Other

- **Don't use rules.** They rewrite queries unexpectedly. Use triggers.
- **Don't use table inheritance.** Use foreign keys or native partitioning.
