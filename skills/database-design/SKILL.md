---
name: database-design
description: Database design patterns for PostgreSQL, Redshift, and Databricks SQL including schema design, indexing, migrations, and query optimization.
origin: dham-claude-code
---

# Database Design Patterns

Use this skill for designing schemas, writing migrations, optimizing queries, and modeling data for relational and analytical databases.

## Naming Conventions

```sql
-- Tables: snake_case plural nouns
CREATE TABLE orders ( ... );
CREATE TABLE order_items ( ... );

-- Columns: snake_case, descriptive
order_id, customer_id, created_at, updated_at, is_active

-- Indexes: idx_{table}_{column(s)}
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Foreign keys: fk_{table}_{referenced_table}
CONSTRAINT fk_order_items_orders FOREIGN KEY (order_id) REFERENCES orders(id)
```

## Standard Table Template

```sql
CREATE TABLE orders (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    external_id UUID NOT NULL DEFAULT gen_random_uuid(),
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    status      TEXT NOT NULL CHECK (status IN ('pending','shipped','delivered','cancelled')),
    total_cents INTEGER NOT NULL CHECK (total_cents >= 0),
    metadata    JSONB NOT NULL DEFAULT '{}',
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Audit trigger
CREATE TRIGGER orders_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

## PostgreSQL Index Strategy

```sql
-- B-tree (default): equality and range queries
CREATE INDEX CONCURRENTLY idx_orders_status_created
    ON orders (status, created_at DESC)
    WHERE status IN ('pending', 'shipped');  -- partial index

-- GIN: JSONB and full-text search
CREATE INDEX idx_orders_metadata ON orders USING gin(metadata);

-- BRIN: time-series, append-only tables
CREATE INDEX idx_events_occurred_at ON events USING brin(occurred_at);

-- Verify with EXPLAIN
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE customer_id = 123 AND status = 'pending';
```

## Migration Patterns

```sql
-- Safe column addition (non-blocking)
ALTER TABLE orders ADD COLUMN discount_cents INTEGER;
UPDATE orders SET discount_cents = 0 WHERE discount_cents IS NULL;
ALTER TABLE orders ALTER COLUMN discount_cents SET NOT NULL;
ALTER TABLE orders ALTER COLUMN discount_cents SET DEFAULT 0;

-- Non-blocking index creation
CREATE INDEX CONCURRENTLY idx_orders_discount ON orders(discount_cents)
    WHERE discount_cents > 0;

-- Rename with backwards compatibility (two-deploy pattern)
-- Deploy 1: add new column, backfill, dual-write
ALTER TABLE orders ADD COLUMN total_amount_cents INTEGER;
-- Deploy 2: remove old column after all readers updated
ALTER TABLE orders DROP COLUMN total_cents;
```

## Redshift Design

```sql
-- Fact table: KEY distribution on most common join key
CREATE TABLE fact_orders (
    order_id    BIGINT NOT NULL,
    customer_id BIGINT NOT NULL,
    order_date  DATE NOT NULL,
    amount      DECIMAL(18,2),
    PRIMARY KEY (order_id)
) DISTKEY(customer_id)
  COMPOUND SORTKEY(order_date, customer_id);

-- Dimension table: ALL distribution (replicated to all nodes)
CREATE TABLE dim_customers (
    customer_id BIGINT NOT NULL,
    full_name   VARCHAR(255),
    PRIMARY KEY (customer_id)
) DISTSTYLE ALL;
```

## Query Optimization Checklist

- [ ] `EXPLAIN ANALYZE` before and after adding indexes
- [ ] Avoid `SELECT *` in application queries
- [ ] Use parameterized queries (never string interpolation)
- [ ] Add `LIMIT` on exploratory queries
- [ ] Index all foreign key columns
- [ ] Use `CONCURRENTLY` for index creation in production
- [ ] Partition large tables by date for time-range queries
- [ ] `VACUUM` + `ANALYZE` after large deletes/updates (PostgreSQL)

## Anti-Patterns

- Storing comma-separated values in a single column (use junction table)
- Using `VARCHAR(MAX)` / `TEXT` everywhere (use appropriate lengths)
- Missing NOT NULL constraints relying on application validation
- Auto-increment starting at 1 for public-facing IDs (use UUID or opaque ID)
- Dropping and recreating tables for schema changes (use ALTER)
