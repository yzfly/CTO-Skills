---
name: "Vela and PostgreSQL"
displayName: "Vela PostgreSQL Database Management"
version: "1.0.0"
author: "Simplyblock"
description: "Comprehensive guide to PostgreSQL database management, including schemas, tables, indexes, views, full-text search, vector similarity search, time-series data, and Vela serverless database integration with PostgREST API"
category: "database"
tags:
  - postgresql
  - database
  - sql
  - vela
  - serverless
  - postgrest
  - pgvector
  - full-text-search
  - time-series
  - relational-database
icon: "database"
homepage: "https://vela.simplyblock.io/"
repository: "https://github.com/simplyblock/vela-skill"
documentation: "SKILL.md"
license: "MIT"
topics:
  - schemas
  - tables
  - table_creation
  - table_alteration
  - foreign_keys
  - constraints
  - indexes
  - partial_indexes
  - expression_indexes
  - index_ordering
  - partitioned_tables
  - declarative_partitioning
  - time_series_data
  - hypertables
  - continuous_aggregates
  - views
  - regular_views
  - materialized_views
  - updatable_views
  - incremental_view_maintenance
  - pg_ivm
  - data_manipulation
  - insert
  - update
  - delete
  - returning
  - querying_data
  - select
  - joins
  - ctes
  - aggregations
  - pagination
  - cursor_pagination
  - offset_pagination
  - full_text_search
  - tsvector
  - tsquery
  - ranking
  - highlighting
  - weighted_search
  - vector_similarity_search
  - pgvector
  - embeddings
  - semantic_search
  - hybrid_search
  - hnsw_indexes
  - ivfflat_indexes
  - functions_and_operators
  - logical_operators
  - comparison_operators
  - string_functions
  - mathematical_functions
  - date_time_functions
  - type_conversion
  - casting
  - row_level_security
  - rls
  - policies
  - privileges
  - access_control
  - user_management
  - roles
  - grants
  - vela_integration
  - postgrest_api
  - serverless_postgres
  - extensions
  - postgis
  - timescaledb
  - pg_trgm
  - plv8
  - pgcrypto
  - uuid_ossp
  - http
  - best_practices
  - common_mistakes
  - performance_optimization
  - security_best_practices
keywords:
  - PostgreSQL
  - SQL
  - database
  - Vela
  - serverless
  - PostgREST
  - pgvector
  - vector search
  - full-text search
  - time-series
  - partitioning
  - indexing
  - views
  - security
dependencies: []
vela:
  extensions:
    - vector
    - pg_trgm
    - postgis
    - uuid-ossp
    - pgcrypto
    - http
    - timescaledb
    - plv8
    - pg_ivm
  features:
    - postgrest
    - direct-sql
    - database-branching
examples:
  - schemas
  - table_creation
  - indexing
  - full_text_search
  - vector_search
  - pagination
  - partitioning
  - views
  - row_level_security
relatedSkills: []
---

# PostgreSQL Skill Guide

Welcome to the PostgreSQL skill guide for Agents and OpenClaw! This comprehensive guide covers everything you need to know about managing PostgreSQL databases, from basic table operations to advanced features like partitioning, row-level security, and integration with Vela serverless database.

**⚡ Quick Start**: See [USAGE-GUIDE.md](USAGE-GUIDE.md) for how to activate and use this skill effectively, including safety rules and best practices.

## Table of Contents

1. [Schemas](sections/01-schemas.md)
2. [Tables](sections/02-tables.md)
3. [Foreign Keys](sections/03-foreign-keys.md)
4. [Indexes](sections/04-indexes.md)
5. [Partitioned Tables](sections/05-partitioned-tables.md)
6. [Time-Series Data](sections/06-time-series.md)
7. [Views](sections/07-views.md)
8. [Data Manipulation](sections/08-data-manipulation.md)
9. [Querying Data](sections/09-querying-data.md)
10. [Full-Text Search](sections/10-full-text-search.md)
11. [Vector Similarity Search](sections/11-vector-search.md)
12. [Functions and Operators](sections/12-functions-operators.md)
13. [Type Conversion](sections/13-type-conversion.md)
14. [Row-Level Security](sections/14-row-level-security.md)
15. [Privileges and Access Control](sections/15-privileges.md)
16. [Vela Integration](sections/16-vela-integration.md)
17. [Common Mistakes and Best Practices](sections/17-best-practices.md)
18. [Pagination](sections/18-pagination.md)
19. [PostgreSQL Extensions](sections/19-extensions.md)

---

## Quick Reference

### Common Operations

```sql
-- Create table with identity column
CREATE TABLE users (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Insert data
INSERT INTO users (email) VALUES ('john@example.com') RETURNING id;

-- Query data
SELECT * FROM users WHERE email = 'john@example.com';

-- Update data
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Delete data
DELETE FROM users WHERE id = 1;
```

### Vela Quick Start

