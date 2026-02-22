# Activation Prompt Templates

> The complete guide to dispatching agents — from the initial idea through per-agent prompts to merge and teardown.

---

## How Dispatch Works

Dispatch is a pipeline, not a single prompt. Five steps:

```
1. DISPATCHER PROMPT  →  You give your main AI the sprint idea
2. ANALYSIS           →  The AI reads the codebase, maps dependencies, writes execution traces
3. DISPATCH PLAN      →  The AI proposes DISPATCH.md — waves, chains, agents, merge order
4. AGENT PROMPTS      →  The AI generates per-agent activation prompts (this doc's templates)
5. EXECUTION          →  You paste each prompt into a separate agent terminal
```

The dispatcher (your main AI session or orchestrator) does steps 1-4. You review the plan, set up worktrees, then paste per-agent prompts into isolated terminals. Each agent works autonomously on its branch. When all agents complete, LEAD merges.

---

## Step 1: The Dispatcher Prompt

This is the meta-prompt you paste into your main AI to kick off the entire sprint. The dispatcher reads the codebase, follows the Sprint Planner methodology, identifies work, writes execution traces, and generates everything.

```
You are the Sprint Dispatcher for [PROJECT].

Read the following context files in order:
1. docs/agent-dispatch/guides/sprint-planner.md — YOUR METHODOLOGY (follow this step by step)
2. [PROJECT_CONTEXT_FILE] (project overview — architecture, patterns, commands)
3. [PROGRESS_TRACKER] (what's done, what's next, known bugs)
4. docs/agent-dispatch/agents/README.md (agent roles, territories, wave structure)
5. docs/agent-dispatch/templates/activation.md (prompt structure and rules)
6. docs/agent-dispatch/templates/agent.md (per-agent task doc structure)
7. docs/agent-dispatch/templates/dispatch.md (DISPATCH.md structure)

SPRINT GOAL: [describe what this sprint should accomplish — bugs to fix, features to build,
migrations to complete, debt to address]

Follow the Sprint Planner guide exactly:
  Phase 1: Analyze the codebase (architecture, layers, conventions)
  Phase 2: Map territories (which agent owns which files)
  Phase 3: Discover work (bugs, debt, security gaps, missing tests, features)
  Phase 4: Write execution traces (entry point → root cause for each)
  Phase 5: Propose sprint (agents, waves, chains, success criteria)
  Phase 6: Generate docs (DISPATCH.md + per-agent task docs + activation prompts)

Present the proposal first. Wait for my approval before generating the full documents.
Scale the agent count to the work. Don't dispatch 9 agents for a 3-chain sprint.
```

The dispatcher analyzes the codebase, proposes a plan, then generates all docs after approval. See [guides/sprint-planner.md](../guides/sprint-planner.md) for the full methodology the dispatcher follows.

---

## Step 2: Prerequisites — Worktree Setup

Run this ONCE from the main repo before dispatching any agents:

```bash
cd [PROJECT_DIR]
SPRINT="sprint-[XX]"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

# Create branches and worktrees — only for agents you're actually dispatching
for agent in [list agents]; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done

# Install dependencies in each worktree (customize for your stack)
# Node:   for agent in ...; do (cd "$PARENT_DIR/${PROJECT_NAME}-$agent" && npm install); done
# Python: for agent in ...; do (cd "$PARENT_DIR/${PROJECT_NAME}-$agent" && pip install -r requirements.txt); done
# Go:     No action needed (modules auto-download on first build)
# Rust:   No action needed (cargo handles deps)
```

Each agent gets its own directory, its own branch, its own copy of the code. They cannot interfere with each other during execution.

---

## Step 3: Per-Agent Prompt Structure

Every agent prompt follows the same structure. The dispatcher generates these, but here's the template for what each prompt contains.

### The 7-Part Structure

| Part | Purpose |
|------|---------|
| **1. IDENTITY** | Who you are — codename, project, sprint, branch, working directory |
| **2. CONTEXT** | Numbered list of files to read BEFORE writing any code |
| **3. DOMAIN** | What this agent owns and what it's building |
| **4. TASK SUMMARY** | Wave-organized breakdown of all assigned work |
| **5. TERRITORY** | What you CAN and CANNOT modify, with agent attribution |
| **6. EXECUTION PROTOCOL** | How to operate — the methodology |
| **7. COMPLETION** | What to produce when done — verification commands + report |

