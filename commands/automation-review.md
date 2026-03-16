---
description: Review Airflow DAGs, LangGraph agents, or n8n workflows for correctness, reliability, and observability. Invokes automation-engineer agent.
---

Review the automation code (Airflow DAGs, LangGraph agents, n8n workflows, or CI/CD pipelines) in this codebase.

Use the `automation-engineer` agent to:

1. **Airflow**: Check catchup setting, retry config, failure callbacks, XCom payload sizes, DAG complexity
2. **LangGraph**: Verify state is typed dataclass, agents are pure functions, confidence-based routing, audit logging
3. **n8n**: Check credential handling, error workflows, batch processing for large lists
4. **CI/CD**: Validate dbt slim CI, DAG integrity tests, secret management in workflows

Report findings as: **Data Loss Risk** (incorrect catchup/retry), **Silent Failure** (missing callbacks/error handling), **Performance** (inefficient patterns), **Security** (credential exposure).
