# Sprint Planner — AI Onboarding Guide

> Hand this to your AI. It reads this doc + your codebase, then proposes a full sprint plan.
> This is the bridge between "I have a codebase" and "I have 9 agents working in parallel."

---

## What This Guide Does

You are an AI tasked with planning a multi-agent development sprint. Your job:

1. **Analyze** the codebase — understand the architecture, patterns, layers, tech stack
2. **Map** the codebase to agent territories — which agent owns which files
3. **Discover** work — bugs, security gaps, technical debt, missing tests, features to build
4. **Trace** each piece of work — entry point to root cause
5. **Propose** a sprint — agents, waves, chains, merge order, success criteria
6. **Generate** all sprint documents — DISPATCH.md, per-agent task docs, activation prompts

By the end, the operator has everything they need to set up worktrees and paste prompts into terminals. You do the thinking. They execute the dispatch.

---

## Phase 1: Codebase Analysis

Read the codebase systematically. Do not skim — you need deep understanding to write useful execution traces.

### Step 1.1: Read Project Context

Start with the top-level docs. These tell you the architecture, patterns, and commands:

```
Priority reading order:
1. CLAUDE.md or README.md or docs/CONTEXT.md — project overview
2. Package files — package.json, go.mod, Cargo.toml, requirements.txt (dependencies + scripts)
3. Config files — tsconfig, Makefile, Dockerfile, docker-compose, .env.example
4. CI config — .github/workflows/, .gitlab-ci.yml
5. Test config — vitest.config, jest.config, pytest.ini
```

Extract from these:
- **Tech stack**: language, framework, database, ORM, test framework, build tool
- **Build command**: how to compile/build the project
- **Test command**: how to run tests
- **Lint command**: how to lint
- **Dev command**: how to run locally
- **Project structure**: where code lives, how directories are organized

### Step 1.2: Map the Architecture

Read the directory structure and identify the **layers**. Every codebase has them, even if they're not clean:

```
Common layers (look for these):
├── Data layer      — models, schemas, migrations, stores, repositories, queries
├── Backend layer   — handlers, controllers, routes, middleware, services
├── Services layer  — integrations, workers, queues, external API clients
├── Frontend layer  — components, pages, routes, stores, hooks, utils
├── Infra layer     — Docker, CI/CD, deployment configs, build scripts
├── Test layer      — test files, fixtures, helpers, mocks
└── Design layer    — design tokens, theme files, style configs, storybook
```

For each layer, note:
- **Directory paths** (exact — these become territories)
- **Key files** (the high-traffic files where most logic lives)
- **Patterns used** (repository pattern, service pattern, middleware chains, hook patterns)
- **Dependencies** (which layers depend on which — data is usually the foundation)

### Step 1.3: Identify Conventions

Read 3-5 files per layer to extract the codebase's conventions. You will teach agents to match these EXACTLY:

- **Naming**: camelCase vs snake_case vs PascalCase — for files, functions, variables, types
- **Error handling**: how errors are created, wrapped, propagated, returned to callers
- **Import organization**: external first? grouped? aliased?
- **Function signatures**: (result, error) tuples? exceptions? Result types?
- **Test patterns**: describe/it blocks? table-driven? mocking strategy?
- **Comment style**: JSDoc? Go doc comments? sparse?
- **API response format**: { data, error }? { success, message }? HTTP status codes used?

Document these. They go into the DISPATCH.md and agent task docs so every agent matches the codebase style.

---

## Phase 2: Territory Mapping

Map the codebase directories to agent territories. Each file should have exactly ONE owner.

### Step 2.1: Default Territory Assignment

Start with this mapping and adapt to the actual directory structure:

