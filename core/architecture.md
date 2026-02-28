# System Architecture

> How every file connects. What produces what. The complete flow from sprint idea to shipped code.

---

## The Flow

```
OPERATOR                    DISPATCHER (AI)                 AGENTS (AI × N)
   │                             │                               │
   │  "Fix auth bugs,            │                               │
   │   add rate limiting"        │                               │
   │ ───────────────────────►    │                               │
   │                             │                               │
   │                        ┌────┴────────────────┐              │
   │                        │ READS:              │              │
   │                        │ 1. sprint-planner.md│              │
   │                        │ 2. Your codebase    │              │
   │                        │ 3. agents/README.md │              │
   │                        │ 4. dispatch-styles  │              │
   │                        │ 5. code-standards   │              │
   │                        └────┬────────────────┘              │
   │                             │                               │
   │                        ┌────┴────────────────┐              │
   │                        │ PRODUCES:           │              │
   │                        │ • DISPATCH.md       │              │
   │                        │ • agent-X-*.md (×N) │              │
   │                        │ • activation prompts│              │
   │                        └────┬────────────────┘              │
   │                             │                               │
   │  Reviews proposal           │                               │
   │ ◄───────────────────────────│                               │
   │  Approves                   │                               │
   │ ───────────────────────►    │                               │
   │                             │                               │
   │  Sets up worktrees          │                               │
   │  Opens terminals            │                               │
   │  Pastes prompts ──────────────────────────────────────►     │
   │                             │                          ┌────┴──────┐
   │                             │                          │ Each agent│
   │                             │                          │ works on  │
   │                             │                          │ its branch│
   │                             │                          │ in its    │
   │                             │                          │ territory │
   │  Monitors progress ◄──────────────────────────────────│           │
   │                             │                          │ Produces: │
   │                             │                          │ • Code    │
   │                             │                          │ • Tests   │
   │                             │                          │ • Complet-│
   │                             │                          │   ion doc │
   │                             │                          └────┬──────┘
   │                             │                               │
   │  LEAD merges in order       │                               │
   │  Build+test after each      │                               │
   │  Ship ✓                     │                               │
```

---

## File Dependency Map

What reads what. Follow the arrows.

```
                        ┌──────────────────┐
                        │  OPERATOR GIVES  │
                        │  sprint goal +   │
                        │  codebase access │
                        └────────┬─────────┘
                                 │
                                 ▼
                   ┌─────────────────────────┐
                   │  guides/sprint-planner   │ ◄── The dispatcher's methodology
                   └─────────────┬───────────┘
                                 │ reads
            ┌────────────────────┼────────────────────┐
            ▼                    ▼                     ▼
   ┌────────────────┐  ┌────────────────┐   ┌────────────────┐
   │ agents/README  │  │ config/dispatch │   │ config/code    │
   │ (roles, waves, │  │ -styles        │   │ -standards     │
   │  merge order)  │  │ (tone, quality │   │ (how agents    │
   │                │  │  protocol)     │   │  write code)   │
   └───────┬────────┘  └───────┬────────┘   └───────┬────────┘
           │                   │                     │
           │         ┌────────┘                     │
           ▼         ▼                               │
   ┌───────────────────────┐                        │
   │ DISPATCHER PRODUCES:  │                        │
   │                       │                        │
   │ sprint-XX/DISPATCH.md │ ◄── templates/dispatch │
   │ sprint-XX/agent-X-*.md│ ◄── templates/agent    │
   │ activation prompts    │ ◄── templates/activation│
   └───────────┬───────────┘                        │
               │ pasted into terminals               │
               ▼                                     │
   ┌───────────────────────┐                        │
   │ EACH AGENT READS:     │                        │
   │                       │                        │
   │ 1. Project context    │                        │
   │ 2. Its task doc       │                        │
   │ 3. code-standards ◄───┼────────────────────────┘
   │ 4. Source files       │
   │ 5. Its activation     │
   │    prompt (inline)    │
   └───────────┬───────────┘
               │ works, then produces
               ▼
   ┌───────────────────────┐
   │ AGENT PRODUCES:       │
   │                       │
   │ • Code changes        │
   │ • agent-X-completion  │ ◄── templates/completion
   │   .md                 │
   └───────────┬───────────┘
               │ read by RED TEAM + LEAD
               ▼
   ┌───────────────────────┐
   │ MERGE + SHIP          │
   │                       │
   │ LEAD reads:           │
   │ • All completion docs │
   │ • RED TEAM findings   │ ◄── templates/red-team-findings
   │ • Merges in order     │
   │ • Build+test each     │
   │ • Ships or blocks     │
   └───────────────────────┘
```