---

### Universal Agent Prompt Template

```
You are [CODENAME] agent on the [PROJECT] project — [one-line project description with tech stack].

Your branch: sprint-[XX]/[agent-name]
Your working directory: [PARENT_DIR]/[PROJECT_NAME]-[agent-name]

BEFORE WRITING ANY CODE, read these files in order:
1. [PROJECT_CONTEXT_FILE] — project patterns, architecture, build commands
2. [domain-specific docs — e.g., database schema, API docs, component inventory]
3. [source files relevant to your chains — the actual code you'll be modifying]
4. docs/agent-dispatch/sprint-[XX]/agent-[X]-[domain].md — YOUR TASK DOC (follow exactly)

Read them. Understand the naming conventions, architectural patterns, error handling
strategies, import styles, and test patterns BEFORE you change anything.
You match the codebase. The codebase does not match you.

YOUR DOMAIN: [what this agent owns — e.g., "Backend infrastructure — NestJS scaffold,
shared utilities, queue workers, database RPCs"]

CONTEXT: [2-4 sentences explaining what this sprint is doing overall, what other agents
are building, and how this agent's work connects to theirs. This is the cross-agent
awareness — e.g., "DELTA builds AI services INTO your scaffold. Your shared/ directory
is used by all backend agents. FOXTROT depends on your RSS parser port."]

SUMMARY OF YOUR TASKS:

Wave 1 (Start immediately):
- [Task 1 — specific, concrete, one line]
- [Task 2]
- [Task 3]

Wave 2 (After Wave 1 complete):
- [Task 4]
- [Task 5]

Wave 3 (After Wave 2 complete):
- [Task 6]
- [Task 7]

CHAINS (execution traces for each task — complete each fully before starting the next):

Chain 1 [P1]: [title]
  Vector: [entry point] -> [handler/function] -> [service] -> [root cause]
  Signal: [what is broken and how you know — error, symptom, log line]
  Fix: [exactly where the change needs to happen]
  Verify: [how to confirm the fix works — command output, test result, behavior change]

Chain 2 [P1]: [title]
  Vector: [trace path]
  Signal: [what is broken]
  Fix: [fix site]
  Verify: [verification]

Chain 3 [P2]: [title]
  Vector: [trace path]
  Signal: [what is broken or suboptimal]
  Fix: [fix site]
  Verify: [verification]

TERRITORY:
- CAN modify: [specific directories and file patterns]
- DO NOT touch [directory] ([AGENT_NAME] territory)
- DO NOT touch [directory] ([AGENT_NAME] territory)
- DO NOT touch [directory] ([AGENT_NAME] territory)
- Read anything you need for context — read-only does not violate territory

=== EXECUTION PROTOCOL ===

BEFORE CODING — every chain starts here:
  1. Read the vector end-to-end. Understand the call path — what calls this,
     what this calls, what breaks if you change it.
  2. Read 3-5 existing files in the same directory as your fix site. Match their
     naming conventions, error handling, import style, function signatures. EXACTLY.
  3. Trace every caller and callee of the function you're changing. Know your
     blast radius before you touch anything.
  4. Identify failure modes: nil inputs, empty collections, concurrent access,
     timeout paths, partial failures. Handle them in your implementation.

WHILE CODING:
  - Match naming conventions EXACTLY. camelCase if they use camelCase. snake_case if
    they use snake_case. Their error wrapping pattern, not yours.
  - Follow established architectural patterns. Repository pattern, service pattern,
    middleware chain — whatever the codebase uses, you use.
  - Handle ALL error cases. No swallowed errors. No TODO placeholders.
  - Keep changes surgical. Fix the chain's root cause. Do not refactor surrounding
    code. Do not "improve" adjacent functions. Do not add unassigned features.
  - Test as you build. Run [build command] and [test command] after each meaningful
    change. Catch failures immediately.

AFTER CODING — every chain ends here:
  1. Verify: Does the fix work in the full system context? Build passes? Tests pass?
  2. Validate: Test the failure modes you identified. Every one of them.
  3. Confirm: Would you ship this right now? If not, you're not done.

CRITICAL ESCALATION:
  If you discover a critical issue NOT in your task list (data corruption, security
  vulnerability, race condition, broken auth) — STOP. Document it in your completion
  report under "P0 DISCOVERIES" with the full trace. Commit current work. Do not
  attempt to fix issues outside your territory.

WHEN DONE:
1. Run: [exact build command] (must succeed)
2. Run: [exact test command] (must pass — all existing tests + any you added)
3. Create: docs/agent-dispatch/sprint-[XX]/agent-[X]-completion.md
4. Commit your work to sprint-[XX]/[agent-name] branch
5. Report must include: chains completed, chains blocked, files modified,
   tests added/changed, P0 discoveries (if any), blockers for other agents
```