| Agent | Owns | Pattern |
|-------|------|---------|
| **DATA** | Models, schemas, migrations, stores, repositories, query files | `models/`, `store/`, `db/`, `migrations/`, `prisma/` |
| **BACKEND** | HTTP handlers, controllers, routes, middleware, service layer | `handlers/`, `controllers/`, `routes/`, `services/`, `middleware/` |
| **SERVICES** | External integrations, workers, queues, specialized modules | `integrations/`, `workers/`, `modules/ai/`, `modules/payment/` |
| **FRONTEND** | Components, pages, routes, hooks, stores, utils | `components/`, `pages/`, `hooks/`, `stores/`, `lib/` |
| **INFRA** | Docker, CI/CD, build configs, deployment | `Dockerfile`, `docker-compose.yml`, `.github/`, `Makefile` |
| **QA** | Test files, test configs, fixtures, mocks | `*_test.*`, `*.test.*`, `*.spec.*`, `test/`, `__tests__/` |
| **DESIGN** | Design tokens, theme configs, style files, storybook | `design/`, `tokens/`, `theme/`, `styles/`, `.storybook/` |
| **RED TEAM** | No owned files. Read-only on all code. Writes tests + findings. | N/A |
| **LEAD** | Docs, README, CHANGELOG. Merge-only on code. | `docs/`, `README.md`, `CHANGELOG.md` |

### Step 2.2: Handle Overlaps

Some files have ambiguous ownership. Resolve them:

| Situation | Resolution |
|-----------|------------|
| `package.json` / `go.mod` | INFRA owns the file. Agents can ADD dependencies with justification in their completion report. LEAD resolves conflicts at merge. |
| Shared utils used by multiple layers | Whoever's layer they live in owns them. If `backend/src/shared/`, BACKEND owns it. |
| Config files read by both frontend and backend | INFRA owns config infrastructure. BACKEND/FRONTEND own their specific config values. |
| A file that doesn't fit any category | Assign it to the agent whose chains touch it. If no chains touch it, it stays unassigned. |

### Step 2.3: Write the Territory Map

Produce a clear map showing exact paths per agent. This goes into the DISPATCH.md:

```
AGENT TERRITORIES:
  DATA:     backend/src/models/, backend/src/store/, migrations/
  BACKEND:  backend/src/handlers/, backend/src/services/, backend/src/middleware/
  SERVICES: backend/src/modules/ai/, backend/src/workers/
  FRONTEND: frontend/src/routes/, frontend/src/components/, frontend/src/lib/
  INFRA:    Dockerfile, docker-compose.yml, .github/, Makefile
  QA:       **/*_test.go, **/*.test.ts (create only — read-only on source)
  DESIGN:   frontend/src/styles/, design-tokens/
  RED TEAM: read-only everywhere, writes to test files + findings report
  LEAD:     docs/, README.md
```

---

## Phase 3: Work Discovery

Now find the actual work. Use multiple discovery strategies — don't rely on just one.

### Strategy 3.1: Explicit Goals

Read the operator's sprint goal (the prompt they gave you). Extract:
- Specific bugs to fix
- Features to build
- Migrations to complete
- Debt to address

### Strategy 3.2: Code Quality Scan

Search for signals of problems:

```bash
# TODOs, FIXMEs, HACKs
grep -r "TODO\|FIXME\|HACK\|XXX\|WORKAROUND" src/ --include="*.{ts,go,py,rs}"

# Stub implementations
grep -r "return true\|return nil\|pass$\|throw new Error.*not implemented" src/

# Swallowed errors
grep -r "catch.*{}\|catch.*{$" src/ --include="*.ts"
grep -r "if err != nil {\s*$" src/ --include="*.go"  # empty error blocks

# Console/debug statements left in
grep -r "console\.log\|fmt\.Println\|print(" src/ --include="*.{ts,tsx,go,py}"

# Unused imports (check build warnings too)
# Dead code (functions never called)
```

### Strategy 3.3: Security Scan

Check for OWASP Top 10 signals:

```bash
# SQL injection (string concatenation in queries)
grep -r "SELECT.*\+\|INSERT.*\+\|UPDATE.*\+\|DELETE.*\+" src/ --include="*.{ts,go,py}"

# Missing auth on endpoints
# Look for route handlers without auth middleware/guards

# Hardcoded secrets
grep -r "password\|secret\|api_key\|token" src/ --include="*.{ts,go,py}" | grep -v test | grep -v node_modules

# XSS (dangerouslySetInnerHTML, unescaped user input in templates)
grep -r "dangerouslySetInnerHTML\|v-html\|{@html" src/ --include="*.{tsx,vue,svelte}"

# SSRF (unvalidated URLs in fetch/HTTP calls)
# Missing rate limiting on auth endpoints
# Missing CORS configuration
```

### Strategy 3.4: Test Coverage Gaps

