# System Architecture

> How every file connects. What produces what. The complete flow from sprint idea to shipped code.

---

## The Flow

```
OPERATOR                    ORCHESTRATOR (Agent O)           AGENTS (AI × N)
   │                             │                               │
   │  "Fix auth bugs,            │                               │
   │   add rate limiting"        │                               │
   │ ───────────────────────►    │                               │
   │                             │                               │
   │                  ┌──────────┴──────────────┐                │
   │                  │ WAVE 0 — PLANNING:      │                │
   │                  │ 1. Reads codebase       │                │
   │                  │ 2. Analyzes architecture │                │
   │                  │ 3. Discovers work        │                │
   │                  │ 4. Writes exec traces    │                │
   │                  │ 5. Maps territories      │                │
   │                  └──────────┬──────────────┘                │
   │                             │                               │
   │                  ┌──────────┴──────────────┐                │
   │                  │ PRODUCES:               │                │
   │                  │ • DISPATCH.md           │                │
   │                  │ • agent-X-*.md (×N)     │                │
   │                  │ • activation prompts    │                │
   │                  └──────────┬──────────────┘                │
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
   │                             │  WAVES 1-3:              │ Each agent│
   │                             │  Monitors progress,      │ works on  │
   │                             │  intervenes when         │ its branch│
   │                             │  agents drift            │ in its    │
   │                             │  off-mission             │ territory │
   │                             │ ◄────────────────────────│           │
   │                             │                          │ Produces: │
   │                             │                          │ • Code    │
   │                             │                          │ • Tests   │
   │                             │                          │ • Complet-│
   │                             │                          │   ion doc │
   │                             │                          └────┬──────┘
   │                             │                               │
   │                  ┌──────────┴──────────────┐                │
   │                  │ WAVE 4: RED TEAM reviews │                │
   │                  │ WAVE 5: LEAD merges      │                │
   │                  └──────────┬──────────────┘                │
   │                             │                               │
   │                  ┌──────────┴──────────────┐                │
   │                  │ WAVE 6 — ASSESSMENT:    │                │
   │                  │ • Grades each agent     │                │
   │                  │ • Mission alignment     │                │
   │                  │ • SPRINT-ASSESSMENT.md  │                │
   │                  │ • Carry-forward work    │                │
   │                  └──────────┬──────────────┘                │
   │                             │                               │
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
                   │  ORCHESTRATOR (Agent O)  │ ◄── agents/orchestrator.md
                   │  reads:                  │
                   │  • guides/sprint-planner │
                   │  • agents/README         │
                   │  • config/dispatch-styles│
                   │  • config/code-standards │
                   │  • Your codebase         │
                   └─────────────┬───────────┘
                                 │ produces (Wave 0)
            ┌────────────────────┼────────────────────┐
            ▼                    ▼                     ▼
   ┌────────────────┐  ┌────────────────┐   ┌────────────────┐
   │ DISPATCH.md    │  │ agent-X-*.md   │   │ activation     │
   │ (sprint plan,  │  │ (per-agent     │   │ prompts        │
   │  waves, merge  │  │  task docs)    │   │ (copy-paste    │
   │  order)        │  │                │   │  into terminals│
   └───────┬────────┘  └───────┬────────┘   └───────┬────────┘
           │                   │                     │
           └───────────────────┼─────────────────────┘
                               │ pasted into terminals
                               ▼
                   ┌───────────────────────┐
                   │ EACH AGENT READS:     │
                   │                       │
                   │ 1. Project context    │
                   │ 2. Its task doc       │
                   │ 3. code-standards     │
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
                               │ read by RED TEAM, ORCHESTRATOR, LEAD
                               ▼
                   ┌───────────────────────┐
                   │ MERGE + ASSESS        │
                   │                       │
                   │ ORCHESTRATOR reads:    │
                   │ • All completion docs  │
                   │ • RED TEAM findings    │
                   │ • Grades each agent    │
                   │ • Produces SPRINT-     │
                   │   ASSESSMENT.md        │
                   │                       │
                   │ LEAD reads:            │
                   │ • ORCHESTRATOR grade   │
                   │ • RED TEAM findings    │ ◄── templates/red-team-findings
                   │ • Merges in order      │
                   │ • Build+test each      │
                   │ • Ships or blocks      │
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
| `agents/README.md` | ORCHESTRATOR | Roster, waves, merge order |
| `agents/orchestrator.md` | ORCHESTRATOR | Sprint command: planning, monitoring, grading |
| `agents/[role].md` | ORCHESTRATOR + that agent | Territory, responsibilities, merge position |

### Guides — How-To

| File | Read By | Job |
|------|---------|-----|
| `guides/orchestrator-playbook.md` | ORCHESTRATOR | **THE PLAYBOOK** — single sequential doc with hard checkpoints |
| `guides/carry-forward-protocol.md` | ORCHESTRATOR | How deferred work bridges sprints |
| `guides/agent-briefing.md` | Any AI first encountering this system | "What is this. What do I do." |
| `guides/sprint-planner.md` | ORCHESTRATOR | 6-phase methodology for analyzing codebase → sprint plan |
| `guides/parallel-execution.md` | Operator | Terminal scaling model, sub-agent math |
| `guides/operators-guide.md` | Operator | Full tutorial, start to finish |
| `guides/quickstart.md` | Operator | 5-minute version |
| `guides/customization.md` | ORCHESTRATOR + operator | Adapt territories per stack |
| `guides/tool-guide.md` | Operator | AI tool comparison + setup |
| `guides/legacy-codebases.md` | ORCHESTRATOR | How to handle codebases with no tests/docs |
| `guides/dispatch-config.md` | Automation scripts | Machine-readable YAML config |

### Templates — Copy-Paste Starting Points

| File | Produces | Job |
|------|----------|-----|
| `templates/dispatcher-prompt.md` | ORCHESTRATOR activation | Meta-prompt to kick off sprint |
| `templates/dispatch.md` | `sprint-XX/DISPATCH.md` | Sprint plan structure |
| `templates/agent.md` | `sprint-XX/agent-X-*.md` | Per-agent task doc structure |
| `templates/activation.md` | Activation prompts | Prompt structure + dispatch pipeline |
| `templates/completion.md` | `agent-X-completion.md` | Agent completion report structure |
| `templates/sprint-assessment.md` | `SPRINT-ASSESSMENT.md` | ORCHESTRATOR grading output |
| `templates/sprint-registry.md` | `REGISTRY.md` | Sprint index + carry-forward tracker |
| `templates/status.md` | Status board | Sprint tracking during execution |
| `templates/red-team-findings.md` | RED TEAM report | Adversarial findings format |
| `templates/preflight-checklist.md` | Pre-dispatch gate | Verify before pasting prompts |
| `templates/post-sprint-checklist.md` | Post-sprint gate | Teardown + carry-forward |
| `templates/retrospective.md` | Sprint retro | Post-sprint review |

### Runtime — While Agents Work

| File | Read By | Job |
|------|---------|-----|
| `runtime/pipeline-gates.md` | ORCHESTRATOR + operator | 6 hard gates: plan → dispatch → waves → merge → close |
| `runtime/retry-protocol.md` | Agents + ORCHESTRATOR | Escalating 3-tier retries with per-chain verification |
| `runtime/status-tracking.md` | ORCHESTRATOR + operator | Agent states, health indicators |
| `runtime/reactions.md` | ORCHESTRATOR + operator | 12 decision trees for runtime events |
| `runtime/interventions.md` | ORCHESTRATOR + operator | 24 copy-paste correction messages |

### Scaling — Beyond 10 Agents

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
    ├── DISPATCH.md                  # Sprint plan (generated by ORCHESTRATOR)
    ├── codebase-analysis.md         # Codebase snapshot (generated by ORCHESTRATOR)
    ├── DISPATCH-PROMPTS.md          # All activation prompts (archive)
    │
    ├── agent-A-backend.md           # Task docs (generated by ORCHESTRATOR)
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
    ├── SPRINT-SUMMARY.md            # LEAD's final summary + ship decision
    └── SPRINT-ASSESSMENT.md         # ORCHESTRATOR's grading + mission assessment
```

