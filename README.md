# Agent Dispatch

> Multi-agent development sprint framework for AI coding agents.
> Drop into any codebase. Coordinate parallel AI agents. Ship faster.

---

## What This Is

Agent Dispatch coordinates multiple AI coding agents working simultaneously on a single codebase. You describe what needs to happen. An AI planner reads your codebase, maps the architecture, discovers work, writes execution traces, and proposes a sprint. You approve. It generates everything — the dispatch plan, per-agent task docs with exact implementation specs, and activation prompts ready to paste into agent terminals. Each agent works on its own git branch with defined file-level territory. When they finish, branches merge in dependency order with build + test validation after every merge.

**It is:** A methodology, a planning system, and a template library. Markdown docs you drop into any repo.

**It is not:** Software. No runtime, no daemon, no CLI, no SDK. The agents are whatever AI coding tools you already use — Claude Code, Cursor, Codex, Aider, Windsurf, or anything else that can read files and run commands.

---

## How It Works

```
PLAN → DISPATCH → EXECUTE → MONITOR → MERGE → SHIP

1. PLAN
   You: "Fix the auth bugs, add rate limiting, close the security findings"
   AI reads: sprint-planner.md + your codebase
   AI produces: sprint proposal (agents, waves, chains, success criteria)
   You: approve / adjust

2. DISPATCH
   AI generates: DISPATCH.md + per-agent task docs + activation prompts
   You run: git worktree setup (one isolated branch per agent)
   You paste: activation prompts into agent terminals

3. EXECUTE (agents work in parallel, in waves)
   Wave 1: DATA, QA, INFRA, DESIGN       → foundation, no dependencies
   Wave 2: BACKEND, SERVICES              → need stable data layer
   Wave 3: FRONTEND                       → needs design specs + backend API
   Wave 4: RED TEAM                       → adversarial review of all branches
   Wave 5: LEAD                           → merge + ship

4. MONITOR
   Track agent status, chain progress, sprint health
   React to events (CI fails, stuck agents, territory violations)
   Intervene with correction messages when needed

5. MERGE
   RED TEAM reviews all branches for security/edge cases (can BLOCK merge)
   LEAD merges branches in dependency order: DATA → DESIGN → BACKEND → ...
   Build + test after EVERY merge. No exceptions.

6. SHIP
   Tag release. Delete worktrees. Done.
```

---

## The Core Idea: Execution Traces

This is what separates Agent Dispatch from "just ask an AI to fix things."

Agents don't work on "directories." They work on **execution traces** — a traced path through the codebase from entry point to root cause, with the exact fix site and verification steps.

```
Weak prompt:
  "BACKEND: fix bugs in handlers/"
  → Agent scans randomly, makes scattered changes, misses root causes

Execution trace:
  "Chain 1 [P1]: POST /webhooks/stripe → webhookHandler.ProcessEvent()
   → paymentService.HandleInvoicePaid() → subscriptionStore.Activate()
   Signal: 504 timeout. Mutex held during network I/O under lock.
   Fix: subscriptionStore.Activate() — release lock before notification call.
   Verify: Test webhook returns < 2s. No race condition on concurrent activations."
  → Agent follows the signal, finds the exact failure point, fixes surgically
```

Every agent gets chains like this. They complete one fully (trace → fix → verify → document) before starting the next. Priority order: P0 (stop everything) → P1 (critical) → P2 (important) → P3 (if time permits).

See [core/methodology.md](core/methodology.md) for the full theory.

---

## Quick Start

### 1. Drop into your project

```bash
cp -r agent-dispatch/ your-project/docs/agent-dispatch/
```

### 2. Plan your first sprint

Give your AI this prompt:

```
Read docs/agent-dispatch/guides/sprint-planner.md — follow it step by step.
Then read the codebase. Analyze the architecture, discover work, write execution
traces, and propose a sprint plan.

Sprint goal: [what you want to accomplish]
```

The AI reads the Sprint Planner guide, analyzes your codebase, maps territories, discovers bugs/debt/security gaps, writes execution traces, and proposes a sprint with agents, waves, and success criteria. You review and approve.

### 3. Dispatch

After approval, the AI generates:
- `sprint-XX/DISPATCH.md` — the full sprint plan
- `sprint-XX/agent-X-*.md` — per-agent task docs with exact implementation specs
- Activation prompts — ready to paste into agent terminals

Set up worktrees, paste prompts, agents start working.

### 4. Run

