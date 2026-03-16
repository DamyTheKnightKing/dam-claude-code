# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Plugin Is

A personal Claude Code plugin for data & cloud architecture, AWS, Databricks, dbt, automation, database design, iOS, and web development.

## Structure

- **agents/** — Specialized sub-agents (invoke via `Agent` tool)
- **skills/** — Domain knowledge loaded as context
- **commands/** — User-facing slash commands
- **hooks/** — Lifecycle automation (session logging, secret detection)
- **rules/** — Always-follow coding standards (python/, sql/, swift/)

## Agents

| File | Purpose | Model |
|------|---------|-------|
| `data-architect.md` | Medallion architecture, data mesh, lakehouse design | opus |
| `aws-architect.md` | AWS infrastructure, CDK, IAM, cost optimization | opus |
| `databricks-expert.md` | Delta Lake, Unity Catalog, DLT, PySpark | sonnet |
| `dbt-reviewer.md` | dbt model quality, testing, incremental patterns | sonnet |
| `ios-reviewer.md` | SwiftUI, Swift Concurrency, MVVM | sonnet |
| `automation-engineer.md` | Airflow, LangGraph, n8n, CI/CD | sonnet |
| `database-reviewer.md` | PostgreSQL, Redshift, SQL optimization | sonnet |
| `web-developer.md` | Next.js App Router, TypeScript, React Query | sonnet |

## Commands

| Command | What It Does |
|---------|-------------|
| `/data-arch-review` | Review medallion architecture and data flow |
| `/aws-review` | Security, cost, and reliability review of AWS infra |
| `/dbt-review` | Model layering, test coverage, incremental strategies |
| `/ios-review` | Swift/SwiftUI architecture and concurrency review |
| `/automation-review` | Airflow DAGs, LangGraph agents, n8n workflows |

## Adding a New Skill

1. Create `skills/your-skill-name/SKILL.md`
2. Add YAML frontmatter: `name`, `description`, `origin: dham-claude-code`
3. Write structured content: When to use, patterns, code examples, key rules

## Adding a New Agent

1. Create `agents/your-agent.md`
2. Add frontmatter: `name`, `description`, `tools`, `model`
3. Follow the pattern: Role → Process → Code Patterns → Red Flags

## GitHub Plugin Setup

To install this plugin from GitHub in Claude Code:

```json
// Add to ~/.claude/settings.json
{
  "extraKnownMarketplaces": {
    "dham": {
      "source": { "source": "github", "repo": "DamyTheKnightKing/dham-claude-code" }
    }
  }
}
```

Then: `/plugin install dham-claude-code@dham`
