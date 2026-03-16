---
description: SQL coding standards for dbt, PostgreSQL, Redshift, and Databricks SQL.
---

# SQL Rules

## Always

- Explicit column lists in `SELECT` (never `SELECT *` in production)
- Use `snake_case` for table and column names
- Alias all tables in joins (`JOIN orders o ON ...`)
- Use parameterized queries in application code (never string interpolation)
- Add `created_at` and `updated_at` columns to all tables
- Use `NOT NULL` constraints on required columns

## dbt Specific

- Use `{{ ref() }}` for model references, `{{ source() }}` for raw tables
- Never hardcode database or schema names
- Every primary key column must have `not_null` + `unique` tests
- Staging models: rename and cast only — no joins, no business logic
- Incremental models must define `unique_key` and `on_schema_change`

## Redshift Specific

- Define `DISTKEY` and `SORTKEY` on all fact tables
- Use `DISTSTYLE ALL` for small dimension tables
- Run `VACUUM` + `ANALYZE` after large data loads

## Databricks SQL Specific

- Prefer `MERGE INTO` for upserts over `INSERT OVERWRITE`
- Use liquid clustering (Databricks 13.3+) over static `PARTITIONBY` for high-cardinality columns
- Schedule `OPTIMIZE` after large `MERGE` operations
- Enable `delta.autoOptimize.autoCompact` on frequently written tables
