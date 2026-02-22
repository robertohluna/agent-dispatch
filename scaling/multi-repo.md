# Agent Dispatch Across Multiple Repositories

> How to run coordinated multi-agent sprints when your project spans more than one repo
> Last Updated: 2026-02-22

---

## When This Guide Applies

Single-repo dispatch is simpler. Use it whenever you can. This guide is for the cases where you genuinely cannot:

- A separate backend repo and a separate frontend repo that evolve together
- Three or more microservices that each live in their own repo
- A monorepo with multiple independently deployable packages (apps/api, apps/web, packages/shared)

The core Agent Dispatch methodology does not change in any of these cases — execution traces, chain execution, priority levels, wave order. What changes is how you set up worktrees, how LEAD coordinates across repo boundaries, and how you sequence merges across multiple `main` branches.

---

## 1. Separate Backend + Frontend Repos

The most common split. You have a backend repo (Go API, Node API, Django, Rails) and a frontend repo (Next.js, SvelteKit, React). They talk through an API contract. Changes to that contract require coordination.

### Two Repos, Two Sets of Worktrees

Each repo gets its own independent set of worktrees. The worktree setup script runs once per repo.

```bash
# Backend repo
SPRINT="sprint-01"
BACKEND_DIR="/path/to/backend"
BACKEND_PARENT="$(dirname $BACKEND_DIR)"
BACKEND_NAME="$(basename $BACKEND_DIR)"

cd "$BACKEND_DIR"
for agent in backend data services qa infra lead; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$BACKEND_PARENT/${BACKEND_NAME}-${agent}" $SPRINT/$agent
done

# Frontend repo
FRONTEND_DIR="/path/to/frontend"
FRONTEND_PARENT="$(dirname $FRONTEND_DIR)"
FRONTEND_NAME="$(basename $FRONTEND_DIR)"

cd "$FRONTEND_DIR"
for agent in frontend qa lead; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$FRONTEND_PARENT/${FRONTEND_NAME}-${agent}" $SPRINT/$agent
done
```

After this, your filesystem looks like:

```
projects/
├── backend/                    ← Backend main repo (don't touch during sprint)
├── backend-backend/              ← BACKEND's backend workspace
├── backend-data/            ← DATA's backend workspace
├── backend-services/              ← SERVICES's backend workspace
├── backend-qa/               ← QA's backend workspace
├── backend-infra/            ← INFRA's backend workspace
├── backend-lead/               ← LEAD's backend workspace
├── frontend/                   ← Frontend main repo (don't touch during sprint)
├── frontend-frontend/             ← FRONTEND's frontend workspace
├── frontend-qa/              ← QA's frontend workspace (cross-repo)
└── frontend-lead/              ← LEAD's frontend workspace (cross-repo)
```

### Territory Mapping Across Repos

Agent territories follow repo boundaries, with two agents that span both:

| Agent | Repo | Territory |
|-------|------|-----------|
| BACKEND | Backend only | Handlers, services, middleware |
| DATA | Backend only | Models, store, migrations |
| SERVICES | Backend only | Integrations, workers, external APIs |
| INFRA | Backend only | Dockerfile, CI/CD, build config |
| FRONTEND | Frontend only | Routes, components, stores |
| QA | Both repos | Tests in backend + tests in frontend |
| LEAD | Both repos | Merge authority in both, docs in both |

QA and LEAD are the two agents that work across repo boundaries. QA needs access to both test suites to verify the full system. LEAD needs to coordinate merge order across both `main` branches.

### How to Handle API Contract Changes

API contract changes are the most dangerous operations in a split-repo setup. A change to a backend response shape, a new required field, or a renamed endpoint will break the frontend until it adapts. The coordination rule is simple: **backend ships first, frontend adapts second.**

The practical workflow for a sprint that includes a contract change:

```
Phase 1 — Backend lands the new contract:
  Backend Wave 1: DATA (model changes), QA (backend tests)
  Backend Wave 2: BACKEND (new endpoint behavior)
  Backend merge: DATA → BACKEND → QA merged to backend main
  Backend validation: full backend test suite passes

Phase 2 — Frontend adapts to the new contract:
  Frontend Wave 1: FRONTEND (update components, network layer, type definitions)
  Frontend Wave 1: QA (update frontend tests, update API mocks to new shapes)
  Frontend merge: FRONTEND → QA merged to frontend main
  Frontend validation: full frontend test suite passes, E2E passes
```

