# dam-claude-code

Personal Claude Code plugin for data & cloud architecture, AWS, Databricks, dbt, automation, iOS, and web development.

## What's Included

| Component | Count | Coverage |
|-----------|-------|----------|
| Agents | 8 | Data architecture, AWS, Databricks, dbt, iOS, web, automation, database |
| Skills | 10 | aws-patterns, databricks-patterns, dbt-patterns, data-architecture, ios-swiftui, automation-patterns, database-design, claude-agents, frontend-patterns, python-data-patterns |
| Commands | 5 | /data-arch-review, /aws-review, /dbt-review, /ios-review, /automation-review |
| Rules | 3 | python, sql, swift |
| Hooks | 2 | Secret detection on file writes, session logging |

## Installation

### From GitHub (once published)

1. Add marketplace to `~/.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "dha": {
      "source": { "source": "github", "repo": "DamyTheKnightKing/dam-claude-code" }
    }
  }
}
```

2. Install via Claude Code:
```
/plugin install dam-claude-code@dha
```

### Local Development

Clone this repo, then symlink or reference the local path in your Claude Code settings.

## Agents

- **data-architect** — Medallion architecture, data mesh, lakehouse design, data contracts
- **aws-architect** — AWS CDK, IAM, S3, Glue, Step Functions, cost optimization
- **databricks-expert** — Delta Lake, Unity Catalog, DLT, Asset Bundles, PySpark tuning
- **dbt-reviewer** — Model layering, testing coverage, incremental strategies, macros
- **ios-reviewer** — SwiftUI, Swift 6 concurrency, @Observable, SwiftData
- **automation-engineer** — Airflow DAGs, LangGraph multi-agent, n8n, CI/CD
- **database-reviewer** — PostgreSQL, Redshift, Databricks SQL, schema design
- **web-developer** — Next.js App Router, TypeScript, React Query, Tailwind

## Commands

- `/data-arch-review` — Review data platform architecture
- `/aws-review` — AWS security, cost, and reliability audit
- `/dbt-review` — dbt model quality and testing coverage
- `/ios-review` — iOS Swift/SwiftUI code review
- `/automation-review` — Airflow, LangGraph, n8n workflow review

## License

MIT