Follow the [Operator's Guide](guides/operators-guide.md) for the full tutorial, or the [Quick Start](guides/quickstart.md) for the fastest path.

---

## The 9 Agents

| Code | Name | Domain | Wave |
|------|------|--------|------|
| F | **DATA** | Models, stores, migrations, queries | 1 |
| E | **QA** | Tests, security audits, coverage | 1 |
| C | **INFRA** | Docker, CI/CD, builds, deployment | 1 |
| H | **DESIGN** | Design tokens, component specs, accessibility | 1 |
| A | **BACKEND** | Handlers, routes, services, middleware | 2 |
| D | **SERVICES** | Workers, integrations, external API clients | 2 |
| B | **FRONTEND** | Components, routes, stores, hooks | 3 |
| R | **RED TEAM** | Adversarial review — break other agents' work before merge | 4 |
| G | **LEAD** | Merge, docs, ship decision | 5 |

Scale to the work. A 3-chain bug fix needs 2 agents, not 9. A full-stack migration might need all of them. For 20-30+ agents, roles split into nested teams — see [scaling/scaling.md](scaling/scaling.md).

Each agent has its own file in [`agents/`](agents/) with territory definitions, responsibilities, wave placement, and merge order.

---

## What Gets Generated

When the AI plans a sprint, it produces three types of documents:

### DISPATCH.md — The Sprint Plan
Sprint goals, execution traces for every chain, wave assignments, agent territories, merge order, success criteria, worktree setup script. Template: [templates/dispatch.md](templates/dispatch.md)

### Agent Task Docs — Per-Agent Implementation Specs
One per agent. Contains: numbered context reading list, files owned, tasks with IDs (current state → required changes with code examples), wave organization, territory rules with agent attribution, verification checklist with exact commands and expected output, commit strategy. Template: [templates/agent.md](templates/agent.md)

### Activation Prompts — Copy-Paste Into Terminals
One per agent. Contains: identity, context files to read, domain + cross-agent awareness, task summary by wave, chains with execution traces, territory, execution protocol (BEFORE/WHILE/AFTER coding methodology), completion instructions. Template: [templates/activation.md](templates/activation.md)

Agents produce **completion reports** when done — what changed, what was blocked, P0 discoveries, files modified, verification results. Template: [templates/completion.md](templates/completion.md)

---

## Project Structure

```
agent-dispatch/
├── README.md                         ← You are here
├── LICENSE
│
├── config/                           # Dispatch configuration
│   ├── README.md                     ← What goes in config
│   ├── dispatch-styles.md            ← Execution style: intensity, mesh triggers, optimal path
│   └── code-standards.md             ← Agent coding discipline: minimal, correct, no over-engineering
│
├── agents/                           # Agent role definitions (one per file)
│   ├── README.md                     ← Roster overview, wave structure, merge order
│   ├── backend.md                    ← Agent A — Backend Logic
│   ├── frontend.md                   ← Agent B — Frontend UI
│   ├── infra.md                      ← Agent C — Infrastructure
│   ├── services.md                   ← Agent D — Specialized Services
│   ├── qa.md                         ← Agent E — QA / Security
│   ├── data.md                       ← Agent F — Data Layer
│   ├── lead.md                       ← Agent G — Orchestrator
│   ├── design.md                     ← Agent H — Design & Creative
│   └── red-team.md                   ← Agent R — Adversarial Review
│
├── core/                             # Theory + methodology
│   ├── methodology.md                ← Execution traces, chain execution, priorities
│   ├── workflow.md                   ← Sprint lifecycle: plan → dispatch → merge → ship
│   └── anti-patterns.md              ← Common mistakes and how to avoid them
│
├── guides/                           # How-to guides
│   ├── agent-briefing.md             ← AI-facing system overview (READ THIS FIRST if you're an agent)
│   ├── parallel-execution.md         ← Scaling model: terminals × sub-agents = 100+ agents
│   ├── sprint-planner.md             ← AI onboarding: codebase → sprint plan (START HERE)
│   ├── quickstart.md                 ← 5-minute overview
│   ├── operators-guide.md            ← Full operator tutorial
│   ├── customization.md              ← Adapt territories for your stack
│   ├── tool-guide.md                 ← AI agent comparison + setup
│   ├── legacy-codebases.md           ← Legacy code: archaeology + characterization tests
│   └── dispatch-config.md            ← Machine-readable config for automation
│
├── runtime/                          # Runtime operations (while agents are working)
│   ├── reactions.md                  ← 12 decision trees for in-sprint events
│   ├── status-tracking.md            ← Agent states, chain progress, health indicators
│   └── interventions.md              ← 24 copy-paste correction messages
│
├── scaling/                          # Beyond the standard roster
│   ├── scaling.md                    ← Nested teams, role splitting, 20-30+ agents
│   └── multi-repo.md                 ← Multi-repository coordination
│
├── templates/                        # Copy-paste templates
│   ├── dispatch.md                   ← Sprint dispatch plan
│   ├── agent.md                      ← Per-agent task document
│   ├── activation.md                 ← Activation prompts + full dispatch flow
│   ├── completion.md                 ← Agent completion report
│   ├── status.md                     ← Sprint status board
│   ├── red-team-findings.md          ← RED TEAM findings report
│   └── retrospective.md             ← Sprint retrospective
│
└── examples/                         # Complete sprint dispatches
    ├── ecommerce-api/                ← Go + Chi + Stripe: Payment bug fix
    ├── saas-dashboard/               ← Python + FastAPI: Performance + security
    ├── realtime-chat/                ← Elixir + Phoenix: Reliability + scale
    ├── embedded-firmware/            ← C/C++ + FreeRTOS: Memory safety + OTA
    ├── legacy-php-webapp/            ← PHP 7.4 + jQuery: Security + modernization
    └── enterprise-java-api/          ← Java 17 + Spring Boot: Performance + events
```

---

## Documentation

### Start Here
| Document | What You Get |
|----------|-------------|
| [guides/agent-briefing.md](guides/agent-briefing.md) | **AI agents: read this first.** What the system is, what you're about to do, how you fit in. |
| [guides/sprint-planner.md](guides/sprint-planner.md) | Hand this to your AI. It reads your codebase and proposes a sprint. |
| [guides/quickstart.md](guides/quickstart.md) | 5-minute overview of the system |
| [guides/operators-guide.md](guides/operators-guide.md) | Full tutorial — planning, dispatch, monitoring, merge, ship |

### Agents
| Document | Role |
|----------|------|
| [agents/README.md](agents/README.md) | Roster overview, wave structure, merge order |
| [agents/backend.md](agents/backend.md) | Agent A — Backend Logic |
| [agents/frontend.md](agents/frontend.md) | Agent B — Frontend UI |
| [agents/infra.md](agents/infra.md) | Agent C — Infrastructure |
| [agents/services.md](agents/services.md) | Agent D — Specialized Services |
| [agents/qa.md](agents/qa.md) | Agent E — QA / Security |
| [agents/data.md](agents/data.md) | Agent F — Data Layer |
| [agents/lead.md](agents/lead.md) | Agent G — Orchestrator |
| [agents/design.md](agents/design.md) | Agent H — Design & Creative |
| [agents/red-team.md](agents/red-team.md) | Agent R — Adversarial Review |

### Methodology
| Document | Purpose |
|----------|---------|
| [core/methodology.md](core/methodology.md) | Execution traces, chain execution, priority levels |
| [core/workflow.md](core/workflow.md) | Sprint lifecycle: plan → dispatch → monitor → merge → ship |
| [core/anti-patterns.md](core/anti-patterns.md) | What not to do |

### Runtime Operations
| Document | Purpose |
|----------|---------|
| [runtime/reactions.md](runtime/reactions.md) | 12 decision trees for in-sprint events |
| [runtime/status-tracking.md](runtime/status-tracking.md) | Agent states, chain progress, sprint health |
| [runtime/interventions.md](runtime/interventions.md) | 24 copy-paste correction messages |

### Scaling
| Document | Purpose |
|----------|---------|
| [scaling/scaling.md](scaling/scaling.md) | Nested teams, role splitting, 20-30+ agents |
| [scaling/multi-repo.md](scaling/multi-repo.md) | Multi-repository coordination |

### Templates
| Template | Use For |
|----------|---------|
| [templates/dispatch.md](templates/dispatch.md) | Sprint plan |
| [templates/agent.md](templates/agent.md) | Per-agent task doc with implementation specs |
| [templates/activation.md](templates/activation.md) | Activation prompts + full dispatch pipeline |
| [templates/completion.md](templates/completion.md) | Agent completion report |
| [templates/status.md](templates/status.md) | Sprint status board |
| [templates/red-team-findings.md](templates/red-team-findings.md) | RED TEAM adversarial findings |
| [templates/retrospective.md](templates/retrospective.md) | Sprint retrospective |

### Config
| Document | Purpose |
|----------|---------|
| [config/dispatch-styles.md](config/dispatch-styles.md) | Execution style: mesh triggers, intensity protocol, optimal path, prompt assembly order |
| [config/code-standards.md](config/code-standards.md) | Agent coding discipline: minimal correct code, no over-engineering, conflict prevention |

### Guides
| Document | Purpose |
|----------|---------|
| [guides/agent-briefing.md](guides/agent-briefing.md) | AI-facing system overview — what this is, what you're doing, how you fit in |
| [guides/parallel-execution.md](guides/parallel-execution.md) | Scaling model: terminals × sub-agents, 30-100+ parallel agents |
| [guides/customization.md](guides/customization.md) | Adapt territories for your stack |
| [guides/tool-guide.md](guides/tool-guide.md) | AI agent comparison, setup, configuration |
| [guides/legacy-codebases.md](guides/legacy-codebases.md) | Legacy code: archaeology, characterization tests |
| [guides/dispatch-config.md](guides/dispatch-config.md) | Machine-readable config for automation |

### Examples
| Example | Stack | Sprint Theme |
|---------|-------|-------------|
| [E-Commerce API](examples/ecommerce-api/) | Go + Chi + PostgreSQL + Stripe | Payment Bug Fix |
| [SaaS Dashboard](examples/saas-dashboard/) | Python + FastAPI + PostgreSQL + React | Performance + Security |
| [Real-Time Chat](examples/realtime-chat/) | Elixir + Phoenix + LiveView + Redis | Reliability + Scale |
| [Embedded Firmware](examples/embedded-firmware/) | C/C++ + FreeRTOS + STM32 HAL | Memory Safety + OTA |
| [Legacy PHP Webapp](examples/legacy-php-webapp/) | PHP 7.4 + MySQL + jQuery | Security + Modernization |
| [Enterprise Java API](examples/enterprise-java-api/) | Java 17 + Spring Boot + Kafka | Performance + Events |

---

## Works With

| Agent | Fit | Sub-Agents | Autonomy |
|-------|-----|------------|----------|
| Claude Code | Best | Native (Task tool) | Full autonomous |
| Codex CLI | Great | No | Full autonomous |
| Cursor | Great | No | Semi-autonomous |
| Windsurf | Great | No | Semi-autonomous |
| Aider | Good | No | Full autonomous |
| Continue | Good | No | Interactive |
| OpenCode | Good | No | Full autonomous |
| Qwen Coder | Good | No | Full autonomous |

See [guides/tool-guide.md](guides/tool-guide.md) for detailed setup and per-tool configuration.

---

## How Agent Dispatch Differs

**"How is this different from CrewAI / AutoGen / LangGraph?"**

Those are **runtime frameworks** — they run code, manage agent sessions, automate workflows. Agent Dispatch is a **planning methodology** — it defines what agents should work on, in what order, with what boundaries.

| Aspect | Agent Dispatch | Runtime Frameworks |
|--------|---------------|-------------------|
| **What it is** | Methodology + templates (markdown) | Software (TypeScript/Python) |
| **Execution traces** | Core concept | Not addressed |
| **Chain execution** | One trace-fix-verify at a time | Task queues |
| **Territory isolation** | File-level ownership per agent | Not addressed |
| **Wave dependencies** | Dependency-aware dispatch order | Basic task deps |
| **Adversarial review** | RED TEAM reviews all branches | Not addressed |
| **AI-assisted planning** | Sprint Planner reads codebase, proposes plan | Manual task definition |
| **Merge strategy** | Dependency-ordered with validation | Not addressed |
| **Scaling** | Nested teams, 30+ agents | Session scaling |
| **Install** | Copy markdown into repo | `npm install` / `pip install` |

**They're complementary.** Use Agent Dispatch to plan the sprint. Use a runtime framework to automate the dispatch loop. The planning layer makes the automation layer effective.

---

## Stack Compatibility

Stack-agnostic. Works with anything that uses git:

| Stack | Application Type |
|-------|-----------------|
| **Go** (Chi, Gin, Echo, Fiber) | API servers, microservices, CLI tools |
| **TypeScript / Node.js** (Next.js, SvelteKit, Express, NestJS) | Full-stack web apps, APIs, serverless |
| **Python** (Django, FastAPI, Flask) | Web apps, ML pipelines, data services |
| **Rust** (Actix, Axum, Rocket) | Systems programming, high-performance APIs |
| **Elixir / Erlang** (Phoenix, LiveView) | Real-time apps, distributed systems |
| **Java / Kotlin** (Spring Boot, Ktor) | Enterprise APIs, event-driven systems |
| **C / C++** (FreeRTOS, STM32, CMake) | Embedded firmware, IoT, systems |
| **PHP** (Laravel, Symfony, legacy) | Web apps, monoliths, CMS |
| **Ruby** (Rails, Hanami) | Web applications |
| **C# / .NET** (ASP.NET Core, Blazor) | Enterprise web apps, microservices |

Monoliths, microservices, monorepos, full-stack apps, backend-only APIs, frontend SPAs, ML pipelines, embedded firmware, legacy codebases, infrastructure-as-code.

For legacy codebases with no tests, no docs, and spaghetti code, see [guides/legacy-codebases.md](guides/legacy-codebases.md).

---

## Origin

Created by Roberto H. Luna for the MIOSA platform projects. Battle-tested across production codebases spanning Go, TypeScript, SvelteKit, Python, Elixir, Java, C/C++, Rust, and PHP — from embedded firmware and legacy monoliths to AI content platforms and multi-tenant SaaS orchestrators.

## License

MIT — Use it, modify it, ship it.