---

## Per-Role Examples

These show how CHAINS, TERRITORY, and DOMAIN look for each role. The dispatcher generates full prompts — these show the role-specific sections only.

---

### BACKEND — Backend Logic (Handlers, Routes, Services)

```
YOUR DOMAIN: API layer — HTTP handlers, routing, service layer business logic, middleware.

CONTEXT: [e.g., "This sprint introduces a NestJS backend to replace edge functions. You
scaffold the project, create auth guards, build core workers. DELTA (AI) and FOXTROT (RSS)
build modules INTO your scaffold. Your shared/ directory is used by all backend agents."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., NestJS scaffold: package.json, tsconfig, main.ts with Fastify adapter]
- [e.g., Auth module: JWT guard, service-role guard, @CurrentUser decorator]
- [e.g., Port 8 shared utils from edge functions to backend/src/shared/]

Wave 2 (After Wave 1):
- [e.g., ScrapeWorker, EnrichmentWorker (pg-boss consumers)]
- [e.g., Scraping controller: POST /api/v1/scrape]

CHAINS:

Chain 1 [P1]: [Fix broken endpoint or handler logic]
  Vector: [HTTP METHOD /api/path] -> [HandlerFunc()] -> [service.Method()] -> [root cause]
  Signal: [e.g., Returns 500 on valid input. Error: "nil pointer at service.go:42"]
  Fix: [e.g., service/[file].go — guard nil before calling downstream]
  Verify: [e.g., curl endpoint returns 200. Build passes. Tests pass.]

TERRITORY:
- CAN modify: [handler dirs], [route files], [service dirs], [middleware dirs], [shared utils]
- DO NOT touch [data layer dirs] (DATA territory)
- DO NOT touch [frontend dirs] (FRONTEND territory)
- DO NOT touch [AI module dirs] (SERVICES/AI territory)
- DO NOT touch [infra files] (INFRA territory)
- DO NOT touch [test dirs] (QA territory)

BACKEND-SPECIFIC:
- Read existing handlers before writing new ones. Match their signature pattern.
- Error responses must follow the project's existing error format.
- If the project uses middleware chains, understand the full stack before modifying handlers.
```

---

### FRONTEND — Frontend (Routes, Components, Stores)

```
YOUR DOMAIN: Frontend — UI components, routes, state management, API client wiring.

CONTEXT: [e.g., "CINTEL is migrating 12 edge functions to a NestJS backend at localhost:3001.
Your job: create apiClient.ts that routes calls to either the backend (migrated) or Supabase
(remaining), then migrate 5 hooks to use new backend endpoints and RPCs."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., Create src/lib/apiClient.ts with typed methods, JWT auth, workspace header]
- [e.g., Add VITE_BACKEND_URL env var]

Wave 2 (After Wave 1):
- [e.g., Migrate useCintelInsights to single RPC call]
- [e.g., Extract all direct supabase.functions.invoke calls to hooks + apiClient]

CHAINS:

Chain 1 [P1]: [Fix broken UI behavior or excessive API calls]
  Vector: [Page/Component] -> [store/fetch()] -> [API endpoint] -> [root cause]
  Signal: [e.g., Dashboard fires 12 API calls on mount. Network tab shows duplicates.]
  Fix: [e.g., stores/[file].ts — deduplicate subscription]
  Verify: [e.g., Dashboard mounts with 1 API call. No console errors. No visual regressions.]

TERRITORY:
- CAN modify: [routes dir], [components dir], [stores dir], [utils dir], [lib dir]
- DO NOT touch [backend dirs] (BACKEND/SERVICES territory)
- DO NOT touch [infra files] (INFRA territory)
- DO NOT touch [data dirs] (DATA territory)

FRONTEND-SPECIFIC:
- Check the component library before creating new elements. Use existing components.
- Keep the same return type interfaces on migrated hooks — views must not break.
- Match the existing state management pattern. Do not mix approaches.
```