---

## Document Roles

Every file in the repo has exactly one job.

### Config — Controls Agent Behavior

| File | Read By | Job |
|------|---------|-----|
| `config/dispatch-styles.md` | Dispatcher | Tone blocks appended to every activation prompt |
| `config/code-standards.md` | Every agent | How to write code. Shortest correct implementation. |

### Core — Theory

| File | Read By | Job |
|------|---------|-----|
| `core/methodology.md` | Dispatcher + agents | Execution traces, chain protocol, priority levels |
| `core/workflow.md` | Operator | Sprint lifecycle: plan → dispatch → merge → ship |
| `core/anti-patterns.md` | Dispatcher | What not to do (12 patterns) |
| `core/architecture.md` | Anyone | This file. How everything connects. |

### Agents — Role Definitions

| File | Read By | Job |
|------|---------|-----|
| `agents/README.md` | Dispatcher | Roster, waves, merge order |
| `agents/[role].md` | Dispatcher + that agent | Territory, responsibilities, merge position |

### Guides — How-To

| File | Read By | Job |
|------|---------|-----|
| `guides/agent-briefing.md` | Any AI first encountering this system | "What is this. What do I do." |
| `guides/sprint-planner.md` | Dispatcher | 6-phase methodology for analyzing codebase → sprint plan |
| `guides/parallel-execution.md` | Operator | Terminal scaling model, sub-agent math |
| `guides/operators-guide.md` | Operator | Full tutorial, start to finish |
| `guides/quickstart.md` | Operator | 5-minute version |
| `guides/customization.md` | Dispatcher + operator | Adapt territories per stack |
| `guides/tool-guide.md` | Operator | AI tool comparison + setup |
| `guides/legacy-codebases.md` | Dispatcher | How to handle codebases with no tests/docs |
| `guides/dispatch-config.md` | Automation scripts | Machine-readable YAML config |

### Templates — Copy-Paste Starting Points

| File | Produces | Job |
|------|----------|-----|
| `templates/dispatch.md` | `sprint-XX/DISPATCH.md` | Sprint plan structure |
| `templates/agent.md` | `sprint-XX/agent-X-*.md` | Per-agent task doc structure |
| `templates/activation.md` | Activation prompts | Prompt structure + dispatch pipeline |
| `templates/completion.md` | `agent-X-completion.md` | Agent completion report structure |
| `templates/status.md` | Status board | Sprint tracking during execution |
| `templates/red-team-findings.md` | RED TEAM report | Adversarial findings format |
| `templates/retrospective.md` | Sprint retro | Post-sprint review |

### Runtime — While Agents Work

| File | Read By | Job |
|------|---------|-----|
| `runtime/status-tracking.md` | Operator | Agent states, health indicators |
| `runtime/reactions.md` | Operator | 12 decision trees for runtime events |
| `runtime/interventions.md` | Operator | 24 copy-paste correction messages |

### Scaling — Beyond 9 Agents

| File | Read By | Job |
|------|---------|-----|
| `scaling/scaling.md` | Dispatcher + operator | Nested teams, 20-30+ agents |
| `scaling/multi-repo.md` | Operator | Multi-repo coordination |