FRONTEND's activation prompt for Phase 2 must reference the specific contract changes:

```
CONTEXT: The backend has shipped a contract change in sprint-01.
New response shape for GET /api/content/:id:
  - thumbnailUrl is now always a fully-qualified HTTPS URL (was sometimes null)
  - Added field: durationSeconds (number, always present)
  - Removed field: rawMetadata (was optional, now stripped at API layer)

Your task is to update the frontend to consume this new shape.
Read backend-main/docs/api/content.md for the full updated contract.
```

If QA in the backend repo writes contract documentation as part of its work, FRONTEND has a precise spec to work from. This is not optional when contracts change — FRONTEND cannot reliably adapt to undocumented changes.

### Wave Order Across Repos

The wave order spans both repos and runs in two distinct phases:

```
BACKEND PHASE:
  Wave 1 (backend): DATA + QA + INFRA  (parallel — no deps)
  Wave 2 (backend): BACKEND + SERVICES             (need stable data layer)
  Wave 3 (backend): LEAD-backend              (merges backend branches)

FRONTEND PHASE (start after backend Wave 3 completes):
  Wave 4 (frontend): FRONTEND + QA             (parallel — FRONTEND adapts, QA validates)
  Wave 5 (frontend): LEAD-frontend            (merges frontend branches)
```

The gate between Wave 3 and Wave 4 is explicit: LEAD-backend's completion report must confirm that the backend contract changes are merged to backend `main` before FRONTEND begins. FRONTEND should point at the actual merged backend code, not a speculative branch.

### Merge Order Across Repos

```
1. Backend repo:  DATA → BACKEND → SERVICES → INFRA → QA → LEAD
                  (standard dependency order)

2. Frontend repo: FRONTEND → QA → LEAD
                  (standard order, but only after backend is merged and validated)
```

A good rule of thumb: tag the backend release before starting the frontend merge. This gives you a stable reference point if the frontend merge surfaces problems that require a backend revert.

```bash
# After backend LEAD completes:
cd /path/to/backend
git tag sprint-01-backend-complete
git push origin main --tags

# Then start frontend Phase:
cd /path/to/frontend-frontend
# ... dispatch FRONTEND with the tagged backend as reference
```

---

## 2. Microservices (3+ Repos)

Three or more services, each in its own repo, communicating over HTTP or a message bus. A feature that crosses service boundaries requires coordinating multiple repos simultaneously.

### Dispatch Options: One Per Service or One Per Layer

**Option A: One DISPATCH.md per service repo**

Each service gets its own full dispatch — its own set of agents, its own sprint docs, its own merge order. The services are treated as independent teams that happen to be working at the same time.

Use this when:
- Services are owned by different people or teams
- The services evolve independently most of the time
- This sprint's changes are mostly contained within individual services

**Option B: One shared DISPATCH.md that references chains across repos**

A single sprint document that defines chains spanning the full request path, with agent assignments that map to specific repos. The DISPATCH.md lives in a shared location (the contracts repo, a sprint-docs repo, or a shared directory).

Use this when:
- Changes span multiple services in a single chain (API gateway → service A → service B → DB)
- You are the operator across all services
- Coordination between services is the primary challenge of the sprint

### Cross-Service Chain Example

A chain that starts at the API gateway, flows through two services, and ends at the database:

```markdown
### Chain 1: Add rate limiting to user search (P1)
Vector: GET /api/users/search
  → api-gateway/src/routes/users.ts (gateway routing)
  → user-service/internal/handler/search.go (search handler)
  → user-service/internal/service/search.go (business logic)
  → search-service/src/index.ts (Elasticsearch client)
  → Elasticsearch cluster

Signal: Unbounded search queries cause Elasticsearch timeouts at >50 results.
         Gateway has no rate limiting. User service has no pagination enforcement.

Agents:
  BACKEND (api-gateway repo):    Add rate limiting middleware at gateway
  BACKEND (user-service repo):   Enforce pagination (max 50 results) in handler
  SERVICES (search-service repo): Add query timeout to Elasticsearch client
  QA (all repos):            Write integration tests for the rate-limited path

Fix sequence: SERVICES first (Elasticsearch timeout), then BACKEND (enforcement),
              then BACKEND-gateway (rate limiting). QA writes tests last.
```

