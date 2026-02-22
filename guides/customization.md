# Customization Guide

> How to adapt Agent Dispatch for your specific project and tech stack

---

## Step 1: Copy Into Your Project

```bash
cp -r agent-dispatch/ your-project/docs/agent-dispatch/
```

Or add as a git submodule:
```bash
cd your-project
git submodule add https://github.com/your-org/agent-dispatch.git docs/agent-dispatch
```

## Step 2: Map Agent Territories to Your Directory Structure

Edit the agent files in `agents/` and replace the default territories with your actual paths.

### Examples by Stack

#### Go + SvelteKit (like ContentOS)

```
BACKEND:    backend/internal/handler/, backend/internal/service/
FRONTEND:    frontend/src/routes/, frontend/src/lib/
INFRA:  Makefile, Dockerfile, .github/
SERVICES:    backend/internal/agent/, backend/internal/worker/
QA:     **/*_test.go, **/*.test.ts
DATA:  backend/internal/store/, backend/internal/model/
LEAD:     docs/, CLAUDE.md, README.md
DESIGN:    design/, design-tokens/, frontend/src/styles/, tailwind.config.*, .storybook/
```

#### Next.js + Prisma (Fullstack TypeScript)

```
BACKEND:    src/app/api/, src/server/
FRONTEND:    src/app/(routes)/, src/components/, src/hooks/
INFRA:  Dockerfile, .github/, next.config.ts, vercel.json
SERVICES:    src/lib/integrations/, src/lib/ai/
QA:     **/*.test.ts, **/*.spec.ts, cypress/
DATA:  prisma/, src/lib/db/, src/types/
LEAD:     docs/, README.md
DESIGN:    design/, src/styles/, tailwind.config.*, .storybook/, **/*.stories.tsx
```

#### Django + React (Python + JavaScript)

```
BACKEND:    backend/views.py, backend/serializers.py, backend/urls.py
FRONTEND:    frontend/src/components/, frontend/src/pages/
INFRA:  Dockerfile, docker-compose.yml, .github/, Makefile
SERVICES:    backend/tasks.py, backend/integrations/
QA:     backend/tests/, frontend/src/__tests__/
DATA:  backend/models.py, backend/migrations/
LEAD:     docs/, README.md
DESIGN:    design/, frontend/src/styles/, frontend/src/theme/
```

#### Rails + Hotwire (Ruby)

```
BACKEND:    app/controllers/, app/services/
FRONTEND:    app/views/, app/javascript/, app/assets/
INFRA:  Dockerfile, .github/, config/deploy/
SERVICES:    app/jobs/, app/mailers/, lib/integrations/
QA:     spec/, test/
DATA:  app/models/, db/migrate/
LEAD:     docs/, README.md
DESIGN:    design/, app/assets/stylesheets/design-system/
```

#### Rust + Yew (or any Rust project)

```
BACKEND:    src/handlers/, src/services/
FRONTEND:    frontend/src/
INFRA:  Dockerfile, .github/, Cargo.toml (deps only)
SERVICES:    src/workers/, src/integrations/
QA:     tests/, benches/
DATA:  src/models/, src/db/, migrations/
LEAD:     docs/, README.md
DESIGN:    design/, frontend/src/styles/, frontend/src/theme/
```

## Step 3: Set Your Build/Test Commands

Update `WORKFLOW.md` post-merge validation with your commands:

```bash
# Go
go build ./... && go test -race ./...

# Node/TypeScript
npm run build && npm test

# Python
python -m pytest && mypy . && ruff check .

# Rust
cargo build && cargo test

# Ruby
bundle exec rails db:migrate && bundle exec rspec

# Multi-service
make build && make test
```

## Step 4: Create Your Project Context File

Every project needs a context file that agents read first. This is your "project brain dump":

```markdown
# [Project Name] - Agent Context

## Overview
[What does this project do? 2-3 sentences]

## Tech Stack
[List all technologies, frameworks, databases]

## Project Structure
[Directory tree with annotations]

## Key Commands
[How to install, run, test, build, deploy]

## Architecture
[How components connect — handlers → services → stores → etc.]

## Key Patterns
[Design patterns used — factory, repository, middleware, etc.]

## Known Issues
[Technical debt, bugs, things to watch out for]
```

If you use Claude Code, this is your `CLAUDE.md`.
For other agents, put it in `README.md` or `docs/CONTEXT.md`.

## Step 5: Customize Wave Order

Default wave order works for most projects:

```
Wave 1: DATA + QA + INFRA + DESIGN  (no dependencies)
Wave 2: BACKEND + SERVICES                     (need stable data layer)
Wave 3: FRONTEND                             (needs DESIGN specs + stable backend)
Wave 4: LEAD                              (needs everything)
```

But customize based on YOUR dependency graph:

```
If frontend is independent (e.g., static site):
  Wave 1: FRONTEND + QA + INFRA
  Wave 2: everything else

If you have no backend (pure frontend):
  Drop BACKEND, SERVICES, DATA
  Use FRONTEND (components) + QA (tests) + INFRA (build) + LEAD (merge)

If backend-only (API, no frontend):
  Drop FRONTEND
  Use DATA + BACKEND + SERVICES + QA + INFRA + LEAD

If monolith (single codebase, no clear layers):
  Split by feature instead of layer:
  BACKEND = Feature A files
  FRONTEND = Feature B files
  SERVICES = Feature C files
  QA = Tests for all
  LEAD = Merge + docs
```

## Step 6: Scale Agent Count

Not every sprint needs all 9 agents. Scale based on scope:

| Sprint Size | Agents | Example |
|-------------|--------|---------|
| Tiny (1-2 tasks) | 1 agent | Single bug fix |
| Small (3-5 tasks) | 3 agents | BACKEND + FRONTEND + LEAD |
| Medium (6-10 tasks) | 5 agents | Add QA + DATA (+ DESIGN if design work) |
| Large (10-20 tasks) | 9 agents | Full roster |
| XL (20+ tasks) | 9+ agents | Split agents by subdomain |

## Step 7: Adapt Completion Report Template

Edit `TEMPLATE-COMPLETION.md` to match your project's needs:

- Add your specific build commands
- Add your test framework output format
- Add metrics you care about (coverage %, bundle size, etc.)
- Add project-specific checklists

---

## Quick Customization Checklist

- [ ] Copied agent-dispatch/ into your project
- [ ] Updated agent files in agents/ with YOUR directory paths
- [ ] Updated build/test commands in WORKFLOW.md
- [ ] Created project context file (CLAUDE.md / CONTEXT.md)
- [ ] Adjusted wave order if needed
- [ ] Ran one small sprint to validate the workflow
- [ ] Shared OPERATORS-GUIDE.md with your team

---

**Legacy codebases:** If adapting for a codebase with no tests, no docs, or spaghetti code, see [LEGACY-CODEBASES.md](legacy-codebases.md) for the modified workflow (Sprint Zero, characterization testing, adjusted critical escalation protocol).

**Related Documents:**
- [OPERATORS-GUIDE.md](operators-guide.md) — Full tutorial
- [METHODOLOGY.md](../core/methodology.md) — Execution traces, chain execution, priority levels
- [LEGACY-CODEBASES.md](legacy-codebases.md) — Adapted workflow for legacy codebases
- [agents/](../agents/) — Role definitions to customize
- [WORKFLOW.md](../core/workflow.md) — Workflow to customize