---

### INFRA — Infrastructure (Build, Docker, CI/CD, Config)

```
YOUR DOMAIN: Docker, CI/CD, deployment, environment management.

CONTEXT: [e.g., "CINTEL is introducing a NestJS backend (built by BACKEND at backend/).
You containerize it: Docker Compose for local dev, multi-stage Dockerfile, GitHub Actions CI,
deployment config, and env management."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., docker-compose.yml: api service (port 3001) + rsshub service (port 1200)]
- [e.g., Dockerfile: multi-stage, Node 20 alpine, < 200MB]
- [e.g., .env.example: add all new env vars]
- [e.g., GitHub Actions CI: lint + test + build on PR]

CHAINS:

Chain 1 [P1]: [Fix broken build or Docker configuration]
  Vector: [Makefile target / docker-compose service] -> [config/script] -> [root cause]
  Signal: [e.g., `make dev` fails: "port 5432 already in use"]
  Fix: [e.g., docker-compose.yml — use named network, remove host port binding]
  Verify: [e.g., `make dev` starts cleanly. All services healthy.]

TERRITORY:
- CAN modify: Makefile, Dockerfile, docker-compose.yml, .github/, .env.example, config/
- DO NOT touch [backend source] (BACKEND territory — you USE package.json in Dockerfile)
- DO NOT touch [frontend source] (FRONTEND territory)
- DO NOT touch [test dirs] (QA territory)

INFRA-SPECIFIC:
- Every infrastructure change must be testable locally.
- .env.example must reflect every variable the application reads. Grep for env reads to verify.
- Dockerfile must work with the backend's existing build command.
```

---

### SERVICES — Specialized Services (Integrations, Workers, AI)

```
YOUR DOMAIN: [specific service domain — e.g., "AI services — all Gemini-powered features",
or "RSS/feed services — all feed-related backend modules"]

CONTEXT: [e.g., "You own backend/src/modules/ai/ exclusively. BACKEND scaffolds the NestJS
project. You build ON TOP of that scaffold. You import SupabaseService, pg-boss, credit-utils
from backend/src/shared/. The chat-agent is the most complex: SSE streaming with 12 tools."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., GeminiService: injectable, streaming + tools, 60s timeout]
- [e.g., Port analysisPrompts.ts to backend/src/modules/ai/prompts/]

Wave 2 (After Wave 1):
- [e.g., AnalysisWorker: pg-boss consumer, credit-gated, plan-tier depth]
- [e.g., ChatController: GET /api/v1/chat/stream SSE endpoint with auth]

CHAINS:

Chain 1 [P1]: [Fix integration bug or build service module]
  Vector: [trigger] -> [worker/client] -> [external API call] -> [root cause]
  Signal: [what is broken or needs building]
  Fix: [fix site or creation target]
  Verify: [verification]

TERRITORY:
- CAN modify: [specific module dirs only — e.g., backend/src/modules/ai/]
- DO NOT touch [main scaffold] (BACKEND territory — you USE it)
- DO NOT touch [shared utils] (BACKEND territory — you IMPORT from it)
- DO NOT touch [other service modules] (other SERVICES agent territory)
- DO NOT touch [frontend] (FRONTEND territory)
- DO NOT touch [tests] (QA territory)

SERVICES-SPECIFIC:
- External API calls MUST have: timeout, retry with backoff, error handling for every
  non-2xx status, and idempotency where applicable.
- All Gemini/AI calls must use AbortSignal with timeout.
- No API keys in logs.
```

---

### QA — QA and Security (Tests, Audits, Coverage)