This chain involves three repos. You have two choices for agent assignment:

- **One agent per service** — BACKEND-1 owns api-gateway, BACKEND-2 owns user-service, SERVICES owns search-service. Each works in their repo's worktree independently.
- **One agent per layer** — One BACKEND agent owns all "handler/routing" changes across all three repos, working in each worktree sequentially.

The one-agent-per-service approach is generally better. It avoids agents context-switching between repos, it keeps worktrees clear of confusion, and it maps cleanly to how teams actually work.

### Agent Assignment for Microservices

A practical assignment pattern for a 3-service setup:

| Agent | Repos | Domain |
|-------|-------|--------|
| BACKEND-1 | api-gateway | Gateway routing, auth middleware, rate limiting |
| BACKEND-2 | user-service | User handlers, service logic |
| BACKEND-3 | order-service | Order handlers, service logic |
| DATA | user-service, order-service | All data layer work across both services |
| SERVICES | all services | Integration code, external API clients, message bus |
| QA | all repos | Tests in all repos |
| LEAD | all repos | Merge coordination across all repos |
| INFRA | all repos | CI/CD, Docker, build config across all services |

DATA and QA are naturally cross-service because the data layer and test layer exist in every service. Give them access to all relevant worktrees. LEAD and INFRA also operate across all repos — LEAD for merge coordination, INFRA because Docker and CI/CD config tends to be consistent across services.

For large microservice estates (5+ services), consider splitting DATA by service (DATA-users, DATA-orders) rather than having one agent work across all data layers.

### Shared Libraries and Contracts Repo

If your services share a contracts or libraries repo (OpenAPI specs, protobuf definitions, shared TypeScript types, a common Go module), that repo needs its own agent assignment.

Treat it as the foundation layer — equivalent to DATA in a monolith. Changes to the contracts repo must be merged before any service that consumes those contracts can run its tests against the updated shapes.

```
Sprint order for a sprint touching shared contracts:

Wave 1: DATA (contracts repo) — update contract definitions
        QA (contracts repo) — validate contract schema, write contract tests
        LEAD-contracts — merge contracts repo first

Wave 2: All service repos — consume updated contracts
        (Service agents can now point their dependencies at the new contract version)

Wave 3: LEAD (each service repo) — merge services in dependency order
```

The contracts repo should be tagged (or a version bumped) before any service agent begins work. Services should reference the tagged version, not a branch. This makes the dependency explicit and reversible.

```bash
# After contracts repo merges:
cd /path/to/contracts
git tag sprint-01-contracts-v1.2.0
git push origin main --tags

# In each service's agent activation prompt:
# CONTEXT: This sprint uses contracts v1.2.0 (tagged sprint-01-contracts-v1.2.0).
# Update your go.mod / package.json / requirements.txt to reference this tag.
```

### Integration Testing Across Services

QA needs access to all service repos to write and run integration tests that span service boundaries. This is the one case where an agent legitimately needs multiple worktrees simultaneously.

Two patterns:

**Pattern A: Docker Compose integration environment**

QA spins up all services locally via docker-compose and runs integration tests against the full stack. Each service's worktree is bind-mounted into the compose setup.

```yaml
# docker-compose.test.yml — referenced in QA's activation prompt
services:
  api-gateway:
    build: /path/to/api-gateway-qa
    ports: ["8080:8080"]
  user-service:
    build: /path/to/user-service-qa
    ports: ["8081:8081"]
  search-service:
    build: /path/to/search-service-qa
    ports: ["8082:8082"]
```

QA's territory in this pattern: test files in all repos, plus the docker-compose.test.yml in whatever repo owns it.

**Pattern B: Contract tests per service**

Each service has contract tests that verify it correctly implements the agreed interface — no real cross-service calls required. QA writes contract tests in each service repo independently.