---

## Execution Timeline

```
TIME ──────────────────────────────────────────────────────────────────────────►

WAVE 0         WAVE 1              WAVE 2           WAVE 3      WAVE 4    WAVE 5    WAVE 6
│              │                   │                │           │         │         │
│ ORCHESTRATOR │ DATA ████████░    │ BACKEND ██████ │ FRONTEND  │ RED     │ LEAD    │ ORCHESTRATOR
│ analyzes     │ QA   ████████░    │ SERVICES █████ │ ████████░ │ TEAM    │ merges  │ grades
│ codebase,    │ INFRA ███████░    │                │           │ ██████░ │ ships   │ assesses
│ generates    │ DESIGN ██████░    │                │           │         │         │ closes
│ all docs     │                   │                │           │         │         │ sprint
│              │ All Wave 1 done   │ All Wave 2     │ Wave 3    │ Review  │ Merged  │
│              │ before Wave 2     │ done before    │ done      │ done    │         │ Done
│              │ starts            │ Wave 3 starts  │           │         │         │
```

Each `████` block = one terminal with one agent + its sub-agents running in parallel.

---

## Reading Order for New Users

1. `guides/agent-briefing.md` — What is this system
2. `guides/quickstart.md` — 5-minute overview
3. `core/architecture.md` — This file. How everything connects.
4. `guides/operators-guide.md` — Full tutorial (when ready to run a sprint)

## Reading Order for ORCHESTRATOR

1. `agents/orchestrator.md` — Your role, grading rubric, assessment template
2. `guides/sprint-planner.md` — 6-phase planning methodology
3. `agents/README.md` — Roles, waves, merge order
4. `config/dispatch-styles.md` — Style blocks for prompts
5. `config/code-standards.md` — How agents write code
6. `templates/` — All templates for generating docs

## Reading Order for Activated Agents

1. Your activation prompt (given to you)
2. Your task doc (`agent-X-*.md`)
3. `config/code-standards.md` — How to write code
4. `templates/completion.md` — What you produce when done
