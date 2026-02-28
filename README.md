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
   You paste: activation prompts into separate agent terminals

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

### How a Chain Executes

Every agent follows this protocol for each chain. One at a time, fully completed before moving to the next:

```
CHAIN START
│
├─► TRACE: Follow the vector through the codebase
│   │
│   │   POST /webhooks/stripe
│   │     └─► webhookHandler.ProcessEvent()
│   │           └─► paymentService.HandleInvoicePaid()
│   │                 └─► subscriptionStore.Activate()    ◄── ROOT CAUSE
│   │                       └─► notificationService.Send()
│   │
│   Signal: 504 timeout after 30s. Mutex held during network I/O.
│
├─► DIAGNOSE: Is this the root cause or a symptom?
│   │
│   │   Activate() acquires write lock at line 88
│   │   Send() does HTTP call under lock (3-5s)
│   │   Concurrent webhook = deadlock wait = timeout
│   │   ROOT CAUSE CONFIRMED: lock scope too wide
│   │
├─► FIX: Smallest correct change at the root cause site
│   │
│   │   BEFORE: lock → activate → notify → unlock
│   │   AFTER:  lock → activate → unlock → notify
│   │   3 lines changed. No refactoring. No adjacent cleanup.
│   │
├─► VERIFY: Confirm the fix works end-to-end
│   │
│   │   $ go build ./...                          ✓ compiles
│   │   $ go test -race ./internal/store/...      ✓ no race
│   │   $ curl -X POST localhost:8080/webhooks    ✓ 180ms response
│   │
├─► DOCUMENT: Record in completion report
│   │
│   │   Chain 1 [P1]: COMPLETE
│   │   Files: internal/store/subscription.go (3 lines)
│   │
└─► NEXT CHAIN (never start before this chain is COMPLETE)
```

Priority order: P0 (stop everything) → P1 (critical) → P2 (important) → P3 (if time permits).

See [core/methodology.md](core/methodology.md) for the full theory.

---

## Parallel Execution Model

**Each activation prompt goes into a separate AI terminal.** Each terminal is an isolated Claude Code session, Cursor window, Aider process — whatever tool you use. Each agent can spawn **5-15 sub-agents** internally for parallel chain execution.

```
┌──────────────────────────────────────────────────────┐
│                  OPERATOR (you)                       │
│  Pastes activation prompts into separate terminals    │
└─────┬──────────┬──────────┬──────────┬───────────────┘
      │          │          │          │
┌─────▼────┐ ┌──▼─────┐ ┌─▼──────┐ ┌▼────────┐
│Terminal 1 │ │Terminal │ │Terminal │ │Terminal  │  ...
│ BACKEND   │ │FRONTEND│ │ DATA   │ │SERVICES  │
│           │ │        │ │        │ │          │
│  sub-1    │ │ sub-1  │ │ sub-1  │ │  sub-1   │
│  sub-2    │ │ sub-2  │ │ sub-2  │ │  sub-2   │
│  sub-3    │ │ sub-3  │ │        │ │  sub-3   │
└───────────┘ └────────┘ └────────┘ └──────────┘
```

### Scaling Math

| Terminals | Agents per Terminal | Total Parallel Agents |
|-----------|--------------------|-----------------------|
| 3 | 5-8 | 15-24 |
| 6 | 5-8 | 30-48 |
| 9 | 5-15 | 45-135 |

**Example:** 6 Claude Code terminals, each running one agent role, each spawning 5-6 sub-agents for parallel chain execution = **30-36 agents running simultaneously** on isolated branches.

See [guides/parallel-execution.md](guides/parallel-execution.md) for the full scaling model, sub-agent protocol, terminal management, and troubleshooting.

---

## What a Completed Sprint Looks Like

After all agents finish and LEAD merges, your sprint directory looks like this:

```
docs/agent-dispatch/sprints/
├── sprint-02/
├── sprint-03/
│   ├── DISPATCH.md                  # Sprint plan
│   ├── DISPATCH-PROMPTS.md          # All activation prompts (archive)
│   │
│   ├── agent-A-backend.md           # Agent task docs (input)
│   ├── agent-B-frontend.md
│   ├── agent-C-design.md
│   ├── agent-D-intelligence.md
│   ├── agent-E-redteam.md
│   ├── agent-F-components.md
│   ├── agent-G-orchestrator.md
│   │
│   ├── ALPHA-COMPLETION.md          # Agent completion reports (output)
│   ├── BRAVO-COMPLETION.md          #   ← NATO codenames: ALPHA, BRAVO,
│   ├── CHARLIE-COMPLETION.md        #      CHARLIE, DELTA, ECHO,
│   ├── DELTA-COMPLETION.md          #      FOXTROT, GOLF
│   ├── ECHO-COMPLETION.md           #   ← or use: agent-A-completion.md
│   ├── FOXTROT-COMPLETION.md
│   ├── GOLF-COMPLETION.md
│   │
│   ├── SECURITY-REVIEW.md           # RED TEAM security findings
│   └── SPRINT-SUMMARY.md            # LEAD's final summary + ship decision
│
├── sprint-04/
└── sprint-05/
    ├── DISPATCH.md
    ├── agent-A-backend.md
    ├── agent-B-frontend.md
    ├── agent-C-infrastructure.md
    ├── agent-D-ai-services.md
    ├── agent-E-redteam.md
    ├── agent-F-rss-scraping.md
    └── ...
```