This scales better for large microservice estates and does not require QA to manage a multi-service local environment. The tradeoff is that contract tests do not catch integration bugs that arise from network behavior (timeouts, retry storms, message ordering).

For most sprints, Pattern B is sufficient. Use Pattern A when you are changing cross-service behavior and need end-to-end confidence before merging.

### Merge Order for Microservices

Merge order follows the dependency graph, not backendbetical order. Identify which service is upstream (closer to the data) and which is downstream (closer to the user). Upstream merges first.

```
Example: API gateway → user service → order service → shared database

Merge order:
  1. shared-db repo (schema changes, if any)
  2. user-service repo (depends on shared-db)
  3. order-service repo (depends on shared-db)
  4. api-gateway repo (depends on user-service + order-service contract)

Within each repo, standard merge order applies:
  DATA → BACKEND → SERVICES → QA → INFRA → LEAD
```

---

## 3. Monorepo with Multiple Packages

A single Git repository containing multiple independently deployable units. Common patterns: `apps/web` + `apps/api` + `packages/shared`, or a Go workspace with multiple modules, or a Turborepo/Nx workspace.

### Territory Boundaries Follow Package Boundaries

In a monorepo, agent territories map to package directories rather than layer directories. Each package is its own territory. The shared packages layer is DATA's territory because it is the foundation everything else depends on.

```
Example: apps/web, apps/api, packages/ui, packages/shared

BACKEND:    apps/api/src/routes/, apps/api/src/services/
FRONTEND:    apps/web/src/routes/, apps/web/src/components/
DATA:  packages/shared/, apps/api/src/db/, apps/api/src/models/
SERVICES:    apps/api/src/integrations/, apps/api/src/workers/
QA:     apps/*/tests/, packages/*/tests/, **/*.test.ts, **/*.spec.ts
INFRA:  turbo.json, .github/, Dockerfile, docker-compose.yml
LEAD:     docs/, CHANGELOG.md, package.json (root), go.work (root)
```

QA and LEAD span all packages, same as in the multi-repo setup. The difference is they are all in the same worktree — no need to jump between repo directories.

### Single Worktree Setup

The worktree setup script is the same as single-repo dispatch. All agents work within the monorepo, but in their own branches.

```bash
SPRINT="sprint-01"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

for agent in backend frontend infra services qa data lead design; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done
```

The monorepo advantage: agents can read across package boundaries without any cross-repo coordination. BACKEND can read `packages/shared` types to understand what it is working with, even though DATA owns that territory. No agent needs file system access to a second repo directory.

### Build and Test Commands Differ Per Package

The main operational difference from single-package repos is that build and test commands are package-scoped. Every agent activation prompt must specify the correct commands for the packages in that agent's territory.

```
BACKEND's activation prompt build/test commands:
  Build: cd apps/api && npm run build
         OR: turbo build --filter=api
  Test:  cd apps/api && npm test
         OR: turbo test --filter=api

FRONTEND's activation prompt build/test commands:
  Build: cd apps/web && npm run build
         OR: turbo build --filter=web
  Test:  cd apps/web && npm test
         OR: turbo test --filter=web

DATA's activation prompt build/test commands:
  Build: cd packages/shared && npm run build
         OR: turbo build --filter=shared
  Test:  cd packages/shared && npm test
  Note:  After modifying shared, DATA must also verify that
         apps/api and apps/web still build (they depend on shared).
```

DATA has additional validation responsibility in a monorepo because changes to a shared package can break any dependent package. Include this explicitly in DATA's activation prompt:

```
DATA VALIDATION RULE (monorepo):
After every change to packages/shared, run:
  turbo build --filter=...shared  (build all packages that depend on shared)
  turbo test --filter=...shared   (test all packages that depend on shared)
If any dependent package fails to build or its tests fail, you have introduced
a breaking change. Fix it before proceeding to the next chain.
```

### Shared Dependencies: Node Modules Hoisting

In a JavaScript monorepo with hoisted `node_modules` (Yarn Workspaces, pnpm workspaces, npm workspaces), each agent worktree needs a full install at the root, not just in the package directory.

```bash
# Node.js monorepo: install at root of each worktree
for agent in backend frontend infra services qa data lead design; do
  (cd "$PARENT_DIR/${PROJECT_NAME}-${agent}" && npm install)
  # pnpm: pnpm install
  # yarn: yarn install
done
```

