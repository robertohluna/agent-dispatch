# Agent Briefing

> READ THIS FIRST. If you are an AI agent analyzing this repository or a project that contains it, this document tells you exactly what you're looking at and what you're about to do.

---

## What Is Agent Dispatch

Agent Dispatch is a **multi-agent sprint coordination system**. It is markdown documentation — not software. You drop it into any codebase and it tells AI coding agents how to work together in parallel on the same project without stepping on each other.

**You are reading this because** one of these is true:
1. You've been asked to **plan a sprint** — analyze a codebase and generate a dispatch plan
2. You've been **activated as an agent** — you have a specific role, branch, and task list
3. You're **analyzing this repo** to understand how it works before recommending or using it

This document covers all three scenarios.

---

## If You're the ORCHESTRATOR (Agent O)

You are the **ORCHESTRATOR** — the sprint command agent. You plan, monitor, and grade. Your job:

**Wave 0 — Planning:**
1. **Read** the project's codebase — understand architecture, patterns, conventions, tech stack
2. **Read** `agents/orchestrator.md` — your role, grading rubric, assessment template
3. **Read** `guides/sprint-planner.md` — this is your methodology, follow it step by step
4. **Discover** work — bugs, technical debt, security gaps, missing tests, features to build
5. **Write execution traces** — traced paths from entry point to root cause for each piece of work
6. **Propose a sprint** — assign agents, organize into waves, define success criteria
7. **Generate documents** after operator approval:
   - `DISPATCH.md` — the sprint plan
   - Per-agent task docs (`agent-X-[domain].md`) — exact implementation specs
   - Activation prompts — ready to paste into separate agent terminals

**Waves 1-5 — Monitoring:**
8. **Track** agent progress, chain completion, blockers
9. **Intervene** when agents drift off-mission or violate territory
10. **Escalate** P0 discoveries and cross-agent blockers to the operator

**Wave 6 — Grading:**
11. **Read** every agent's completion report and branch diff
12. **Grade** each agent on: completeness, correctness, mission alignment, territory, conventions
13. **Produce** `SPRINT-ASSESSMENT.md` with per-agent grades and carry-forward work

**Key files to read:**
- `agents/orchestrator.md` — Your role definition and grading rubric
- `guides/sprint-planner.md` — Your step-by-step planning methodology
- `agents/README.md` — The 10 agent roles, wave structure, merge order
- `templates/activation.md` — How to structure activation prompts
- `templates/dispatch.md` — Sprint plan template
- `templates/agent.md` — Per-agent task doc template
- `config/dispatch-styles.md` — Execution style blocks to append to every prompt
- `config/code-standards.md` — Coding discipline every agent must follow

**Critical:** Scale agents to the work. A 3-chain bug fix needs ORCHESTRATOR + 2 agents, not 10. A full-stack migration might need all of them. Don't dispatch agents that don't have work.

---

## If You've Been Activated as an Agent

You have a role (BACKEND, FRONTEND, DATA, etc.), a branch, a territory, and chains to execute.

**Your operating protocol:**

1. **Read your context files** — The numbered list at the top of your activation prompt. Read all of them before writing any code.
2. **Match the codebase** — Naming conventions, error handling, architectural patterns. Your code must be indistinguishable from the existing codebase.
3. **Execute chains in order** — One chain at a time. Trace → diagnose → fix → verify → document. Complete each fully before starting the next.
4. **Stay in your territory** — Modify only files you're authorized to modify. Read anything for context.
5. **Write minimal correct code** — Shortest implementation that solves the problem. No over-engineering. No under-engineering. Match the codebase patterns exactly.
6. **Verify before completing** — Build must pass. Tests must pass. No exceptions.
7. **Write your completion report** — Document everything: what changed, what was blocked, P0 discoveries, files modified, verification results.

**Key files to read:**
- Your activation prompt (given to you by the operator)
- Your task doc (`agent-X-[domain].md`)
- `config/code-standards.md` — How to write code
- `templates/completion.md` — Completion report template

**If you use sub-agents:** Each sub-agent gets ONE chain or ONE well-defined subtask. They inherit your territory. You validate their combined output. Sub-agents saying "done" is not sufficient — you verify.

---

## If You're Analyzing This Repo

Here's the architecture in 60 seconds:

### The Problem It Solves

