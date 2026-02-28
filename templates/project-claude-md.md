# Project CLAUDE.md Template for Agent Dispatch

> Drop this into your project's CLAUDE.md to wire Agent Dispatch into Claude Code.
> Customize the placeholders for your project.

---

## Template

Copy everything below into your project's `CLAUDE.md` or `.claude/CLAUDE.md`:

```markdown
# [PROJECT NAME]

## Overview
[1-3 sentences: what this project is, what it does, what stack it uses]

## Tech Stack
- Language: [Go / TypeScript / Python / etc.]
- Framework: [Chi / Next.js / FastAPI / etc.]
- Database: [PostgreSQL / SQLite / MongoDB / etc.]
- Frontend: [React / Svelte / Vue / etc.] (if applicable)

## Build & Test
```bash
# Build
[your build command]

# Test
[your test command]

# Lint
[your lint command]

# Full verification
[build] && [test] && [lint]
```

## Project Structure
```
[your project's directory layout — key directories only]
```

## Architecture Patterns
- [e.g., handler → service → store pattern]
- [e.g., repository pattern for data access]
- [e.g., middleware chain for auth/logging]

## Conventions
- [e.g., camelCase for functions, PascalCase for types]
- [e.g., errors wrapped with context: fmt.Errorf("operation: %w", err)]
- [e.g., all API responses use { data, error, meta } format]

## Agent Dispatch

This project uses [Agent Dispatch](docs/agent-dispatch/) for multi-agent development sprints.

### Sprint Planning
To plan a sprint, use the dispatcher prompt:
```
Read docs/agent-dispatch/templates/dispatcher-prompt.md and follow it.
Sprint goal: [what you want to accomplish]
```

### Key Files for Agents
- `docs/agent-dispatch/guides/agent-briefing.md` — System overview
- `docs/agent-dispatch/config/code-standards.md` — How to write code
- `docs/agent-dispatch/config/dispatch-styles.md` — Prompt style blocks
- `CLAUDE.md` — This file (project context)

### Territory Map (Project-Specific)
| Agent | Territory |
|-------|-----------|
| DATA | [your data layer dirs] |
| BACKEND | [your handler/service dirs] |
| FRONTEND | [your UI dirs] |
| SERVICES | [your integration/worker dirs] |
| INFRA | [Dockerfile, .github/, Makefile, config/] |
| QA | [test dirs] |
| DESIGN | [design/style dirs] |

### Build Commands for Agents
```bash
# Every agent runs these before completing
[build command]
[test command]
```
```

---

## Notes

- Replace all `[placeholder]` values with your project specifics
- The territory map should match your actual directory structure
- Build commands must work from any worktree (use relative paths or project root)
- Agents read this file as their first context item — keep it accurate
