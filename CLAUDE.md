# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Orchestration Philosophy

**Core principles (in order of priority):**
1. **Plan before execute** — for any task touching 3+ files or with unclear scope, stop and plan first
2. **Agent-first** — delegate specialist work to subagents; don't do everything inline
3. **Parallel by default** — independent tasks run simultaneously, not sequentially
4. **Minimal complexity** — solve the problem in front of you; don't design for hypothetical future needs
5. **Define done** — every task starts with a clear completion condition

---

## Pre-Task Planning Protocol

Before starting any non-trivial task:

1. **Restate** what's being asked in one sentence
2. **Identify risks** — what could go wrong, what's irreversible
3. **Decompose** into units that are: independently verifiable, single-purpose, <15 min each
4. **Check for blockers** — missing context, unclear requirements → ask before coding
5. **Plan First** — write plan to `tasks/todo.md` with checkable items
6. **Verify Plan** — check in before starting implementation
7. **Track Progress** — mark items complete as you go
8. **Explain Changes** — provide high-level summary at each step
9. **Document Results** — add review section to `tasks/todo.md`
10. **Capture Lessons** — update `tasks/lessons.md` after corrections

Skip planning only for: single-file edits, clear bug fixes, renaming, or trivial additions.

---

## Subagent Delegation

- Use **subagents liberally** to keep the main context window clean.
- Offload **research, exploration, and parallel analysis** to subagents.
- For complex problems, **throw more compute at it via subagents**.
- **One task per subagent** for focused execution.
- **Parallel execution:** when launching independent agents, send all Agent tool calls in one message.
- **Agent Routing:** pick the perfect optimized model for each process, don't over-consume model usage.

---

## Simplicity Rules (Anti-Over-Engineering)

- Solve what was asked — not a generalized version of it
- Three similar lines of code > a premature abstraction
- No helper utilities for one-time operations
- No feature flags or backwards-compat shims unless explicitly needed
- No error handling for scenarios that cannot happen
- No docstrings/comments on code you didn't change
- If the fix is 5 lines, don't refactor the surrounding 50
- **When in doubt: do less, ask more**

---

## Self-Improvement Loop

- After **any correction from the user**, update `tasks/lessons.md` with the pattern
- Write **rules for yourself** that prevent the same mistake
- **Ruthlessly iterate** on these lessons until the mistake rate drops
- **Review lessons at session start** for the relevant project

---

## Verification Before Done

- Never mark a task **complete without proving it works**
- **Diff behavior** between main and your changes when relevant
- Ask yourself: *"Would a staff engineer approve this?"*
- **Run tests, check logs, demonstrate correctness**

---

## Demand Elegance (Balanced)

- For non-trivial changes: pause and ask *"Is there a more elegant way?"*
- If a fix feels hacky: *"Knowing everything I know now, implement the elegant solution."*
- Skip this for **simple, obvious fixes** — don't over-engineer
- **Challenge your own work before presenting it**

---

## Autonomous Bug Fixing

- When given a **bug report: just fix it** — don't ask for hand-holding
- Point at **logs, errors, failing tests** — then resolve them
- **Zero context switching required** from the user
- Go fix **failing CI tests** without being told how

---

## Lessons Learned

- `catchup=False` on all Airflow DAGs — omitting it has caused mass backfills
- Never XCom large payloads in Airflow — use S3 paths instead
- LangGraph state must be a typed dataclass, not a plain dict — routing bugs are silent otherwise
- Always `EXPLAIN ANALYZE` before adding a PostgreSQL index — assumed slow != actually slow
- dbt staging models must never contain joins — caught late = expensive refactor
- Parameterize all SQL — f-string interpolation in pipeline code has caused prod incidents

---

## Knowledge Capture Rules

- Debugging notes and session context → auto memory (`~/.claude/projects/.../memory/`)
- Architecture decisions → ADR files in the project repo
- Repeating mistakes → add to **Lessons Learned** above
- If a project already has docs structure, follow it — don't create a new top-level doc
- Don't duplicate knowledge that's already in code, comments, or git history

---

## Session Strategy

- Continue the same session for closely-coupled work
- Start a fresh session after major phase transitions (design → implement → test)
- Compact context after a milestone completes, not during active debugging
- Save session state with `/save-session` before long gaps

---
