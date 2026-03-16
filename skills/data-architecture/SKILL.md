---
name: data-architecture
description: Data platform architecture patterns including medallion architecture, data mesh, lakehouse design, data contracts, and governance frameworks.
origin: dham-claude-code
---

# Data Architecture Patterns

Use this skill when designing data platforms, evaluating architecture patterns, or planning data mesh implementations.

## Medallion Architecture

```
Bronze (Raw)
  └── Exact copy of source data
  └── Append-only or CDC
  └── Partitioned by ingestion date
  └── No business transformations

Silver (Cleansed)
  └── Validated, deduplicated, typed
  └── Conformed schemas
  └── SCD Type 1/2 for slowly changing dims
  └── Partitioned by business date

Gold (Business)
  └── Aggregated for specific use cases
  └── Star/wide schema for BI consumption
  └── Strict SLAs (freshness, quality)
  └── One model per consumer use case
```

## Data Mesh Domains

```
┌──────────────────────────────────────────────────────┐
│                   Platform Team                       │
│  (self-serve infra, catalog, governance policies)    │
└────────────────────┬─────────────────────────────────┘
                     │ enables
     ┌───────────────┼────────────────┐
     ▼               ▼                ▼
 Sales Domain    Finance Domain   Marketing Domain
 (owns: orders,  (owns: revenue,  (owns: campaigns,
  customers)      invoices)        attribution)
```

### Data Product Standards
Each domain's data product must be:
1. **Discoverable** — registered in Unity Catalog / Glue Catalog
2. **Addressable** — stable URI / table name
3. **Trustworthy** — SLA defined, quality tests passing
4. **Self-describing** — schema + column descriptions + owner
5. **Interoperable** — standard formats (Delta, Parquet, Avro)

## Data Contract Template

```yaml
# data-contracts/orders/v1.yaml
name: orders
version: "1.0.0"
owner: data-platform-team@company.com
domain: sales
sla:
  freshness: 1 hour
  availability: 99.5%
schema:
  - name: order_id
    type: bigint
    nullable: false
    description: "Unique order identifier"
  - name: customer_id
    type: bigint
    nullable: false
  - name: status
    type: string
    nullable: false
    allowed_values: [pending, shipped, delivered, cancelled]
  - name: created_at
    type: timestamp
    nullable: false
quality:
  - rule: "order_id IS NOT NULL AND order_id > 0"
    severity: error
  - rule: "created_at <= CURRENT_TIMESTAMP()"
    severity: warning
```

## Technology Selection Matrix

| Criteria | Databricks | AWS Glue | dbt | Airflow |
|----------|-----------|----------|-----|---------|
| Best for | Unified analytics + ML | Serverless ETL | SQL transforms | Orchestration |
| Storage | Delta Lake native | S3 + Glue Catalog | Warehouse native | N/A |
| Scale | Large (PB) | Medium | Medium-Large | N/A |
| Cost model | DBU-based | Per DPU | Per run | Per task |
| Use when | Spark + ML needed | AWS-native, serverless | SQL-first teams | Multi-system pipelines |

## Lakehouse vs Warehouse vs Lake

| | Data Lake | Data Warehouse | Lakehouse |
|--|----------|---------------|-----------|
| Format | Raw files | Proprietary | Open (Delta, Iceberg) |
| ACID | No | Yes | Yes |
| Schema | Schema-on-read | Schema-on-write | Both |
| ML support | Native | Limited | Native |
| Cost | Low storage | High | Medium |
| Best for | Raw storage | BI/reporting | Unified platform |

## Governance Checklist

- [ ] Data catalog with column-level descriptions
- [ ] PII tagged at column level
- [ ] Row/column-level security configured
- [ ] Data lineage tracked (column-level preferred)
- [ ] Retention policies defined per domain
- [ ] Data quality rules enforced in pipeline (not just dbt tests)
- [ ] Data contracts published and versioned