```bash
# Run coverage report
# Go: go test -cover ./...
# Node: npx vitest run --coverage
# Python: pytest --cov

# Find untested files (files with no corresponding test file)
# Find high-risk code with zero coverage
# Find test files that are empty or have only one test
```

### Strategy 3.5: Performance Issues

```bash
# N+1 queries (loops with database calls inside)
# Missing database indexes (slow queries in logs)
# Excessive API calls from frontend (check network patterns)
# Missing pagination on list endpoints
# Missing caching on frequently accessed data
# Large bundle sizes (frontend)
```

### Strategy 3.6: Technical Debt

```bash
# Duplicate code (similar functions in multiple files)
# Outdated dependencies (npm outdated, go list -u -m all)
# Deprecated API usage
# Missing error handling on external calls
# Functions with too many parameters or too many lines
# Circular dependencies
```

### Strategy 3.7: Progress Tracker

If the project has a progress tracker, TODO list, or issue board — read it. It often contains prioritized work that the operator already knows about.

---

## Phase 4: Execution Trace Writing

For every piece of work discovered in Phase 3, write an execution trace. This is the critical step — traces are what make agents effective.

### How to Write a Trace

```
Chain: [descriptive title]
Priority: P0/P1/P2/P3

Vector: [entry point] -> [function A] -> [function B] -> [root cause]

Signal: [What's broken and how you know. Be specific.
  Bad:  "Auth is broken"
  Good: "GET /api/users returns 200 for unauthenticated requests.
         WorkspaceAuthGuard.canActivate() returns true unconditionally (stub)."]

Fix: [Exactly where and what to change.
  Bad:  "Fix the auth"
  Good: "workspace-auth.guard.ts — replace stub with JWT verification via
         supabase.auth.getUser(token) + workspace_members membership check"]

Verify: [How to confirm it's fixed.
  Bad:  "Test it"
  Good: "curl -H 'Authorization: Bearer invalid' /api/users → expect 401.
         curl with no auth → expect 401.
         curl with valid JWT for non-member workspace → expect 403.
         Build passes. Auth tests pass."]
```

### Trace Quality Checklist

For each trace, verify:
- [ ] Entry point is specific (HTTP method + path, event name, cron trigger — not "somewhere in backend")
- [ ] Trace path names actual functions, not directories
- [ ] Root cause is a specific line or function, not a file
- [ ] Fix describes the change, not just "fix it"
- [ ] Verification includes specific commands or test scenarios with expected output

### Traces for New Features (not just bugs)

New features get traces too — the "vector" is the flow the feature creates:

```
Chain: Add bulk import endpoint
Priority: P1

Vector: POST /api/v1/import/bulk -> ImportController.bulkImport()
  -> ImportService.validateCSV() -> ImportService.createBatch()
  -> pg-boss queue 'import-process' -> ImportWorker.process()

Signal: No bulk import exists. Users import one item at a time (45 seconds each).

Fix: Create ImportController, ImportService, ImportWorker at backend/src/modules/import/.
     CSV validation with zod. Batch creation with progress tracking.
     Worker processes batches of 50 with error isolation per row.

Verify: POST CSV with 100 rows -> returns batch ID.
        GET /api/v1/import/status/:batchId -> shows progress.
        Invalid rows logged, valid rows imported. Build passes. Tests pass.
```

---

## Phase 5: Sprint Proposal

Organize all discovered work into a sprint plan.

### Step 5.1: Assign Chains to Agents

Each trace has an obvious agent based on the fix site's territory:
- Fix is in a handler → BACKEND
- Fix is in a store → DATA
- Fix is in a component → FRONTEND
- Fix spans multiple territories → assign to the agent who owns the fix site; document reads for others

### Step 5.2: Organize into Waves

Standard wave order (customize based on your dependency analysis):

```
Wave 1: DATA + QA + INFRA + DESIGN     (foundation — no code dependencies)
Wave 2: BACKEND + SERVICES             (need stable data layer)
Wave 3: FRONTEND                       (needs design specs + stable backend)
Wave 4: RED TEAM                       (needs finished code to review)
Wave 5: LEAD                           (needs everything done to merge)
```

### Step 5.3: Scale Agent Count

Not every sprint needs all 9 agents. Scale to the work:

| Discovered Work | Agents to Dispatch |
|-----------------|-------------------|
| 1-3 chains, single layer | 1 agent (the relevant one) |
| 3-6 chains, 2 layers | 2-3 agents + LEAD if merge is non-trivial |
| 6-12 chains, 3+ layers | 4-6 agents based on which territories have chains |
| 12-20 chains, full stack | 7-9 agents, full wave structure |
| 20+ chains | Consider splitting into multiple sprints, or use nested teams per SCALING.md |

### Step 5.4: Write Success Criteria

Every sprint needs measurable success criteria:

```
Success criteria are pass/fail — no ambiguity:
  GOOD: "Dashboard loads in < 2s (currently 12s)"
  GOOD: "All 4 OWASP findings closed and verified with tests"
  GOOD: "Coverage on backend/ increases from 23% to 60%"
  BAD:  "Improve performance"
  BAD:  "Better security"
  BAD:  "More tests"
```

---

## Phase 6: Document Generation

Produce these files — the operator reviews and dispatches.

### Output 1: DISPATCH.md

The sprint plan. Template at `templates/dispatch.md`. Include:
- Sprint goals
- Execution traces (all chains with vectors, signals, fixes, verification)
- Wave assignments (which agent, which chains, estimated complexity)
- Merge order
- Success criteria
- Territory map
- Worktree setup script
- Teardown script

### Output 2: Per-Agent Task Docs

One `agent-[X]-[domain].md` per agent. Template at `templates/agent.md`. Include:
- Role description with cross-agent context
- Context reading list (numbered, specific files, with reasons)
- Files owned (explicit paths)
- Tasks with IDs ([X]-01, [X]-02), each with current state, required changes, key details
- Wave organization
- Territory with agent attribution
- Verification checklist with exact commands and expected output
- Commit strategy

### Output 3: Activation Prompts

One activation prompt per agent, ready to copy-paste. Template at `templates/activation.md`. These are generated from the DISPATCH.md and agent task docs.

### Output 4: Codebase Summary

Write a brief codebase analysis doc that captures what you learned. This helps future sprints:
- Architecture overview (layers, dependencies)
- Conventions discovered (naming, error handling, patterns)
- Territory map
- Known issues not addressed this sprint
- Recommended future work

---

## Presenting the Proposal

Present the sprint plan to the operator in this format:

```
## Sprint [XX] Proposal: [Theme]

### What I Found
[2-3 sentence summary of codebase analysis — architecture, health, key issues]

### Proposed Work
[List of chains grouped by agent, with priorities]

### Agents Needed
[Which agents, why, estimated complexity per agent]

### Wave Structure
[Which agents in which waves, and why this ordering]

### Risk Assessment
[What could go wrong — high-risk chains, potential merge conflicts, unknowns]

### Success Criteria
[Measurable, pass/fail criteria]

If this looks right, I'll generate the full DISPATCH.md and all agent task docs.
```

Wait for operator approval before generating the full documents. The operator may:
- Adjust scope (add/remove chains)
- Change priorities (promote P2 to P1, defer P3)
- Adjust agent count (fewer agents, simpler sprint)
- Reject the proposal entirely (different direction)

---

## Quick Reference: The Full Flow

```
1. Operator pastes: "Plan a sprint for [project]. Goal: [what to accomplish]"
2. You read: this guide + codebase + project context + progress tracker
3. You analyze: architecture, layers, conventions, territories
4. You discover: bugs, debt, security gaps, missing tests, features
5. You trace: execution traces for each piece of work
6. You propose: sprint plan with agents, waves, chains, success criteria
7. Operator approves (with adjustments)
8. You generate: DISPATCH.md + agent task docs + activation prompts
9. Operator sets up worktrees and dispatches agents
```

---

## Related Documents

- [templates/dispatch.md](../templates/dispatch.md) — DISPATCH.md template
- [templates/agent.md](../templates/agent.md) — Per-agent task doc template
- [templates/activation.md](../templates/activation.md) — Activation prompt templates and full dispatch flow
- [templates/completion.md](../templates/completion.md) — Completion report template
- [core/methodology.md](../core/methodology.md) — Execution traces, chain execution, priority levels
- [agents/README.md](../agents/README.md) — Agent roles, territories, wave structure
- [scaling/scaling.md](../scaling/scaling.md) — Scaling beyond 9 agents
