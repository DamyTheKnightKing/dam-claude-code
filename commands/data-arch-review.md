---
description: Review data architecture decisions, pipeline design, and medallion layer structure. Invokes data-architect agent.
---

Review the data architecture in this codebase or the described design.

Use the `data-architect` agent to:

1. Analyze the current data flow (ingestion → processing → serving)
2. Evaluate the medallion architecture layer separation (Bronze/Silver/Gold)
3. Check for missing data contracts or domain ownership gaps
4. Identify partitioning, incremental strategy, and schema evolution issues
5. Recommend improvements with trade-off analysis

Produce a prioritized list of findings with: **Critical** (blocks correctness), **High** (performance/cost risk), **Medium** (maintainability), and **Low** (nice-to-have).
