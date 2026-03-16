---
name: automation-patterns
description: Automation patterns for Airflow DAG design, LangGraph multi-agent systems, n8n workflows, and CI/CD pipeline configuration.
origin: dham-claude-code
---

# Automation Patterns

Use this skill for designing and building automation systems including Airflow pipelines, LangGraph agents, n8n workflows, and CI/CD.

## Airflow DAG Patterns

### TaskFlow API (preferred)
```python
from airflow.decorators import dag, task
from airflow.operators.trigger_dagrun import TriggerDagRunOperator
from datetime import datetime, timedelta

default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,
    "on_failure_callback": slack_alert,
}

@dag(
    dag_id="daily_data_pipeline",
    schedule="@daily",
    start_date=datetime(2025, 1, 1),
    catchup=False,
    max_active_runs=1,
    default_args=default_args,
    tags=["data-platform", "daily"],
)
def daily_data_pipeline():
    @task
    def extract_from_source() -> dict:
        # Return small metadata, not actual data
        return {"s3_path": "s3://bucket/raw/2025-03-16/", "record_count": 5000}

    @task
    def validate_data(metadata: dict) -> dict:
        assert metadata["record_count"] > 0, "No records extracted"
        return metadata

    @task
    def trigger_dbt(metadata: dict):
        # Trigger dbt job via API
        pass

    metadata = extract_from_source()
    validated = validate_data(metadata)
    trigger_dbt(validated)

dag = daily_data_pipeline()
```

### DAG Organization
```
dags/
├── ingestion/          # Source extraction DAGs
├── transformation/     # dbt / Spark transformation DAGs
├── reporting/          # Downstream delivery DAGs
└── utils/              # Shared callbacks, helpers
```

## LangGraph Agent Pattern

```python
from dataclasses import dataclass, field
from langgraph.graph import StateGraph, END
from typing import List, Optional

@dataclass
class PipelineState:
    """Shared state flows through all agents unchanged except additions."""
    input_data: dict
    validated: bool = False
    transformed_data: Optional[dict] = None
    errors: List[str] = field(default_factory=list)
    should_escalate: bool = False
    notifications_sent: List[str] = field(default_factory=list)

# Agent functions: pure — take state, return updated state
def validate_agent(state: PipelineState) -> PipelineState:
    try:
        assert state.input_data.get("records"), "No records"
        state.validated = True
    except AssertionError as e:
        state.errors.append(str(e))
        state.should_escalate = True
    return state

def transform_agent(state: PipelineState) -> PipelineState:
    if not state.validated:
        return state
    state.transformed_data = {k: v for k, v in state.input_data.items()}
    return state

def route_after_validation(state: PipelineState) -> str:
    return "notify" if state.should_escalate else "transform"

# Build graph
workflow = StateGraph(PipelineState)
workflow.add_node("validate", validate_agent)
workflow.add_node("transform", transform_agent)
workflow.add_node("notify", notify_agent)

workflow.set_entry_point("validate")
workflow.add_conditional_edges("validate", route_after_validation)
workflow.add_edge("transform", "notify")
workflow.add_edge("notify", END)

graph = workflow.compile()
```

## n8n Workflow Patterns

- **Webhook trigger** → Validate → Transform → Store → Notify
- Use **Error Workflow** node on every production workflow
- **SplitInBatches** for processing >100 items
- **Wait** node + webhook resume for human approval steps
- Store all credentials in n8n Credential Store (never in node expressions)

## GitHub Actions CI/CD

```yaml
name: Data Platform CI
on:
  pull_request:
    paths: ['dbt/**', 'airflow_dags/**']

jobs:
  dbt-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dbt
        run: pip install dbt-databricks
      - name: dbt slim CI
        run: |
          dbt deps
          dbt build --select state:modified+ --defer --state ./prod-artifacts
        env:
          DBT_DATABRICKS_TOKEN: ${{ secrets.DBT_TOKEN }}

  dag-integrity:
    runs-on: ubuntu-latest
    steps:
      - name: Validate DAG structure
        run: python -m pytest tests/test_dag_integrity.py -v
```

## Monitoring & Alerting

```python
# Slack alert callback for Airflow
def slack_alert(context):
    from airflow.providers.slack.hooks.slack_webhook import SlackWebhookHook
    dag_id = context['dag'].dag_id
    task_id = context['task_instance'].task_id
    execution_date = context['execution_date']
    SlackWebhookHook(slack_webhook_conn_id="slack_alerts").send(
        text=f":red_circle: *{dag_id}.{task_id}* failed on {execution_date}"
    )
```
