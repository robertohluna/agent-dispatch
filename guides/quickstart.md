# Agent Dispatch — 5-Minute Overview

A methodology for running up to 9 AI coding agents in parallel on the same codebase.
Each agent gets an isolated git branch, a defined territory, and a list of tasks.
You coordinate. They work. You merge. You ship.

---

## The Idea

```
You (Operator)
  → dispatch agents, each on its own git branch
  → they work in parallel, no file conflicts
  → you merge branches in dependency order
  → build + test after each merge
  → ship
```

---

## The 9 Agents

| Code | Name    | What it does                                    |
|------|---------|-------------------------------------------------|
| A    | BACKEND   | Backend logic — handlers, services, API routes  |
| B    | FRONTEND   | Frontend UI — components, routes, stores        |
| C    | INFRA | Infrastructure — Docker, CI/CD, builds          |
| D    | SERVICES   | Specialized services — workers, integrations    |
| E    | QA    | QA and security — tests, audits, scanning       |
| F    | DATA | Data layer — models, migrations, storage        |
| G    | LEAD    | Orchestrator — merges, docs, ship decisions     |
| H    | DESIGN   | Design & Creative — design system, tokens, a11y, visual specs |
| R    | RED TEAM  | Adversarial review — break other agents' work before merge |

---

## How It Actually Works

1. **Plan** — Give your AI the [Sprint Planner](sprint-planner.md) guide. It reads the codebase, discovers work, traces bugs, and proposes a sprint. Or plan manually — your choice.
2. **Branch** — `git worktree` creates isolated copies, one per agent
3. **Prompt** — Paste activation prompts into each agent's terminal
4. **Monitor** — Check chain states, answer questions, handle escalations
5. **Merge** — Merge branches in dependency order (data → backend → frontend)
6. **Ship** — Tag the release, delete worktrees, done

---

## The Execution Method

This is what separates Agent Dispatch from "just ask Claude to fix things."

- **Execution traces** — Don't tell agents "look in src/". Trace the bug from entry
  point to root cause and give them that exact path. They follow the signal.

- **Chain execution** — Finish one fix completely (trace → fix → verify) before
  starting the next. No context-switching mid-chain.

- **Priority levels** — P0: stop everything and escalate. P1: fix this sprint
  first. P2: fix this sprint. P3: fix if time permits.

- **Critical escalation** — If an agent finds something critical that wasn't in its
  task list (data corruption, security hole), it stops and alerts you immediately.

---

## Works With

Claude Code, Qwen Coder, OpenCode, Cursor, Windsurf, Aider, Continue,
GitHub Copilot Workspace — anything that can read files and run commands.
Stack-agnostic: Go, TypeScript, Python, Rust, Elixir, PHP, Java, C/C++, more.

---

## Next Step

Read `operators-guide.md` for the full tutorial — setup, sprint planning,
activation prompts, merge procedures, and worked examples.
