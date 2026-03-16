---
name: claude-agents
description: Patterns for building Claude Code agents, MCP servers, skills, hooks, and multi-agent systems using the Anthropic SDK and Claude Code plugin system.
origin: dham-claude-code
---

# Claude Agents & Skills Patterns

Use this skill when building Claude Code agents, custom skills, MCP servers, or Anthropic API integrations.

## Claude Code Agent Format

```markdown
---
name: my-agent
description: What this agent does and WHEN to use it proactively.
tools: ["Read", "Grep", "Glob", "Bash"]
model: sonnet   # haiku | sonnet | opus
---

You are a specialist in [domain].

## Your Role
- Bullet points of responsibilities

## Process
1. Step one
2. Step two

## Patterns
[code examples]

## Red Flags
- Anti-patterns to flag
```

## Skill Format

```markdown
---
name: skill-name
description: One-line description for context matching.
origin: your-plugin-name
---

# Skill Title

Use this skill when [trigger condition].

## Pattern 1
[explanation + code]

## Pattern 2
[explanation + code]

## Key Rules
- Rule 1
- Rule 2
```

## Anthropic SDK — Python

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

# Basic completion
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)
print(message.content[0].text)

# Streaming
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a pipeline..."}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

# Tool use
tools = [{
    "name": "get_dag_status",
    "description": "Get the current status of an Airflow DAG",
    "input_schema": {
        "type": "object",
        "properties": {
            "dag_id": {"type": "string", "description": "The DAG ID"}
        },
        "required": ["dag_id"]
    }
}]

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "Check status of orders_pipeline"}]
)
```

## MCP Server Pattern (Python)

```python
from mcp.server import Server
from mcp.server.models import InitializationOptions
import mcp.types as types

server = Server("my-data-tools")

@server.list_tools()
async def handle_list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="query_databricks",
            description="Run a SQL query on Databricks",
            inputSchema={
                "type": "object",
                "properties": {
                    "sql": {"type": "string"},
                    "catalog": {"type": "string", "default": "prod_catalog"}
                },
                "required": ["sql"]
            }
        )
    ]

@server.call_tool()
async def handle_call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "query_databricks":
        result = await run_databricks_query(arguments["sql"], arguments.get("catalog", "prod_catalog"))
        return [types.TextContent(type="text", text=str(result))]
    raise ValueError(f"Unknown tool: {name}")
```

## Claude Code Hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[hook] File modified: checking for secrets...' && grep -r 'sk-' $CLAUDE_TOOL_OUTPUT || true"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo '# Session started' >> ~/.claude/session-log.md"
          }
        ]
      }
    ]
  }
}
```

## Model Selection Guide

| Task | Model |
|------|-------|
| Architecture, root-cause analysis | claude-opus-4-6 |
| Implementation, code review | claude-sonnet-4-6 |
| Classification, boilerplate | claude-haiku-4-5 |

## Plugin Registration

To use your plugin with Claude Code:
1. Push repo to GitHub
2. Add to `~/.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "your-marketplace-name": {
      "source": { "source": "github", "repo": "your-username/your-repo" }
    }
  }
}
```
3. Install: `/plugin install your-plugin-name@your-marketplace-name`