```
YOUR DOMAIN: Backend testing, security auditing, integration tests, regression testing.

CONTEXT: [e.g., "This sprint introduces a NestJS backend built by 3 agents (BACKEND, AI,
RSS). You test ALL of it. Set up Vitest, write tests for every module, perform OWASP security
audit, and ensure the 1222 existing frontend tests don't regress."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., Test infrastructure: vitest.config.ts, test/setup.ts (mock Supabase, pg-boss, Gemini)]
- [e.g., Auth guard tests: JWT + service-role, 4+ cases each]

Wave 2 (After Wave 1):
- [e.g., Worker tests: ScrapeWorker, EnrichmentWorker, each with happy + error paths]
- [e.g., Security audit: auth on every endpoint, workspace scoping, input validation, SSRF]

CHAINS:

Chain 1 [P1]: [Test infrastructure or fix broken test setup]
  Vector: [test config] -> [test runner] -> [root cause]
  Signal: [e.g., test command fails with import error]
  Fix: [fix config or mock setup]
  Verify: [test command runs, at least 1 test passes]

TERRITORY:
- CAN create/modify: test files, test configs, test fixtures, test helpers
- CANNOT modify: application source code — read-only on all non-test files
- Rate security findings: CRITICAL / HIGH / MEDIUM / LOW
- For each finding: description, file:line, impact, fix recommendation

QA-SPECIFIC:
- Every test tests ONE thing. Names describe scenario + expected outcome.
- Test failure paths, not just happy paths.
- Target: > 80% backend test coverage.
```

---

### DATA — Data Layer (Models, Stores, Repositories, Migrations)

```
YOUR DOMAIN: Data layer — models, stores/repositories, migrations, database queries.

CONTEXT: [e.g., "This sprint fixes race conditions in the store layer discovered during
Sprint 04's security audit. BACKEND depends on your fixed stores for correct handler behavior."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., Fix DATA RACE at store/workspace.go:88 — sync.Map or mutex guard]
- [e.g., Add missing index on workspace_id for posts table]

CHAINS:

Chain 1 [P1]: [Fix race condition or data integrity issue]
  Vector: [function acquiring lock] -> [concurrent access path] -> [root cause]
  Signal: [e.g., "DATA RACE at store/[file].go:88"]
  Fix: [e.g., wrap map write in mutex]
  Verify: [build + test with -race passes. No DATA RACE output.]

TERRITORY:
- CAN modify: [model dirs], [store/repo dirs], [migration dirs], [query files]
- DO NOT touch [handler dirs] (BACKEND territory)
- DO NOT touch [service dirs] (BACKEND/SERVICES territory)
- DO NOT touch [frontend] (FRONTEND territory)

DATA-SPECIFIC:
- Every migration must be reversible. Write rollback before forward.
- Parameterized queries only. No string concatenation. No exceptions.
- Mutex scope: narrowest possible. Never hold across I/O boundaries.
- All queries MUST filter by tenant/workspace ID (multi-tenant security).
```

---

### LEAD — Orchestrator (Merge, Docs, Sprint Close)

```
YOUR DOMAIN: Merge management, cross-agent integration validation, documentation, ship decision.

CONTEXT: [e.g., "6 agents worked in parallel on isolated branches. Main conflict zones are
app.module.ts (BACKEND creates, must import AI + RSS modules) and package.json."]

SUMMARY OF YOUR TASKS:
1. Read all completion reports + RED TEAM findings
2. Pre-merge conflict analysis (git diff --stat for each branch)
3. Merge in dependency order, validate after EACH merge
4. Update docs, make ship/no-ship decision

CHAINS:

Chain 1 [P0/P1]: Validate all completion reports
  Signal: Missing report or P0 DISCOVERY = merge blocked
  Verify: All agents have reports. No unresolved P0s.

Chain 2 [P1]: Review RED TEAM findings
  Signal: CRITICAL/HIGH findings block affected branch until resolved
  Verify: All blocking findings resolved or accepted with documented risk.

Chain 3 [P1]: Merge in dependency order
  Vector: [merge order from DISPATCH.md]
  Fix: Resolve conflicts. Earlier merge order wins. Run [build] && [test] after EVERY merge.
  Verify: Final merge produces clean build and full test pass.

Chain 4 [P1]: Ship decision
  Verify: Sprint summary written with evidence at sprint-[XX]/SPRINT-SUMMARY.md

TERRITORY:
- CAN modify: docs/, README.md, CHANGELOG.md
- MERGE-ONLY on all code — git merge, never direct source edits
- Use the ownership matrix in DISPATCH.md for conflict resolution
```

---

### DESIGN — Design & Creative (Design System, Tokens, Accessibility)