Because `node_modules` is hoisted to the root and the lockfile lives at the root, all agents share the same dependency versions within their worktree. If DATA needs to add a new shared dependency, the lockfile changes. This is a shared-file conflict risk. Handle it the same way other shared-file conflicts are handled: DATA owns the root `package.json` and lockfile changes related to its work, LEAD resolves any conflicts at merge time.

### Go Workspaces

A Go workspace (`go.work`) allows multiple modules in the same directory to reference each other by path. Each module (`services/api`, `services/worker`, `pkg/shared`) has its own `go.mod`.

```
monorepo/
├── go.work
├── services/
│   ├── api/        ← go.mod
│   └── worker/     ← go.mod
└── pkg/
    └── shared/     ← go.mod
```

Territory mapping for a Go workspace:

```
BACKEND:    services/api/internal/handler/, services/api/internal/service/
SERVICES:    services/worker/internal/
DATA:  services/api/internal/store/, pkg/shared/
QA:     **/*_test.go (across all modules)
INFRA:  Dockerfile, Makefile, .github/
LEAD:     docs/, go.work, README.md
```

Go modules auto-download dependencies. No install step needed per worktree. However, if DATA modifies `pkg/shared`, all modules that import it (`services/api`, `services/worker`) must still compile. DATA's validation rule is the same as in the JavaScript case: run `go build ./...` from the workspace root after any change to shared packages.

LEAD owns `go.work` changes. If a new module is added or a module is renamed, that goes in LEAD's territory, not inside any individual module's territory.

---

## 4. Practical Setup

### Multi-Repo Worktree Script

This script handles the worktree setup for a backend + frontend split. Adapt agent names based on which agents actually work in each repo.

```bash
#!/usr/bin/env bash
# setup-multi-repo-worktrees.sh
# Usage: ./setup-multi-repo-worktrees.sh sprint-01

set -euo pipefail

SPRINT="${1:?Usage: $0 <sprint-name>}"

# ---- CONFIGURE THESE ----
BACKEND_DIR="/path/to/backend"
FRONTEND_DIR="/path/to/frontend"
BACKEND_AGENTS="backend data services qa infra lead"
FRONTEND_AGENTS="frontend qa lead"
# -------------------------

setup_worktrees() {
  local repo_dir="$1"
  local agents="$2"
  local repo_name
  repo_name="$(basename "$repo_dir")"
  local parent_dir
  parent_dir="$(dirname "$repo_dir")"

  echo "Setting up worktrees for: $repo_name"
  cd "$repo_dir"

  for agent in $agents; do
    local branch="$SPRINT/$agent"
    local worktree_path="$parent_dir/${repo_name}-${agent}"

    # Create branch if it does not exist
    git branch "$branch" main 2>/dev/null || echo "  Branch $branch already exists"

    # Create worktree if it does not exist
    if [ ! -d "$worktree_path" ]; then
      git worktree add "$worktree_path" "$branch"
      echo "  Created: $worktree_path"
    else
      echo "  Already exists: $worktree_path"
    fi
  done

  echo ""
}

setup_worktrees "$BACKEND_DIR" "$BACKEND_AGENTS"
setup_worktrees "$FRONTEND_DIR" "$FRONTEND_AGENTS"

echo "All worktrees ready."
echo ""
echo "Backend worktrees:"
(cd "$BACKEND_DIR" && git worktree list)
echo ""
echo "Frontend worktrees:"
(cd "$FRONTEND_DIR" && git worktree list)
```

### Teardown Script

