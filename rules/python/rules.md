---
description: Python coding standards for data engineering and pipeline work.
---

# Python Rules

## Always

- Use type hints on all function signatures
- Use `pydantic-settings` `BaseSettings` for configuration (never raw `os.environ`)
- Use `pathlib.Path` for file paths
- Use `structlog` with JSON output for logging in production code
- Use `httpx.AsyncClient` for async HTTP requests
- Use `tenacity` for retry logic on external calls
- Parameterize all SQL queries (never f-string interpolation)

## Never

- `from module import *`
- Mutable default arguments (`def fn(items=[])` — use `None` and set inside)
- `print()` for logging in pipeline code (use `structlog`)
- String formatting for SQL (`f"SELECT * WHERE id = {user_id}"`)
- `pd.DataFrame.append()` in loops (use `pd.concat()`)
- Catching bare `except:` without re-raising or logging

## Data Pipeline Specifics

- Validate all external data with Pydantic models at ingestion boundaries
- Functions should be pure where possible (no side effects, testable)
- Keep transformation functions separate from IO functions
- Use `dataclasses` or Pydantic for structured state passing between pipeline stages