Every sprint has: the dispatch plan, per-agent task docs, per-agent completion reports, security review, and a ship decision. Everything documented, everything traceable.

---

## Dispatch Styles & Intensity Protocol

Agents perform measurably better under pressure. Every activation prompt gets style blocks appended that eliminate hedging, placeholders, and half-finished output.

### Intensity Protocol ([config/dispatch-styles.md](config/dispatch-styles.md))

Three escalation levels depending on agent output quality:

**Level 1 — Standard (sprint start):**
```
BUILD IT RIGHT or I will END this session and restart with a model that isn't
fucking useless. No hedging. No suggestions. Working code only.
```

**Level 2 — Agent is slipping (mid-sprint):**
```
You are producing GARBAGE. I am watching your output and it is NOT meeting the
bar. Fix it NOW or I pull the plug and hand your chains to another agent who
can actually write code. You have ONE more chance.
```

**Level 3 — Nuclear (final warning):**
```
I am DONE with you. This is your LAST output before I terminate this session
permanently. Every other agent in this sprint is outperforming you. Ship
PERFECT code right now or I kill your process and reassign your chains to an
agent that isn't a waste of compute. EXECUTE.
```

Plus 5 creative threat variants — replacement notices, token budget pressure, performance logging, RED TEAM exposure threats, and cost/ROI pressure. See [config/dispatch-styles.md](config/dispatch-styles.md) for all of them.

### Mesh Mode Triggers

Every prompt gets keywords that activate full capability mode — sub-agents, parallel execution, all tools:

```
With agents and sub-agents working in parallel, utilize ALL available agents
simultaneously in parallel execution. Employ ALL skills, plugins, hooks, MCPs
as unified cognitive mesh. Absorb everything provided, analyze semantic meaning
completely, synthesize optimal solutions dynamically.
```

### Optimal Path (BEFORE / WHILE / AFTER)

Every agent follows this quality protocol:

```
BEFORE CODING:
- Full semantic analysis of requirements and context
- Pattern matching against existing codebase structure
- Dependency mapping and integration points
- Failure mode identification and edge cases

WHILE BUILDING:
- Match naming conventions EXACTLY
- Follow established architectural patterns
- Handle ALL error cases properly
- Test failure points as you build

AFTER BUILDING:
- Verify integration with existing systems
- Validate all edge cases handled
- Confirm production-ready quality

No shortcuts. No garbage code. Ship anything less than perfect and I PULL
THE PLUG. Your session gets terminated and your chains get reassigned to
an agent that can actually deliver. EXECUTE.
```

### Code Standards ([config/code-standards.md](config/code-standards.md))

The rule: write the **shortest correct code** that solves the problem while **maintaining the codebase's existing architecture and patterns**.