**Recommended: Direct PostgreSQL Connection**
```
postgresql://postgres:[PASSWORD]@db.{branchid}.demo.vela.run:{port}/postgres
```

⚠️ **Important:** Do not use TLS/SSL parameters (e.g., `?sslmode=require`). Vela does not support transport layer security for PostgreSQL connections.

**Alternative: REST API** (use PostgreSQL connections when possible for better performance)
```
https://{branchid}.demo.vela.run/rest
```

**Example PostgreSQL Query:**
```bash
psql "postgresql://postgres:[PASSWORD]@db.{branchid}.demo.vela.run:{port}/postgres" -c "SELECT * FROM users LIMIT 10;"
```

**Example REST Query:**
```bash
curl "https://{branchid}.demo.vela.run/rest/users?select=id,email&limit=10"
```

---

## Documentation Structure

Each section in the `sections/` directory covers a specific topic in detail:

- **01-schemas.md** - Schema creation and management
- **02-tables.md** - Table creation, alteration, and data types
- **03-foreign-keys.md** - Relationships and referential integrity
- **04-indexes.md** - Index types and optimization
- **05-partitioned-tables.md** - Table partitioning strategies
- **06-time-series.md** - Time-series data patterns
- **07-views.md** - Views and materialized views
- **08-data-manipulation.md** - INSERT, UPDATE, DELETE operations
- **09-querying-data.md** - SELECT queries, JOINs, CTEs
- **10-full-text-search.md** - Full-text search with tsvector
- **11-vector-search.md** - Semantic search with pgvector
- **12-functions-operators.md** - Built-in functions and operators
- **13-type-conversion.md** - Type casting
- **14-row-level-security.md** - Row-level access control
- **15-privileges.md** - User management and permissions
- **16-vela-integration.md** - Vela serverless PostgreSQL
- **17-best-practices.md** - Common mistakes to avoid
- **18-pagination.md** - Cursor vs offset pagination strategies
- **19-extensions.md** - Available PostgreSQL extensions on Vela

---

## Key Features

### PostgreSQL Core

- ✅ **ACID Compliance** - Full transactional support
- ✅ **Rich Data Types** - JSON, Arrays, UUIDs, Ranges, and more
- ✅ **Advanced Indexing** - B-tree, GiST, GIN, BRIN, HNSW
- ✅ **Full-Text Search** - Built-in text search capabilities
- ✅ **Partitioning** - Range, list, and hash partitioning
- ✅ **Row-Level Security** - Fine-grained access control
- ✅ **Extensibility** - pgvector, PostGIS, and more

### Vela-Specific Features

- ✅ **Serverless** - Automatic scaling, no infrastructure management
- ✅ **PostgREST API** - Automatic REST API for your database
- ✅ **pgvector Support** - Vector similarity search built-in
- ✅ **Time-Series Optimized** - Efficient partitioning and retention
- ✅ **Full-Text Search** - PostgreSQL FTS via REST API
- ✅ **Branching** - Database branches for development/testing

---

## Best Practices Summary

### Schema Design
- ✅ Use `GENERATED AS IDENTITY` instead of `SERIAL`
- ✅ Use `TIMESTAMP WITH TIME ZONE` for all timestamps
- ✅ Use `NUMERIC` for money, not `REAL` or `MONEY` type
- ✅ Use `TEXT` or `VARCHAR`, avoid `CHAR(n)`
- ✅ Name all constraints explicitly

### Querying
- ✅ Specify columns instead of `SELECT *`
- ✅ Use `EXISTS` instead of `COUNT(*) > 0`
- ✅ Use cursor/keyset pagination instead of `OFFSET`
- ✅ Use `>= and <` instead of `BETWEEN` for timestamps
- ✅ Always use parameterized queries

### Indexing
- ✅ Index foreign key columns
- ✅ Index columns used in WHERE, JOIN, ORDER BY
- ✅ Use partial indexes for filtered queries
- ✅ Create indexes AFTER bulk data loading
- ❌ Don't over-index (slows writes)

### Security
- ✅ Use Row-Level Security for multi-tenant apps
- ✅ Follow principle of least privilege
- ✅ Never store passwords in plain text
- ✅ Always use parameterized queries (prevent SQL injection)
- ❌ Never use superuser for applications

### Performance
- ✅ Use connection pooling
- ✅ Use `EXPLAIN ANALYZE` regularly
- ✅ Keep transactions short
- ✅ Vacuum and analyze regularly
- ❌ Don't disable autovacuum

---

## Additional Resources

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Don't Do This](https://wiki.postgresql.org/wiki/Don%27t_Do_This)
- [pgvector Extension](https://github.com/pgvector/pgvector)
- [PostgREST Documentation](https://docs.postgrest.org/en/v14/)
- [Vela Documentation](https://docs.vela.run)

---

**Skill Version**: 1.0.0  
**Last Updated**: February 2026  
**License**: MIT