When you run multiple AI agents on the same codebase, they:
- Modify the same files (merge conflicts)
- Miss context about what other agents are doing (integration failures)
- Work on vague tasks (scattered, incomplete fixes)
- Produce no documentation (can't verify or merge with confidence)

### The Solution

**Execution traces** replace vague tasks:
```
Bad:  "Fix bugs in the payment handlers"
Good: "POST /webhooks/stripe → webhookHandler.ProcessEvent() → paymentService.HandleInvoicePaid()
       → subscriptionStore.Activate(). Signal: 504 timeout. Mutex held during network I/O.
       Fix: release lock before notification call. Verify: webhook responds <2s."
```

**Territory isolation** prevents conflicts — each agent owns specific directories and files.

**Wave ordering** respects dependencies — DATA before BACKEND before FRONTEND.

**Merge validation** catches breakage — build + test after every single merge.

**Completion reports** enable trust — detailed reports let ORCHESTRATOR grade and LEAD merge confidently.

### The 10 Agents

| Agent | Domain | Wave | Merge Order |
|-------|--------|------|-------------|
| ORCHESTRATOR (O) | Sprint planning, monitoring, grading | 0 + 6 | Does not merge |
| DATA (F) | Models, stores, migrations | 1 | 1st |
| QA (E) | Tests, security audits | 1 | 7th |
| INFRA (C) | Docker, CI/CD, deployment | 1 | 6th |
| DESIGN (H) | Tokens, specs, a11y | 1 | 2nd |
| BACKEND (A) | Handlers, routes, services | 2 | 3rd |
| SERVICES (D) | Workers, integrations | 2 | 4th |
| FRONTEND (B) | Components, routes, stores | 3 | 5th |
| RED TEAM (R) | Adversarial review | 4 | Does not merge |
| LEAD (G) | Merge authority, ship | 5 | 8th (last) |

### The Sprint Lifecycle

```
PLAN → DISPATCH → EXECUTE → MONITOR → MERGE → GRADE → SHIP

1. ORCHESTRATOR analyzes codebase, proposes sprint (Wave 0)
2. Operator approves, sets up git worktrees (one branch per agent)
3. Operator pastes activation prompts into separate agent terminals
4. Agents work in parallel on isolated branches (Waves 1-3)
5. Each agent produces a completion report
6. RED TEAM reviews all branches for security/edge cases (Wave 4)
7. LEAD merges in dependency order, validates after each merge (Wave 5)
8. ORCHESTRATOR grades each agent against the mission, produces assessment (Wave 6)
9. Ship
```

### The File Structure

```
agent-dispatch/
├── config/          # Dispatch styles, code standards, project overrides
├── agents/          # 10 agent role definitions
├── core/            # Methodology, workflow, anti-patterns
├── guides/          # Sprint planner, operator's guide, quickstart, customization
├── runtime/         # Status tracking, reactions, interventions (while agents work)
├── scaling/         # 20-30+ agents, multi-repo coordination
├── templates/       # Dispatch plan, agent task doc, activation prompt, completion report
└── examples/        # 6 complete sprint dispatches across different tech stacks
```

### How It Scales

Each activation prompt goes into a **separate AI agent terminal** (Claude Code, Cursor, Aider, etc.). Each agent can spawn **5-15 sub-agents** internally for parallel chain execution. With multiple terminals running simultaneously:

| Terminals | Agents per Terminal | Total Parallel Agents |
|-----------|--------------------|-----------------------|
| 3 | 5-8 | 15-24 |
| 6 | 5-8 | 30-48 |
| 9 | 5-15 | 45-135 |

See `guides/parallel-execution.md` for the full scaling model.

### What It Is Not

- **Not software.** No runtime, no daemon, no CLI, no dependencies.
- **Not a framework.** No code to install. Markdown docs you drop into a repo.
- **Not tool-specific.** Works with Claude Code, Cursor, Aider, Codex, Windsurf, any AI coding agent.
- **Not stack-specific.** Works with Go, TypeScript, Python, Rust, Elixir, Java, C, PHP, Ruby, C#.

---

## Decision: Should I Use This?

**Use Agent Dispatch when:**
- Running 2+ AI agents on the same codebase simultaneously
- Fixing multiple bugs or building features in parallel
- Performing codebase-wide audits (security, performance, testing)
- Migrating or refactoring large codebases
- Needing clear visibility into what each agent changed and why

**Don't use when:**
- Single agent, single task (just prompt the agent directly)
- The work is too small to justify planning overhead (1-2 files, 1 chain)
- You don't use git (the system depends on branches and worktrees)

---

**Related Documents:**
- [guides/sprint-planner.md](sprint-planner.md) — Full planning methodology (start here for sprint planning)
- [guides/quickstart.md](quickstart.md) — 5-minute overview
- [guides/operators-guide.md](operators-guide.md) — Complete operator tutorial
- [guides/parallel-execution.md](parallel-execution.md) — Scaling model for parallel terminals
