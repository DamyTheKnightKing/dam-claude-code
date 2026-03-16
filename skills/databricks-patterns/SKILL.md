---
name: databricks-patterns
description: Databricks patterns for Delta Lake, Unity Catalog, Delta Live Tables, Databricks Asset Bundles, and PySpark optimization.
origin: dham-claude-code
---

# Databricks Patterns

Use this skill for building data pipelines, Delta Lake tables, and analytics workloads on Databricks.

## Workspace Organization

```
databricks/
├── src/
│   ├── ingestion/        # Bronze layer jobs
│   ├── transformation/   # Silver layer jobs
│   ├── aggregation/      # Gold layer jobs
│   └── dlt/             # Delta Live Tables pipelines
├── tests/
├── notebooks/           # Exploratory only, not production
└── databricks.yml       # Databricks Asset Bundle config
```

## Databricks Asset Bundle (DAB)

```yaml
# databricks.yml
bundle:
  name: data-platform

targets:
  dev:
    mode: development
    workspace:
      host: https://your-workspace.azuredatabricks.net
    variables:
      catalog: dev_catalog
      schema: data_platform

  prod:
    mode: production
    workspace:
      host: https://your-workspace.azuredatabricks.net
    variables:
      catalog: prod_catalog
      schema: data_platform

resources:
  jobs:
    daily_etl:
      name: "Daily ETL Pipeline"
      schedule:
        quartz_cron_expression: "0 0 6 * * ?"
        timezone_id: "UTC"
      tasks:
        - task_key: bronze_ingestion
          python_wheel_task:
            package_name: data_platform
            entry_point: run_bronze
```

## Delta Lake Table Design

```python
# Liquid clustering (preferred over static partitioning for Databricks 13.3+)
spark.sql("""
CREATE TABLE IF NOT EXISTS catalog.schema.orders (
    order_id    BIGINT NOT NULL,
    customer_id BIGINT NOT NULL,
    event_date  DATE NOT NULL,
    status      STRING NOT NULL,
    amount      DECIMAL(18,2),
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
)
CLUSTER BY (customer_id, event_date)
TBLPROPERTIES (
    'delta.enableChangeDataFeed' = 'true',
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true'
)
""")
```

## Delta Live Tables Pipeline

```python
import dlt
from pyspark.sql.functions import col, current_timestamp

@dlt.table(
    comment="Bronze: raw events from cloud storage",
    table_properties={"quality": "bronze"}
)
def bronze_orders():
    return (spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .option("cloudFiles.schemaLocation", "/checkpoints/bronze_orders_schema")
        .load("s3://bucket/raw/orders/"))

@dlt.table(comment="Silver: validated orders")
@dlt.expect_or_drop("valid_order_id", "order_id IS NOT NULL")
@dlt.expect_or_drop("valid_amount", "amount > 0")
def silver_orders():
    return (dlt.read_stream("bronze_orders")
        .select(
            col("order_id").cast("bigint"),
            col("customer_id").cast("bigint"),
            col("amount").cast("decimal(18,2)"),
            current_timestamp().alias("processed_at")
        ))
```

## PySpark Performance

```python
# Use Spark SQL for complex aggregations (Photon-accelerated)
result = spark.sql("""
    SELECT customer_id, DATE_TRUNC('month', event_date) AS month,
           SUM(amount) AS total_spend
    FROM catalog.schema.orders
    WHERE event_date >= '2025-01-01'
    GROUP BY 1, 2
""")

# Broadcast small dimension tables
from pyspark.sql.functions import broadcast
enriched = orders.join(broadcast(customers), "customer_id")

# Repartition before writing (not after reading)
df.repartition(200, "customer_id").write.format("delta").save(...)
```

## Unity Catalog Governance

```sql
-- Data product registration
CREATE EXTERNAL LOCATION raw_data
  URL 's3://your-bucket/raw'
  WITH (STORAGE CREDENTIAL your_credential);

-- Column-level masking for PII
CREATE OR REPLACE FUNCTION mask_email(email STRING)
RETURNS STRING
RETURN CASE WHEN is_account_group_member('pii-readers') THEN email
            ELSE CONCAT(LEFT(email, 2), '***@***.com') END;

ALTER TABLE customers ALTER COLUMN email
SET MASK mask_email;
```