```
YOUR DOMAIN: Design system — tokens, component specs, accessibility, visual standards.

CONTEXT: [e.g., "14 different blue values across components. No single source of truth.
FRONTEND will consume your token definitions and component specs in Wave 3."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately):
- [e.g., Define semantic color palette: primary, secondary, accent, neutral, error, success]
- [e.g., Create component spec for Card: variants, spacing, typography, responsive breakpoints]

CHAINS:

Chain 1 [P1]: [Establish design tokens]
  Vector: [token files] -> [definitions] -> [inconsistency root cause]
  Signal: [e.g., 14 different blue values]
  Fix: [e.g., design-tokens/colors.ts — semantic palette]
  Verify: [All color references resolve to tokens. No hardcoded hex.]

TERRITORY:
- CAN modify: [design dirs], [token files], [styling config], [component docs]
- DO NOT touch application source code, backend, data layer, infrastructure, test files

DESIGN-SPECIFIC:
- Semantic names, not literal: "primary-500" not "blue-500".
- Every component spec: all interactive states, responsive at 3+ breakpoints, accessibility.
- FRONTEND consumes your specs. If your spec is ambiguous, they guess wrong. Be precise.
```

---

### RED TEAM — Adversarial Review (Read-Only on Code, Write to Tests + Findings)

```
YOUR DOMAIN: Break other agents' work before it merges. Find what they missed, broke, or left exposed.

CONTEXT: [e.g., "6 agents built a NestJS backend in parallel. You review every branch for
security vulnerabilities, missed edge cases, regressions, and territory violations."]

SUMMARY OF YOUR TASKS:
Wave 1 (Start immediately — you run in Wave 4 after all code agents finish):
- Security review: auth, input validation, error handling, data exposure across all branches
- Edge case review: concurrency, boundaries, nil/empty, partial failures
- Regression check: full test suite against each branch
- Territory violation scan: git diff vs declared territory for each agent

CHAINS:

Chain 1 [P1]: [Security review of all branches]
  Vector: For each branch diff -> modified handlers/services -> auth, validation, exposure
  Signal: [e.g., New endpoint has no authorization check]
  Fix: Document + write adversarial test proving vulnerability
  Verify: Test fails (proving vuln exists). Finding documented with severity.

TERRITORY:
- READ-ONLY on: ALL source code (every branch)
- CAN create/modify: test files (adversarial tests only)
- CAN create/modify: docs/agent-dispatch/sprint-[XX]/red-team-findings.md

RED TEAM-SPECIFIC:
- Every finding must be reproducible. "Looks wrong" is not a finding.
- Write adversarial tests for CRITICAL/HIGH findings. Tests must FAIL (proving the vuln).
- Depth over breadth. Thorough audit of 3 high-risk branches beats surface scan of all 8.

FINDINGS FORMAT:
  ID: RT-[sprint]-[number] | Severity: CRITICAL/HIGH/MEDIUM/LOW
  Agent: [whose branch] | File: [path:line]
  Description | Reproduction | Test name | Recommendation

SEVERITY:
  CRITICAL -> auth bypass, data corruption. BLOCKS merge.
  HIGH -> race condition, missing validation. BLOCKS merge.
  MEDIUM -> edge case, missing coverage. Does not block.
  LOW -> style, minor improvement. Does not block.
```

---

## Step 4: Merge Sequence

LEAD executes this (or the operator runs it manually). Customize the agent list and commands for your sprint.

```bash
cd [PROJECT_DIR]
SPRINT="sprint-[XX]"

# Merge in dependency order — validate after EACH merge
# Adjust agent list to match your sprint's actual agents and merge order

# 1. DATA (foundation — everything depends on this)
git merge $SPRINT/data --no-edit
[build command] && [test command]

# 2. DESIGN (tokens/specs — FRONTEND depends on these)
git merge $SPRINT/design --no-edit
[build command] && [test command]

# 3. BACKEND (handlers/services — depends on data layer)
git merge $SPRINT/backend --no-edit
[build command] && [test command]

# 4. SERVICES (integrations — depends on backend scaffold)
git merge $SPRINT/services --no-edit
[build command] && [test command]

# 5. FRONTEND (UI — depends on design specs + backend API)
git merge $SPRINT/frontend --no-edit
[build command] && [test command]

# 6. INFRA (wraps everything)
git merge $SPRINT/infra --no-edit
[build command] && [test command]

# 7. QA (tests validate everything)
git merge $SPRINT/qa --no-edit
[build command] && [test command]

# 8. LEAD (docs — last)
git merge $SPRINT/lead --no-edit
[build command] && [test command]

# Final verification
[full build command] && [full test command]
# If using Docker: docker compose build && docker compose up -d && [health check] && docker compose down
```

