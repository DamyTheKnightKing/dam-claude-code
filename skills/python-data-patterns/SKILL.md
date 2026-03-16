---
name: python-data-patterns
description: Python patterns for data engineering, ETL scripts, Pydantic models, async processing, and pandas/PySpark data manipulation.
origin: dham-claude-code
---

# Python Data Patterns

Use this skill for writing Python code for data pipelines, ETL, data validation, and processing workloads.

## Project Layout (Data Engineering)

```
src/
├── config/
│   └── settings.py          # Pydantic Settings from env
├── models/
│   └── schemas.py           # Pydantic data models
├── connectors/
│   ├── airflow_client.py    # External API clients
│   └── databricks_client.py
├── transformations/
│   └── orders.py            # Pure transform functions
├── pipeline/
│   └── run.py               # Entry point
└── utils/
    └── logging.py
```

## Pydantic for Configuration & Validation

```python
from pydantic import BaseModel, Field, field_validator
from pydantic_settings import BaseSettings
from typing import Optional
from enum import Enum

class Environment(str, Enum):
    DEV = "dev"
    STAGING = "staging"
    PROD = "prod"

class Settings(BaseSettings):
    environment: Environment = Environment.DEV
    databricks_host: str
    databricks_token: str
    s3_bucket: str
    slack_webhook_url: Optional[str] = None
    max_retries: int = Field(default=3, ge=1, le=10)

    model_config = {"env_file": ".env", "case_sensitive": False}

settings = Settings()

# Data model with validation
class OrderRecord(BaseModel):
    order_id: int = Field(gt=0)
    customer_id: int = Field(gt=0)
    amount_cents: int = Field(ge=0)
    status: str

    @field_validator("status")
    @classmethod
    def validate_status(cls, v: str) -> str:
        allowed = {"pending", "shipped", "delivered", "cancelled"}
        if v not in allowed:
            raise ValueError(f"status must be one of {allowed}")
        return v

    @property
    def amount_dollars(self) -> float:
        return self.amount_cents / 100
```

## Async HTTP Client Pattern

```python
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential

class AirflowAPIClient:
    def __init__(self, base_url: str, username: str, password: str):
        self.client = httpx.AsyncClient(
            base_url=base_url,
            auth=(username, password),
            timeout=30.0
        )

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
    async def get_dag_runs(self, dag_id: str, limit: int = 25) -> list[dict]:
        response = await self.client.get(
            f"/dags/{dag_id}/dagRuns",
            params={"limit": limit, "order_by": "-start_date"}
        )
        response.raise_for_status()
        return response.json()["dag_runs"]

    async def __aenter__(self): return self
    async def __aexit__(self, *args): await self.client.aclose()
```

## pandas Patterns

```python
import pandas as pd
from typing import Callable

# Pipeline pattern: chain transformations
def process_orders(df: pd.DataFrame) -> pd.DataFrame:
    return (
        df
        .pipe(rename_columns)
        .pipe(cast_types)
        .pipe(filter_valid)
        .pipe(add_derived_columns)
    )

def rename_columns(df: pd.DataFrame) -> pd.DataFrame:
    return df.rename(columns={"orderId": "order_id", "customerId": "customer_id"})

def cast_types(df: pd.DataFrame) -> pd.DataFrame:
    return df.assign(
        created_at=pd.to_datetime(df["created_at"]),
        amount=df["amount_cents"].div(100).astype(float)
    )

def filter_valid(df: pd.DataFrame) -> pd.DataFrame:
    return df.loc[df["order_id"].notna() & (df["amount"] >= 0)]
```

## Structured Logging

```python
import structlog
import logging

logging.basicConfig(format="%(message)s", level=logging.INFO)
structlog.configure(
    processors=[
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ]
)

log = structlog.get_logger()

# Usage
log.info("pipeline_started", dag_id="orders_daily", record_count=5000)
log.error("extraction_failed", dag_id="orders_daily", error=str(e), retry_count=attempt)
```

## Key Rules

- Use `pydantic-settings` for all config — never raw `os.environ`
- Use type hints on all function signatures
- Use `httpx.AsyncClient` for async HTTP; `tenacity` for retries
- Never `pd.DataFrame.append()` in loops — use `pd.concat([list_of_dfs])`
- Use `structlog` with JSON output for production logging
- Use `pathlib.Path` instead of string path concatenation
