---
name: dbt-patterns
description: dbt project patterns for model layering, testing strategy, incremental builds, macros, and dbt Cloud CI/CD.
origin: dham-claude-code
---

# dbt Patterns

Use this skill when building or reviewing dbt projects targeting Databricks, Redshift, Snowflake, or BigQuery.

## Project Layout

```
models/
├── staging/              # 1:1 source, light cleaning, stg_ prefix
│   ├── salesforce/
│   │   ├── _salesforce__sources.yml
│   │   ├── _salesforce__models.yml
│   │   └── stg_salesforce__accounts.sql
│   └── stripe/
├── intermediate/         # int_ prefix: joins, pivots, business logic
│   └── int_customer_orders.sql
└── marts/                # Business-facing, organized by domain
    ├── sales/
    │   ├── fct_orders.sql
    │   └── dim_customers.sql
    └── finance/
        └── fct_revenue.sql
```

## Source & Model YML

```yaml
# staging/salesforce/_salesforce__sources.yml
sources:
  - name: salesforce
    database: raw_catalog
    schema: salesforce
    loaded_at_field: _fivetran_synced
    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: account
        identifier: Account

# staging/salesforce/_salesforce__models.yml
models:
  - name: stg_salesforce__accounts
    description: "Cleaned Salesforce accounts"
    columns:
      - name: account_id
        tests: [not_null, unique]
      - name: account_type
        tests:
          - accepted_values:
              values: ['Customer', 'Partner', 'Prospect']
```

## Incremental Model

```sql
-- models/staging/stripe/stg_stripe__charges.sql
{{ config(
    materialized='incremental',
    unique_key='charge_id',
    incremental_strategy='merge',
    on_schema_change='sync_all_columns',
    cluster_by=['created_date']   -- Databricks
) }}

SELECT
    charge_id,
    customer_id,
    amount / 100.0          AS amount_dollars,
    currency,
    created_at::date        AS created_date,
    status
FROM {{ source('stripe', 'charges') }}

{% if is_incremental() %}
WHERE created_at > (SELECT MAX(created_at) FROM {{ this }})
{% endif %}
```

## Useful Macros

```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name) %}
    ({{ column_name }} / 100.0)
{% endmacro %}

-- macros/safe_divide.sql
{% macro safe_divide(numerator, denominator) %}
    CASE WHEN {{ denominator }} = 0 THEN NULL
         ELSE {{ numerator }}::float / {{ denominator }}
    END
{% endmacro %}
```

## dbt_project.yml Config

```yaml
name: 'data_platform'
version: '1.0.0'
config-version: 2

models:
  data_platform:
    staging:
      +materialized: view
      +schema: staging
      +tags: ['staging']
    intermediate:
      +materialized: ephemeral
    marts:
      +materialized: table
      sales:
        +schema: marts_sales
        +tags: ['daily', 'marts']

seeds:
  data_platform:
    +schema: seeds

vars:
  start_date: '2024-01-01'
```

## Slim CI (dbt Cloud / GitHub Actions)

```bash
# Only run models changed in the PR
dbt build --select state:modified+ --defer --state ./prod-artifacts

# Generate prod artifacts for comparison
dbt ls --output json > ./prod-artifacts/manifest.json
```

## Key Rules

- Staging models: rename, cast, and clean only — no joins
- Intermediate models: joins and business logic — not user-facing
- Mart models: wide, denormalized, consumer-ready
- Every mart model needs a primary key test (`not_null` + `unique`)
- Use `{{ ref() }}` always — never hardcode schema names
