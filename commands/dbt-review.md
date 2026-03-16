---
description: Review dbt models for layer structure, testing coverage, incremental strategies, and project conventions. Invokes dbt-reviewer agent.
---

Review the dbt project in this codebase.

Use the `dbt-reviewer` agent to:

1. Verify model layering (staging → intermediate → marts, correct prefixes)
2. Check test coverage on all models (not_null, unique on PKs, relationships)
3. Validate incremental model configurations (unique_key, strategy, on_schema_change)
4. Identify hardcoded schema references that should use `{{ source() }}` / `{{ ref() }}`
5. Review macros for reusability and correctness
6. Check dbt_project.yml config for materialization and schema assignments

Report findings as: **Breaking** (incorrect behavior), **Missing Tests** (quality risk), **Convention** (project structure), **Performance** (slow builds).