### Examples — Reference Implementations

| Directory | Stack | Job |
|-----------|-------|-----|
| `examples/ecommerce-api/` | Go + Chi + Stripe | Complete dispatch example |
| `examples/saas-dashboard/` | Python + FastAPI + React | Complete dispatch example |
| `examples/realtime-chat/` | Elixir + Phoenix | Complete dispatch example |
| `examples/embedded-firmware/` | C/C++ + FreeRTOS | Complete dispatch example |
| `examples/legacy-php-webapp/` | PHP 7.4 + jQuery | Complete dispatch example |
| `examples/enterprise-java-api/` | Java 17 + Spring Boot | Complete dispatch example |

---

## Sprint Output Structure

What a completed sprint looks like on disk:

```
docs/agent-dispatch/sprints/
├── sprint-01/
├── sprint-02/
└── sprint-03/
    ├── DISPATCH.md                  # Sprint plan (generated by dispatcher)
    ├── DISPATCH-PROMPTS.md          # All activation prompts (archive)
    │
    ├── agent-A-backend.md           # Task docs (generated by dispatcher)
    ├── agent-B-frontend.md
    ├── agent-C-infra.md
    ├── agent-D-services.md
    ├── agent-E-qa.md
    ├── agent-F-data.md
    ├── agent-G-lead.md
    │
    ├── agent-A-completion.md        # Completion reports (written by agents)
    ├── agent-B-completion.md        # ─── or use NATO codenames: ───
    ├── agent-C-completion.md        # ALPHA-COMPLETION.md
    ├── agent-D-completion.md        # BRAVO-COMPLETION.md
    ├── agent-E-completion.md        # CHARLIE-COMPLETION.md
    ├── agent-F-completion.md        # DELTA-COMPLETION.md
    ├── agent-G-completion.md        # ECHO-COMPLETION.md
    │
    ├── red-team-findings.md         # RED TEAM adversarial report
    ├── SECURITY-REVIEW.md           # Security review (if applicable)
    └── SPRINT-SUMMARY.md            # LEAD's final summary + ship decision
```

---

## Execution Timeline

```
TIME ──────────────────────────────────────────────────────────────►

PLAN           WAVE 1              WAVE 2           WAVE 3      WAVE 4    WAVE 5
│              │                   │                │           │         │
│ Dispatcher   │ DATA ████████░    │ BACKEND ██████ │ FRONTEND  │ RED     │ LEAD
│ analyzes     │ QA   ████████░    │ SERVICES █████ │ ████████░ │ TEAM    │ merges
│ codebase,    │ INFRA ███████░    │                │           │ ██████░ │ ships
│ proposes     │ DESIGN ██████░    │                │           │         │
│ sprint       │                   │                │           │         │
│              │ All Wave 1 done   │ All Wave 2     │ Wave 3    │ Review  │ Done
│              │ before Wave 2     │ done before    │ done      │ done    │
│              │ starts            │ Wave 3 starts  │           │         │
```

Each `████` block = one terminal with one agent + its sub-agents running in parallel.

---

## Reading Order for New Users

1. `guides/agent-briefing.md` — What is this system
2. `guides/quickstart.md` — 5-minute overview
3. `core/architecture.md` — This file. How everything connects.
4. `guides/operators-guide.md` — Full tutorial (when ready to run a sprint)

## Reading Order for AI Dispatchers

1. `guides/sprint-planner.md` — Your methodology
2. `agents/README.md` — Roles, waves, merge order
3. `config/dispatch-styles.md` — Style blocks for prompts
4. `config/code-standards.md` — How agents write code
5. `templates/` — All templates for generating docs

## Reading Order for Activated Agents

1. Your activation prompt (given to you)
2. Your task doc (`agent-X-*.md`)
3. `config/code-standards.md` — How to write code
4. `templates/completion.md` — What you produce when done