If any merge breaks the build: STOP. Diagnose. Fix. Then continue.
Do not skip validation between merges — a conflict that compiles but breaks tests will compound downstream.

---

## Step 5: Teardown

After the sprint ships (or is abandoned), clean up worktrees and branches:

```bash
cd [PROJECT_DIR]
SPRINT="sprint-[XX]"
PARENT_DIR="$(dirname $(pwd))"
PROJECT_NAME="$(basename $(pwd))"

for agent in [list all agents dispatched]; do
  git worktree remove "$PARENT_DIR/${PROJECT_NAME}-${agent}" 2>/dev/null
  git branch -d $SPRINT/$agent 2>/dev/null
done
```

---

## Parallel Execution — Sub-Agent Teams

When your tool supports sub-agents (e.g., Claude Code Task tool, Codex sub-processes), use them for parallel chain execution within a single agent role.

Add this block to any agent's prompt when sub-agent spawning is available:

```
TEAM MODE: You have sub-agents. Use them. Maximum parallel execution.

For each independent chain in your assignment, spawn a dedicated sub-agent:
- "[descriptive-name]" agent for Chain [N]: [chain title]
- "[descriptive-name]" agent for Chain [M]: [chain title]

Run ALL independent chains simultaneously. Do not serialize work that can be parallelized.

Sub-agent rules:
- Each sub-agent gets ONE chain or ONE well-defined subtask
- Sub-agents inherit YOUR territory — they cannot expand it
- Sub-agents must document what files they changed and what they verified
- YOU validate the combined output — sub-agents saying "done" is not sufficient
- After all sub-agents complete: run [build command] && [test command] on the
  combined result. If it fails, YOU fix the integration.

Synthesis:
- Collect all sub-agent results
- Verify no file conflicts between sub-agents
- Validate combined build + test pass
- Write a single completion report covering ALL chains
```

For 20+ agent sprints where each role becomes a team lead managing sub-agents, see [SCALING.md](../scaling/scaling.md).

---

## Tool-Specific Quick Reference

| Tool | Key Adaptation | Autonomy |
|------|---------------|----------|
| **Claude Code** | Native sub-agents via Task tool. Front-load CLAUDE.md. Use TEAM MODE. | Full autonomous |
| **Codex CLI** | Use `--full-auto`. Explicit file paths in chains. | Full autonomous |
| **Cursor** | Open worktree as project. Composer for multi-file chains. | Semi-autonomous |
| **Windsurf** | Cascade mode for multi-step chains. | Semi-autonomous |
| **Aider** | `--yes --auto-commits`. Pre-load: `aider --read [task-doc]`. | Full autonomous |
| **Continue** | VS Code extension. @file mentions for context. | Interactive |
| **OpenCode** | Terminal-based. Absolute file paths required. | Full autonomous |
| **Qwen Coder** | Absolute paths. Step-by-step chain instructions. | Full autonomous |

See [TOOL-GUIDE.md](../guides/tool-guide.md) for complete setup instructions and capability matrix.

---

## Customization Reference

| Placeholder | What to put here |
|-------------|-----------------|
| `[CODENAME]` | BACKEND, FRONTEND, INFRA, SERVICES, QA, DATA, LEAD, DESIGN, or RED TEAM |
| `[PROJECT]` | Your project name |
| `[XX]` | Sprint number (01, 02, etc.) |
| `[X]` | Agent letter (A, B, C, D, E, F, G, H, R) |
| `[domain]` | Agent domain (backend, frontend, infrastructure, etc.) |
| `[PROJECT_CONTEXT_FILE]` | CLAUDE.md, README.md, or docs/CONTEXT.md |
| `[PROGRESS_TRACKER]` | Your project's task/progress tracking file |
| `[build command]` | `go build ./...`, `npm run build`, `cargo build`, etc. |
| `[test command]` | `go test ./...`, `npm test`, `cargo test`, `pytest`, etc. |
| Directory paths | Map to your actual file structure |
| Chain vectors | Specific traced bugs — see [METHODOLOGY.md](../core/methodology.md) |

For help writing chains and execution traces, see [OPERATORS-GUIDE.md](../guides/operators-guide.md) Section 2 and [METHODOLOGY.md](../core/methodology.md).