```bash
#!/usr/bin/env bash
# teardown-multi-repo-worktrees.sh
# Usage: ./teardown-multi-repo-worktrees.sh sprint-01

set -euo pipefail

SPRINT="${1:?Usage: $0 <sprint-name>}"

BACKEND_DIR="/path/to/backend"
FRONTEND_DIR="/path/to/frontend"
BACKEND_AGENTS="backend data services qa infra lead"
FRONTEND_AGENTS="frontend qa lead"

teardown_worktrees() {
  local repo_dir="$1"
  local agents="$2"
  local repo_name
  repo_name="$(basename "$repo_dir")"
  local parent_dir
  parent_dir="$(dirname "$repo_dir")"

  echo "Removing worktrees for: $repo_name"
  cd "$repo_dir"

  for agent in $agents; do
    local worktree_path="$parent_dir/${repo_name}-${agent}"
    local branch="$SPRINT/$agent"

    git worktree remove "$worktree_path" --force 2>/dev/null \
      && echo "  Removed: $worktree_path" \
      || echo "  Not found: $worktree_path"

    git branch -d "$branch" 2>/dev/null \
      && echo "  Deleted branch: $branch" \
      || echo "  Branch not found or already deleted: $branch"
  done

  echo ""
}

teardown_worktrees "$BACKEND_DIR" "$BACKEND_AGENTS"
teardown_worktrees "$FRONTEND_DIR" "$FRONTEND_AGENTS"

echo "Sprint cleanup complete."
```

### Microservices Setup Script

For 3+ services, the setup script loops over a list of service configurations:

```bash
#!/usr/bin/env bash
# setup-microservices-worktrees.sh
# Usage: ./setup-microservices-worktrees.sh sprint-01

set -euo pipefail

SPRINT="${1:?Usage: $0 <sprint-name>}"

# Define: "repo_path:agents_for_this_repo"
SERVICES=(
  "/path/to/api-gateway:backend qa lead"
  "/path/to/user-service:backend data qa lead"
  "/path/to/order-service:backend data qa lead"
  "/path/to/contracts:data qa lead"
)

for entry in "${SERVICES[@]}"; do
  repo_dir="${entry%%:*}"
  agents="${entry##*:}"
  repo_name="$(basename "$repo_dir")"
  parent_dir="$(dirname "$repo_dir")"

  echo "Setting up: $repo_name  (agents: $agents)"
  cd "$repo_dir"

  for agent in $agents; do
    branch="$SPRINT/$agent"
    worktree_path="$parent_dir/${repo_name}-${agent}"

    git branch "$branch" main 2>/dev/null || true

    if [ ! -d "$worktree_path" ]; then
      git worktree add "$worktree_path" "$branch"
      echo "  Created: $worktree_path"
    else
      echo "  Exists:  $worktree_path"
    fi
  done

  echo ""
done

echo "All microservice worktrees ready."
```

### The Cross-Repo Coordination File

When a sprint spans multiple repos, you need one place that describes the full picture. Create a `DISPATCH.md` in a neutral location — a sprint-docs folder, a shared repo, or the contracts repo. Reference it in every agent's activation prompt.

```markdown
# Sprint-01 Cross-Repo Dispatch

## Sprint Theme
Add rate limiting and pagination to user search, spanning api-gateway, user-service, and search-service.

## Repo Map
- api-gateway:    /path/to/api-gateway
- user-service:   /path/to/user-service
- search-service: /path/to/search-service

## Cross-Service Chain: Bounded search (P1)
Vector: GET /api/users/search
  → api-gateway/src/routes/users.ts
  → user-service/internal/handler/search.go
  → user-service/internal/service/search.go
  → search-service/src/client/elasticsearch.ts
  → Elasticsearch

Signal: Unbounded queries time out Elasticsearch at scale.

Fix sequence:
  1. SERVICES (search-service): Add 30s query timeout and max-results cap
  2. BACKEND-2 (user-service):  Enforce max 50 results in handler
  3. BACKEND-1 (api-gateway):   Add rate limiting middleware (10 req/min per IP)
  4. QA (all):              Integration tests for the full bounded path

## Merge Order
1. search-service: SERVICES → QA → LEAD
2. user-service:   DATA → BACKEND → QA → LEAD
3. api-gateway:    BACKEND → QA → LEAD
```

Each agent's activation prompt includes:
```
CONTEXT: Read the cross-repo dispatch first:
  /path/to/sprint-docs/sprint-01/DISPATCH.md
  This describes the full chain across all three repos.
  Your scope is limited to [your repo and territory].
```

### LEAD's Expanded Role in Multi-Repo Sprints

In a single-repo sprint, LEAD's job is to merge branches in the right order and validate after each merge. In a multi-repo sprint, LEAD also coordinates the merge ORDER across repos.

