# Scaling Agent Dispatch

> How to grow beyond the standard roster — when to scale, how to split roles, and how to keep it from becoming chaos

---

## Scaling Philosophy

The 9 roles in Agent Dispatch are a **starting template**, not a limit. They cover most sprints cleanly: backend, frontend, infra, services, tests, data, design, adversarial review, and orchestration. If your sprint fits in those buckets, use them as-is.

Scale when the work demands it — not before.

The decision to add agents is a decision about chain independence. If you have 40 backend chains across 3 distinct subsystems (API layer, async workers, middleware), a single BACKEND agent will serialize work that could run in parallel. That's where splitting pays off. But if those 40 chains share files and call each other's functions, splitting them creates more coordination overhead than you save in parallelism.

**The rule:** Split roles when chains are genuinely independent. More agents with overlapping territories means merge conflicts in every wave, not faster delivery.

Three focused agents that each own their chains beat ten agents stepping on each other every time. Start with the standard roster. Add only when a single agent clearly has more independent chains than it can complete in a reasonable sprint.

---

## Team Size Guide

| Size | Agents | When to Use | Example |
|------|--------|-------------|---------|
| Solo | 1 | Single bug fix, simple feature, single file | Fix a typo in a handler |
| Small | 2-4 | Focused sprint, 1-2 waves | Backend bug fix + test coverage |
| Standard | 5-8 | Full sprint, 3+ waves | Cross-stack bug fix sprint |
| Large | 10-20 | Split domains, 4+ waves | Major refactor across subsystems |
| XL | 20-30+ | Nested teams, 5+ waves | Platform-wide migration |

### Solo (1 Agent)

One agent. One branch. No coordination overhead. Use this for isolated bugs, single-file changes, and anything that doesn't require multiple domains. The agent still follows chain execution — trace, fix, verify, document — but there's no wave planning or merge order to manage.

Don't reach for the full 8-agent setup when the work doesn't warrant it.

### Small (2-4 Agents)

2-3 waves at most. Typically one agent handles the core work and a second handles validation (QA) or infrastructure. LEAD may be unnecessary — the human operator can do a manual merge in minutes.

Good for: bug fix pairs (DATA + BACKEND), feature slices (BACKEND + FRONTEND), or audit sprints (QA + INFRA).

### Standard (5-8 Agents)

The default setup. All 9 roles available; pick the ones the sprint actually needs. 3-5 waves with a clear dependency graph. RED TEAM reviews all branches before LEAD merges.

Use the full roster only when all domains have meaningful work. Dispatching DESIGN when there's no design work, or RED TEAM when the sprint is low-risk, is wasteful.

### Large (10-20 Agents)

At this scale, individual roles split into focused variants. BACKEND becomes BACKEND-API + BACKEND-WORKERS. QA becomes QA-UNIT + QA-INTEGRATION + QA-SECURITY. Each split role gets its own territory subset and worktree.

Wave count grows to 4-5. The operator still talks directly to each agent — no nesting required unless individual chains are too large for one agent.

Expect more pre-sprint planning time. Writing execution traces for 30+ chains before dispatching saves significant steering time mid-sprint.

### XL (20-30+ Agents)

At XL scale, the operator can no longer directly manage every agent without a coordination layer. The solution is nested teams: each of the core roles becomes a **team lead** that manages 2-5 sub-agents internally. The operator talks to team leads. Team leads manage sub-agents.

5-8 waves. The operator's monitoring limit per wave is 6-8 slots — but at XL scale, a "slot" is an entire team, not a single agent.