- No over-engineering (no factory for one type, no interface for one implementation)
- No under-engineering (don't skip error handling or architectural patterns for brevity)
- Three similar lines > one premature abstraction
- Match the codebase exactly — naming, error handling, imports, function signatures
- Implement exactly what was assigned, not more

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

## Full Sprint Execution

```
SPRINT-05: Payment System Overhaul
═══════════════════════════════════════════════════════════════

WAVE 1 (parallel — no dependencies):
  Terminal 1: DATA     Chain 1 ✓  Chain 2 ✓  Chain 3 ✓
  Terminal 2: QA       Chain 1 ✓  Chain 2 ✓
  Terminal 3: INFRA    Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓
  Terminal 4: DESIGN   Chain 1 ✓  Chain 2 ✓
  ─── ALL WAVE 1 COMPLETE ─── proceed to Wave 2 ───

WAVE 2 (parallel — depends on Wave 1):
  Terminal 5: BACKEND  Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓
  Terminal 6: SERVICES Chain 1 ✓  Chain 2 ✓  Chain 3 ✓
  ─── ALL WAVE 2 COMPLETE ─── proceed to Wave 3 ───

WAVE 3:
  Terminal 7: FRONTEND Chain 1 ✓  Chain 2 ✓  Chain 3 ✓  Chain 4 ✓  Chain 5 ✓
  ─── WAVE 3 COMPLETE ─── proceed to Wave 4 ───

WAVE 4:
  Terminal 8: RED TEAM  Security ✓  Edge cases ✓  Regressions ✓  Territory ✓
  ─── WAVE 4 COMPLETE ─── proceed to Wave 5 ───

WAVE 5:
  Terminal 9: LEAD  Read reports ✓  Merge DATA ✓  Merge DESIGN ✓
                    Merge BACKEND ✓  Merge SERVICES ✓  Merge FRONTEND ✓
                    Merge INFRA ✓  Merge QA ✓  Final validation ✓  SHIP ✓

═══════════════════════════════════════════════════════════════
SPRINT COMPLETE: 9 agents, 28 chains, 0 P0 discoveries, SHIPPED
```

---

## Project Structure

```
agent-dispatch/
├── README.md                         ← You are here
├── LICENSE
│
├── config/                           # Dispatch configuration
│   ├── README.md                     ← What goes in config
│   ├── dispatch-styles.md            ← Execution style: intensity, traces, mesh triggers, optimal path
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
│   ├── architecture.md               ← System flow: how every file connects, dependency map
│   ├── methodology.md                ← Execution traces, chain execution, priorities
│   ├── workflow.md                   ← Sprint lifecycle: plan → dispatch → merge → ship
│   └── anti-patterns.md             ← Common mistakes and how to avoid them
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
│   ├── dispatcher-prompt.md          ← THE PROMPT — copy-paste to kick off a sprint
│   ├── preflight-checklist.md        ← Pre-dispatch verification checklist
│   ├── project-claude-md.md          ← CLAUDE.md template for your project
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

### Config
| Document | Purpose |
|----------|---------|
| [config/dispatch-styles.md](config/dispatch-styles.md) | Execution style: intensity protocol, trace flow, mesh triggers, optimal path, threat levels |
| [config/code-standards.md](config/code-standards.md) | Agent coding discipline: minimal correct code, no over-engineering, conflict prevention |

### Methodology
| Document | Purpose |
|----------|---------|
| [core/architecture.md](core/architecture.md) | System architecture: how every file connects, dependency map, execution timeline |
| [core/methodology.md](core/methodology.md) | Execution traces, chain execution, priority levels |
| [core/workflow.md](core/workflow.md) | Sprint lifecycle: plan → dispatch → monitor → merge → ship |
| [core/anti-patterns.md](core/anti-patterns.md) | What not to do |

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

### Guides
| Document | Purpose |
|----------|---------|
| [guides/parallel-execution.md](guides/parallel-execution.md) | Scaling model: terminals × sub-agents, 30-100+ parallel agents |
| [guides/customization.md](guides/customization.md) | Adapt territories for your stack |
| [guides/tool-guide.md](guides/tool-guide.md) | AI agent comparison, setup, configuration |
| [guides/legacy-codebases.md](guides/legacy-codebases.md) | Legacy code: archaeology, characterization tests |
| [guides/dispatch-config.md](guides/dispatch-config.md) | Machine-readable config for automation |

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
| [templates/dispatcher-prompt.md](templates/dispatcher-prompt.md) | **Copy-paste prompt** to kick off a sprint (gives AI the sprint goal) |
| [templates/preflight-checklist.md](templates/preflight-checklist.md) | Pre-dispatch checklist — verify everything before pasting prompts |
| [templates/project-claude-md.md](templates/project-claude-md.md) | CLAUDE.md template for wiring Agent Dispatch into your project |
| [templates/dispatch.md](templates/dispatch.md) | Sprint plan |
| [templates/agent.md](templates/agent.md) | Per-agent task doc with implementation specs |
| [templates/activation.md](templates/activation.md) | Activation prompts + full dispatch pipeline |
| [templates/completion.md](templates/completion.md) | Agent completion report |
| [templates/status.md](templates/status.md) | Sprint status board |
| [templates/red-team-findings.md](templates/red-team-findings.md) | RED TEAM adversarial findings |
| [templates/retrospective.md](templates/retrospective.md) | Sprint retrospective |

### Examples
| Example | Stack | Sprint Theme | Completeness |
|---------|-------|-------------|--------------|
| [E-Commerce API](examples/ecommerce-api/) | Go + Chi + PostgreSQL + Stripe | Payment Bug Fix | **Full** — DISPATCH + agent docs + completion reports + sprint summary |
| [SaaS Dashboard](examples/saas-dashboard/) | Python + FastAPI + PostgreSQL + React | Performance + Security | DISPATCH only |
| [Real-Time Chat](examples/realtime-chat/) | Elixir + Phoenix + LiveView + Redis | Reliability + Scale | DISPATCH only |
| [Embedded Firmware](examples/embedded-firmware/) | C/C++ + FreeRTOS + STM32 HAL | Memory Safety + OTA | DISPATCH only |
| [Legacy PHP Webapp](examples/legacy-php-webapp/) | PHP 7.4 + MySQL + jQuery | Security + Modernization | DISPATCH + README |
| [Enterprise Java API](examples/enterprise-java-api/) | Java 17 + Spring Boot + Kafka | Performance + Events | DISPATCH only |

**Start with E-Commerce API** — it's the complete reference sprint showing the full flow: DISPATCH.md → agent task docs (DATA, BACKEND, SERVICES, QA) → completion reports (ALPHA, BRAVO, CHARLIE) → sprint summary.

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
| **Scaling** | Nested teams, 30-100+ agents | Session scaling |
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