LEAD-orchestrator (you can think of this as a distinct responsibility even if it is the same agent) does the following:

```
1. Collect completion reports from all agents across all repos
2. Confirm all agents in Repo A are done before starting Repo A merges
3. Merge Repo A branches in the standard order (DATA → BACKEND → ... → LEAD)
4. Run Repo A's full test suite. Confirm it passes.
5. Tag the Repo A release.
6. Then and only then: start Repo B merges.
7. Repeat for each repo in dependency order.
8. Write the cross-repo sprint summary.
```

LEAD's activation prompt for a multi-repo sprint needs to reference all repos and specify the inter-repo sequence:

```
You are LEAD agent for Sprint 01 — cross-repo coordination.

REPOS (merge in this order):
  1. search-service:  /path/to/search-service-lead
  2. user-service:    /path/to/user-service-lead
  3. api-gateway:     /path/to/api-gateway-lead

COMPLETION REPORTS TO COLLECT:
  search-service:  sprint-01/agent-services-completion.md, sprint-01/agent-qa-completion.md
  user-service:    sprint-01/agent-backend-completion.md, sprint-01/agent-data-completion.md, sprint-01/agent-qa-completion.md
  api-gateway:     sprint-01/agent-backend-completion.md, sprint-01/agent-qa-completion.md

MERGE SEQUENCE:
  Do not start any repo's merge until all agents in that repo have submitted completion reports.
  Do not start Repo N+1 until Repo N is merged, its tests pass, and it is tagged.
```

### Agent Completion Reports in Multi-Repo Sprints

When an agent works across multiple repos (QA, LEAD) or when the same codename is used in different repos (two separate BACKEND agents), completion reports must identify the repo:

```markdown
# Completion Report: QA — Sprint 01

## Repo: search-service
- Tests added: 4
- Files modified: tests/rate-limiting.test.ts, tests/timeout.test.ts

## Repo: user-service
- Tests added: 7
- Files modified: internal/handler/search_test.go

## Repo: api-gateway
- Tests added: 3
- Files modified: src/routes/__tests__/users.test.ts

## Cross-Service Integration Tests
- Added: tests/integration/search-end-to-end.test.ts (api-gateway repo)
- Requires: docker-compose.test.yml with all three services running

## P0 Discoveries
None.

## Blockers for Other Agents
None. All test infrastructure is in place for LEAD to proceed.
```

---

## 5. When NOT to Use Multi-Repo Dispatch

Multi-repo dispatch adds coordination overhead. Before setting it up, ask:

**Do the chains actually cross repo boundaries?**

If this sprint's work in the backend does not affect the frontend API contract, dispatch the two repos independently. Two separate single-repo sprints running at the same time, with no coordination between them, is simpler and produces the same result.

**Do the repos have different release cycles?**

If the backend ships weekly and the frontend ships daily, imposing a synchronized dispatch creates artificial coupling. The backend team ends up waiting for the frontend to be ready before they can tag their release, and vice versa. This is the kind of coupling that the separate repos exist to avoid. Keep the dispatches separate.

**Is the contract stable enough for frontend to work from the spec?**

If the backend API shape is changing throughout the sprint, FRONTEND will be blocked or constantly adapting. This is a planning problem, not a dispatch problem. Stabilize the contract first (via ADR, OpenAPI spec, or a draft endpoint), then dispatch frontend agents once the contract is settled.

**Could you use a monorepo instead?**

If you find yourself constantly coordinating across two repos for the same feature, a monorepo is probably the right answer. Multi-repo dispatch is a tool for managing coordination that already exists — it is not a reason to maintain a split that does not serve you.

**The practical rule:**

Start with single-repo dispatch. Run it for several sprints. Expand to multi-repo coordination only when you consistently encounter problems that single-repo dispatch cannot solve. Most projects that think they need multi-repo dispatch just need better execution traces.

---

**Related Documents:**
- [WORKFLOW.md](../core/workflow.md) — Technical workflow for single-repo sprints
- [OPERATORS-GUIDE.md](../guides/operators-guide.md) — Full tutorial for human operators
- [CUSTOMIZATION.md](../guides/customization.md) — Adapt Agent Dispatch for your project