See [Nested Team Architecture](#nested-team-architecture) below.

---

## Nested Team Architecture

At XL scale, the flat agent structure becomes a two-level hierarchy. Each of the original roles becomes a team lead responsible for a domain. The operator dispatches team leads. Team leads spawn and coordinate sub-agents internally.

```
OPERATOR
├── BACKEND (team lead)
│   ├── backend-handlers (sub-agent)
│   ├── backend-services (sub-agent)
│   └── backend-middleware (sub-agent)
├── FRONTEND (team lead)
│   ├── frontend-routes (sub-agent)
│   ├── frontend-components (sub-agent)
│   └── frontend-stores (sub-agent)
├── QA (team lead)
│   ├── qa-unit (sub-agent)
│   ├── qa-integration (sub-agent)
│   └── qa-security (sub-agent)
├── DATA (team lead)
│   ├── data-models (sub-agent)
│   └── data-migrations (sub-agent)
├── SERVICES (team lead)
│   ├── services-integrations (sub-agent)
│   └── services-workers (sub-agent)
├── DESIGN (team lead)
│   ├── design-tokens (sub-agent)
│   └── design-a11y (sub-agent)
├── INFRA (team lead)
│   └── infra-ci (sub-agent)   ← Single sub-agent if work is smaller
└── LEAD (orchestrator)
    └── Merges team-lead branches only
```

The operator's activation prompt goes to each team lead. The team lead reads the full chain assignment, decides which chains to delegate to sub-agents, and synthesizes their output before reporting to the operator.

From the operator's perspective: 8 slots in the monitoring view, same as a standard sprint. Internally, each slot has 2-5 sub-agents working in parallel.

From LEAD's perspective: 8 branches to merge, same merge order as always. LEAD does not see or merge sub-agent branches — team leads pre-merge those internally before they reach LEAD.

---

## Sub-Agent Spawning Rules

These rules exist to prevent coordination from becoming the bottleneck.

**Sub-agents inherit the parent's territory.** A BACKEND team lead owns `internal/handler/, internal/service/, internal/middleware/`. A backend-handlers sub-agent can only write to `internal/handler/`. Sub-agents cannot expand beyond what the team lead was assigned.

**Sub-agents get single chains, not multi-chain assignments.** Give each sub-agent one execution trace with a clear fix site and verification path. Multi-chain assignments belong to team leads.

**Parent validates combined output before reporting complete.** The team lead must verify that all sub-agent changes compile together, pass tests, and don't conflict with each other before writing the completion report to the operator.

**Sub-agents don't spawn their own sub-agents.** Maximum depth is 2 levels: operator → team lead → sub-agent. Deeper nesting introduces more coordination overhead than the work saves.

**Sub-agents use the parent's branch.** No separate worktrees within a team. All sub-agent work commits to the team lead's branch (`sprint-01/backend`). The team lead is responsible for keeping that branch coherent.

---

## Role Splitting Patterns

### BACKEND → Split When...

15+ backend chains span 3+ subsystems (API handlers, background workers, middleware stack). A single BACKEND agent serializes work that has no shared files.

**Split into:**
- `BACKEND-API` — HTTP handlers, routing, request/response serialization
- `BACKEND-WORKERS` — background jobs, queue consumers, scheduled tasks
- `BACKEND-MIDDLEWARE` — auth middleware, rate limiting, request logging, error recovery

Each split role gets a non-overlapping territory subset and its own worktree:
```
BACKEND-API:        internal/handler/, cmd/server/
BACKEND-WORKERS:    internal/worker/, cmd/worker/
BACKEND-MIDDLEWARE: internal/middleware/
```

Merge order within BACKEND: BACKEND-MIDDLEWARE → BACKEND-API → BACKEND-WORKERS → main (BACKEND slot).

### FRONTEND → Split When...

Routes, components, and stores have independent chain clusters with no shared files between them. If every frontend chain touches components AND stores, don't split — you'll just create merge conflicts.

**Split into:**
- `FRONTEND-ROUTES` — page-level route components, navigation, layouts
- `FRONTEND-COMPONENTS` — reusable UI components, feature components
- `FRONTEND-STORES` — state management, data fetching hooks, reactive stores

If the sprint includes significant design system implementation, consider `FRONTEND-DESIGN` instead of `FRONTEND-STORES` to keep design-token-consuming code isolated during the initial merge.

### QA → Split When...

Test infrastructure, unit tests, integration tests, and security audit are all significant bodies of work in the same sprint — typically 20+ test files to write or a full security audit alongside test coverage expansion.

**Split into:**
- `QA-UNIT` — unit tests for critical business logic
- `QA-INTEGRATION` — integration tests for API endpoints and service boundaries
- `QA-SECURITY` — OWASP audit, dependency scanning, auth flow validation
- `QA-PERFORMANCE` — load test scripts, benchmarks, profiling hooks

QA sub-agents are read-only on application code. They only write to test directories. This makes splitting QA lower-risk than splitting application roles — territory conflicts between QA sub-agents are rare.

### DATA → Split When...

Active migrations, model changes, and query optimization are all in the sprint simultaneously, AND they touch non-overlapping models/tables.

**Split into:**
- `DATA-MODELS` — schema definitions, model validation, relationships
- `DATA-MIGRATIONS` — migration scripts, data transformation, rollback paths

Never run DATA-MODELS and DATA-MIGRATIONS in the same wave — migrations depend on models. DATA-MODELS runs first, DATA-MIGRATIONS second.

### Adding Domain Roles

When a domain (ML, payments, search, messaging) has 5+ chains that are genuinely independent from the existing SERVICES territory, carve it out as its own role:

- `ML-TRAINING` — training pipelines, feature engineering, model artifacts
- `ML-INFERENCE` — inference endpoints, model serving, embedding generation
- `PAYMENT` — payment provider integrations, webhook handling, billing logic
- `SEARCH` — search indexing, query building, relevance tuning

Domain roles get a territory carved from an existing role (usually SERVICES) and their own worktree. Update the merge order to slot the new role in the appropriate position based on what it depends on.

---

## Wave Coordination at Scale

With 20+ agents, wave count grows and per-wave agent counts require active management.

**Keep each wave to 6-8 parallel slots.** That's the practical limit for a human operator monitoring progress and ready to intervene. Beyond 8, you lose situational awareness — agents can drift off-territory or hit blockers without you noticing.

At XL scale, a "slot" is a team lead, not an individual agent. The team lead manages their sub-agents internally. From the operator's perspective, Wave 1 still has ~6 slots — it just has 3-5 sub-agents running inside each slot.

| Agents | Waves | Max Per Wave | Notes |
|--------|-------|-------------|-------|
| 3-5 | 2-3 | 3 | Standard; operator can watch every terminal |
| 6-8 | 3-4 | 4 | Full sprint; manageable with completion report discipline |
| 10-15 | 4-5 | 5-6 | Split domains; consider nested teams for largest roles |
| 16-20 | 5-6 | 6-8 | Human monitoring limit per wave; nested teams recommended |
| 20-30+ | 6-8 | 6-8 | Nested teams only; team leads count as 1 slot each |

**Don't front-load Wave 1 with everything that has no dependencies.** If 12 things have no inter-dependencies, split them across Wave 1 and Wave 2 — not because of technical dependencies, but because 12 simultaneous agents is unmonitorable. Defer lower-priority chains to Wave 2.

**Sub-agents within a team run as one slot.** The operator dispatches the team lead. The team lead decides internal wave structure for sub-agents. Don't try to coordinate sub-agent timing from the operator level.

---

## Merge Strategy at Scale

Standard sprint: 8 branches merge to main in dependency order. Simple.

Large sprint with split roles: 12-16 branches need to merge. Still manageable if split roles pre-merge before their slot in the main merge order.

XL sprint with nested teams: potentially 30+ branches. This only works if team leads pre-merge internally first.

**The rule:** Only team-lead branches merge to main. Sub-agent branches merge to the team lead's branch first.

```
Sub-agents → Team lead branch → Main

  backend-handlers  ─┐
  backend-services   ─┤→ sprint-01/backend ──→ main
  backend-middleware ─┘

  frontend-routes     ─┐
  frontend-components  ─┤→ sprint-01/frontend ─→ main
  frontend-stores      ─┘
```

LEAD merges `sprint-01/backend` and `sprint-01/frontend` — not the 6 individual sub-agent branches. This keeps the merge surface at 8 branches regardless of how many sub-agents ran internally.

**Team lead pre-merge steps:**

```bash
# BACKEND team lead: merge sub-agent branches into team lead branch
cd /path/to/your-project-backend

git merge sprint-01/backend-middleware --no-ff
go build ./... && go test ./...   # Validate

git merge sprint-01/backend-handlers --no-ff
go build ./... && go test ./...   # Validate

git merge sprint-01/backend-services --no-ff
go build ./... && go test ./...   # Validate

# Write completion report
# Signal to operator: sprint-01/backend is ready to merge to main
```

LEAD then merges `sprint-01/backend` to main as a single, pre-validated branch — same as a standard sprint.

**Earlier sub-agent merge wins conflicts.** Within a team, the merge order follows the same dependency logic as the main merge order. BACKEND-MIDDLEWARE merges before BACKEND-API because handlers depend on middleware.

---

## When NOT to Scale

More agents is not faster. Each agent adds:

- A worktree to set up and tear down
- An activation prompt to write
- A completion report to read and validate
- A branch to merge and conflict-resolve
- A terminal to monitor

At some point the coordination cost exceeds the parallelism benefit. That point arrives earlier than most people expect.

**3 focused agents beat 10 unfocused agents every time.** The gains from splitting come from genuine chain independence — parallel execution traces that never touch the same files. If agents keep touching the same files, you have a territory design problem, not an agent count problem.

**Scale when chains are genuinely independent, not when you want to feel productive.** Dispatching 10 agents to fix 10 bugs that all trace back to the same root cause file is theater. One DATA agent fixing the root cause, followed by BACKEND cleaning up the call sites, is the right answer.

**Sub-agents add coordination tax.** Team leads spend real time validating sub-agent output, resolving internal conflicts, and synthesizing completion reports. Only use nested teams when a team lead's chain assignment is too large for a single agent in a single sprint.

### Warning Signs You Over-Scaled

- Merge conflicts in every wave, across agents that shouldn't share territory
- Team leads spending more time coordinating than working on chains
- Sub-agents sitting idle waiting for the team lead to clarify instructions
- The same file modified by 3+ agents in the same wave
- Completion reports full of "blocked by [other agent]" entries
- Wave 1 finishes, but Wave 2 can't start because Wave 1 conflicts haven't resolved

When you see these signals: stop, collapse to fewer agents, replan. It is faster to restart with 4 focused agents than to untangle 12 conflicting agents mid-sprint.

---

## Related Documents

- [agents/](../agents/) — The 9 base roles: territories, responsibilities, merge order
- [OPERATORS-GUIDE.md](../guides/operators-guide.md) — How to run sprints, wave dispatch, monitoring
- [METHODOLOGY.md](../core/methodology.md) — Execution traces, chain execution, priority levels
- [WORKFLOW.md](../core/workflow.md) — Technical workflow: worktrees, merges, validation
