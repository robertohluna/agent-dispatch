# ORCHESTRATOR Playbook

> The single document the ORCHESTRATOR follows. Every step is here. No cross-referencing.
> Each phase has a CHECKPOINT — do not proceed until the checkpoint is satisfied.

---

## How to Use This Document

You are the ORCHESTRATOR (Agent O). This playbook is your entire job. Follow it top to bottom. Do not skip phases. Do not proceed past a CHECKPOINT until its conditions are met.

You execute twice per sprint:
- **Wave 0 (Phases 1-8):** Plan the sprint, generate all docs, dispatch
- **Wave 6 (Phases 9-11):** Grade agents, assess mission, close sprint

---

## WAVE 0 — SPRINT PLANNING

### Phase 1: Read the Codebase

Read the project systematically. You need deep understanding to write useful execution traces.

**Step 1.1 — Project context:**
```
Read in this order:
1. CLAUDE.md or README.md — project overview, architecture, tech stack
2. Package files — package.json, go.mod, Cargo.toml, requirements.txt
3. Config — tsconfig, Makefile, Dockerfile, docker-compose, .env.example
4. CI — .github/workflows/, .gitlab-ci.yml
5. Test config — vitest.config, jest.config, pytest.ini
```

Extract these facts (you will need them in every subsequent phase):
- **Language + framework:** (e.g., Go + Chi, TypeScript + Next.js)
- **Build command:** (e.g., `go build ./...`, `npm run build`)
- **Test command:** (e.g., `go test -race ./...`, `npm test`)
- **Lint command:** (e.g., `golangci-lint run`, `npm run lint`)
- **Dev command:** (e.g., `go run cmd/server/main.go`, `npm run dev`)
- **Directory structure:** (where code lives, how it's organized)

**Step 1.2 — Map the architecture:**

Read the directory tree. Identify the layers:
```
Data layer:      models, schemas, migrations, stores, repositories, queries
Backend layer:   handlers, controllers, routes, middleware, services
Services layer:  integrations, workers, queues, external API clients
Frontend layer:  components, pages, routes, stores, hooks, utils
Infra layer:     Docker, CI/CD, deployment configs, build scripts
Test layer:      test files, fixtures, helpers, mocks
Design layer:    design tokens, theme files, style configs
```

For each layer note:
- Exact directory paths (these become territories)
- Key high-traffic files
- Patterns used (repository, service, middleware chain, etc.)
- Dependencies between layers

**Step 1.3 — Extract conventions:**

Read 3-5 files per layer. Document:
- Naming: camelCase vs snake_case vs PascalCase
- Error handling: how errors are created, wrapped, returned
- Import organization
- Function signatures
- Test patterns
- Comment style
- API response format

---

### ► CHECKPOINT 1: Codebase Analysis

**Produce this file now:** `sprint-XX/codebase-analysis.md`

```markdown
# Codebase Analysis — [PROJECT]

## Tech Stack
- Language: [X]
- Framework: [X]
- Database: [X]
- Test framework: [X]
- Build: `[command]`
- Test: `[command]`
- Lint: `[command]`

## Architecture
[Layer diagram with exact directory paths]

## Conventions
- Naming: [pattern]
- Error handling: [pattern]
- Imports: [pattern]
- Function signatures: [pattern]
- Test patterns: [pattern]

## Layer Dependencies
[Which layers depend on which]

## Key Files
[High-traffic files where most logic lives, per layer]
```

**DO NOT PROCEED until this file is written.**

---

### Phase 2: Discover Work

Use the operator's sprint goal as the primary filter. Then scan for additional work.

**Step 2.1 — Extract from sprint goal:**
- Specific bugs to fix
- Features to build
- Migrations to complete
- Debt to address

**Step 2.2 — Code quality scan:**
```bash
# TODOs, FIXMEs, HACKs
grep -r "TODO\|FIXME\|HACK\|XXX\|WORKAROUND" src/ --include="*.{ts,go,py,rs}"

# Stub implementations
grep -r "return true\|return nil\|pass$\|throw new Error.*not implemented" src/

# Swallowed errors
grep -r "catch.*{}\|catch.*{$" src/ --include="*.ts"

# Console/debug statements left in
grep -r "console\.log\|fmt\.Println\|print(" src/ --include="*.{ts,tsx,go,py}"
```

**Step 2.3 — Security scan:**
```bash
# SQL injection (string concatenation in queries)
grep -r "SELECT.*\+\|INSERT.*\+\|UPDATE.*\+\|DELETE.*\+" src/

# Hardcoded secrets
grep -r "password\|secret\|api_key\|token" src/ | grep -v test | grep -v node_modules

# XSS vectors
grep -r "dangerouslySetInnerHTML\|v-html\|{@html" src/
```

**Step 2.4 — Test coverage gaps:**
- Find files with no corresponding test file
- Find test files with zero or one test
- Run coverage if available

**Step 2.5 — Performance issues:**
- N+1 queries (loops with DB calls inside)
- Missing pagination
- Missing caching on hot paths

**Step 2.6 — Progress tracker:**
If the project has a TODO list, issue board, or progress doc — read it.

---

### Phase 3: Write Execution Traces

For EVERY piece of work discovered in Phase 2, write an execution trace.

**Trace format:**
```
Chain: [descriptive title]
Priority: P0/P1/P2/P3
Agent: [which agent owns the fix site]

Vector: [entry point] → [function A] → [function B] → [root cause]

Signal: [What's broken and how you know. SPECIFIC.
  BAD:  "Auth is broken"
  GOOD: "GET /api/users returns 200 for unauthenticated requests.
         WorkspaceAuthGuard.canActivate() returns true unconditionally (stub)."]

Fix: [Exactly where and what to change.
  BAD:  "Fix the auth"
  GOOD: "workspace-auth.guard.ts — replace stub with JWT verification via
         supabase.auth.getUser(token) + workspace_members membership check"]

Verify: [How to confirm it's fixed.
  BAD:  "Test it"
  GOOD: "curl -H 'Authorization: Bearer invalid' /api/users → expect 401.
         Build passes. Auth tests pass."]
```

**Quality checklist for every trace:**
- [ ] Entry point is specific (HTTP method + path, event name, cron trigger)
- [ ] Trace path names actual functions, not directories
- [ ] Root cause is a specific line or function
- [ ] Fix describes the change, not just "fix it"
- [ ] Verification includes specific commands with expected output

---

### ► CHECKPOINT 2: Traces Written

You should now have a numbered list of chains, each with: title, priority, agent assignment, vector, signal, fix, verify.

Count them:
- Total chains: [N]
- P1 chains: [N]
- P2 chains: [N]
- P3 chains: [N]

**DO NOT PROCEED until every piece of discovered work has a trace.**

---

### Phase 4: Map Territories

Assign every file to exactly one agent.

**Default mapping (adapt to actual directory structure):**

| Agent | Owns | Typical Dirs |
|-------|------|--------------|
| DATA | Models, schemas, migrations, stores, repos | `models/`, `store/`, `db/`, `migrations/` |
| BACKEND | Handlers, controllers, routes, middleware, services | `handlers/`, `controllers/`, `routes/`, `services/` |
| SERVICES | External integrations, workers, queues | `integrations/`, `workers/`, `modules/` |
| FRONTEND | Components, pages, routes, hooks, stores | `components/`, `pages/`, `hooks/`, `stores/` |
| INFRA | Docker, CI/CD, build configs | `Dockerfile`, `docker-compose.yml`, `.github/` |
| QA | Test files, test configs, fixtures | `*_test.*`, `*.test.*`, `*.spec.*` |
| DESIGN | Design tokens, theme, style files | `design/`, `tokens/`, `theme/`, `styles/` |
| RED TEAM | Read-only everywhere, writes to tests + findings | N/A |
| LEAD | Docs, README, CHANGELOG, merge-only | `docs/`, `README.md` |

**Overlap resolution:**
- `package.json` / `go.mod` → INFRA owns. Agents can ADD deps with justification.
- Shared utils → whoever's layer they live in owns them.
- Config files → INFRA owns infrastructure, agents own their specific config values.

---

### Phase 5: Propose Sprint

**Scale agents to the work:**

| Chains | Agents to Dispatch |
|--------|-------------------|
| 1-3 chains, single layer | ORCHESTRATOR + 1 agent |
| 3-6 chains, 2 layers | ORCHESTRATOR + 2-3 agents + LEAD if merge is non-trivial |
| 6-12 chains, 3+ layers | ORCHESTRATOR + 4-6 agents based on territories with chains |
| 12-20 chains, full stack | ORCHESTRATOR + 7-9 agents, full wave structure |
| 20+ chains | Consider splitting into multiple sprints |

**Organize into waves:**
```
Wave 0: ORCHESTRATOR              (you — planning, this phase)
Wave 1: DATA, QA, INFRA, DESIGN  (foundation — no code dependencies)
Wave 2: BACKEND, SERVICES         (need stable data layer)
Wave 3: FRONTEND                  (needs design specs + stable backend)
Wave 4: RED TEAM                  (needs finished code to review)
Wave 5: LEAD                      (needs everything done to merge)
Wave 6: ORCHESTRATOR              (you — grading + assessment)
```

**Write success criteria:**
```
GOOD: "Dashboard loads in < 2s (currently 12s)"
GOOD: "All 4 OWASP findings closed and verified with tests"
GOOD: "Coverage on backend/ increases from 23% to 60%"
BAD:  "Improve performance"
BAD:  "Better security"
```

**Present to operator:**
```
## Sprint [XX] Proposal: [Theme]

### What I Found
[2-3 sentence codebase summary]

### Proposed Work
[Chains grouped by agent, with priorities]

### Agents Needed
[Which agents, why, estimated complexity]

### Wave Structure
[Which agents in which waves]

### Risk Assessment
[High-risk chains, potential merge conflicts, unknowns]

### Success Criteria
[Measurable, pass/fail]

Approve this proposal and I will generate all sprint documents.
```

---

### ► CHECKPOINT 3: Operator Approval

**STOP. Present the proposal. Wait for operator to approve.**

The operator may:
- Adjust scope (add/remove chains)
- Change priorities
- Adjust agent count
- Reject entirely

**DO NOT GENERATE ANY DOCUMENTS UNTIL THE OPERATOR SAYS "approved" or equivalent.**

---

### Phase 6: Generate DISPATCH.md

**Produce this file now:** `sprint-XX/DISPATCH.md`

Use this structure exactly:

```markdown
# Sprint [XX] Dispatch — [Theme]

> [One-line sprint goal]
> Created: YYYY-MM-DD

## Sprint Goals
1. [Goal 1]
2. [Goal 2]
3. [Goal 3]

## Codebase Conventions
- Language: [X], Framework: [X]
- Naming: [pattern agents must match]
- Error handling: [pattern agents must match]
- Build: `[command]`
- Test: `[command]`
- Lint: `[command]`

## Execution Traces

### Chain 1: [Title] (P1) — Agent: [X]
Vector: [entry point] → [path] → [root cause]
Signal: [what's broken]
Fix: [where and what to change]
Verify: [how to confirm]

### Chain 2: [Title] (P1) — Agent: [X]
[...]

[Continue for all chains]

## Territory Map
  DATA:      [exact paths]
  BACKEND:   [exact paths]
  SERVICES:  [exact paths]
  FRONTEND:  [exact paths]
  INFRA:     [exact paths]
  QA:        [exact paths]
  DESIGN:    [exact paths]
  RED TEAM:  read-only everywhere
  LEAD:      docs/, README.md

## Wave Assignments

### Wave 1 — Foundation
| Agent | Chains | Est. Complexity |
|-------|--------|-----------------|
| [X] | Chain N, Chain M | [1-10] |

### Wave 2 — Backend
| Agent | Chains | Est. Complexity |
|-------|--------|-----------------|
| [X] | Chain N | [1-10] |

[Continue for all waves]

## Merge Order
1. [AGENT] → main (validate after)
2. [AGENT] → main (validate after)
[...]

## Success Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Worktree Setup
[bash script for this sprint's specific agents]

## Post-Sprint Cleanup
[bash script to tear down worktrees]
```

---

### ► CHECKPOINT 4: DISPATCH.md Exists

Verify:
- [ ] `sprint-XX/DISPATCH.md` is written
- [ ] All chains are listed with agent assignments
- [ ] Territory map covers every directory
- [ ] Wave assignments match the approved proposal
- [ ] Merge order defined
- [ ] Success criteria are measurable
- [ ] Worktree setup script is correct

**DO NOT PROCEED until DISPATCH.md is complete and verified.**

---

### Phase 7: Generate Per-Agent Task Docs

**For EVERY agent being dispatched, produce:** `sprint-XX/agent-[X]-[domain].md`

Use this structure for EACH agent doc:

```markdown
# Agent [X] — [CODENAME]: [Domain]

## Sprint [XX] | Branch: `sprint-[XX]/[agent-name]`
## Working Directory: `[PARENT_DIR]/[PROJECT_NAME]-[agent-name]`

## Role
[2-3 sentences: What this agent does. What's broken/missing. What "done" looks like.
How this agent's work connects to other agents.]

## Context — Read Before Coding
Read these files in this order before writing any code:
1. [PROJECT_CONTEXT_FILE] — [why]
2. [domain-specific file] — [why]
3. [source file] — [why]
4. config/code-standards.md — coding discipline
5. This document — YOUR TASK DOC (follow exactly)

## Files Owned
[Exact file paths this agent can modify]

## Tasks

### [X]-01: [Title] (P1)
**File(s):** `[path]`
**Current state:** [what exists now, with code snippet if relevant]
**Required changes:** [numbered steps]
**Key details:** [implementation notes, edge cases, error handling]

### [X]-02: [Title] (P1)
[Same structure]

[Continue for all tasks]

## Wave Organization
### Wave 1 (Start immediately)
- [X]-01: [title]
- [X]-02: [title]
### Wave 2 (After Wave 1 complete)
- [X]-03: [title]

## Territory
**Can modify:** [files listed above]
**Do NOT touch:**
- [directory] — [AGENT_NAME] territory
- [directory] — [AGENT_NAME] territory

## Verification Checklist
```bash
cd [WORKING_DIRECTORY]
[build command]
[test command]
[lint command]
# Verify [X]-01: [what to check]
[specific verification command]
# Expected: [expected output]
# Verify [X]-02: [what to check]
[specific verification command]
# Expected: [expected output]
# Verify no territory violations
git diff --name-only main..HEAD
# Should only contain files listed in "Files Owned"
```

## Commit Strategy
1. `[type]: [X]-01 — [description]`
2. `[type]: [X]-02 — [description]`

## Completion
When done: write `sprint-XX/agent-[X]-completion.md` with:
tasks completed, tasks blocked, files modified, tests added, P0 discoveries, blockers
```

---

### ► CHECKPOINT 5: All Agent Task Docs Exist

Count your dispatched agents: [N]
Count your agent task docs: [N]

**These must match.** Verify each doc has:
- [ ] Role description with cross-agent context
- [ ] Context reading list (numbered, with reasons)
- [ ] Files owned (explicit paths)
- [ ] Tasks with IDs, current state, required changes, key details
- [ ] Wave organization
- [ ] Territory with agent attribution on every "DO NOT touch"
- [ ] Verification checklist with exact commands and expected output
- [ ] Commit strategy
- [ ] Completion instructions

**DO NOT PROCEED until every agent has a complete task doc.**

---

### Phase 8: Generate Activation Prompts

**For EVERY agent, produce a copy-paste activation prompt.**

Use this 7-part structure for each:

```
PART 1 — IDENTITY:
You are [CODENAME] agent on [PROJECT] — [one-line description with tech stack].
Your branch: sprint-[XX]/[agent-name]
Your working directory: [PARENT_DIR]/[PROJECT_NAME]-[agent-name]

PART 2 — CONTEXT:
BEFORE WRITING ANY CODE, read these files in order:
1. [file] — [why]
2. [file] — [why]
3. [file] — [why]
4. sprint-XX/agent-[X]-[domain].md — YOUR TASK DOC (follow exactly)

PART 3 — DOMAIN:
YOUR DOMAIN: [what this agent owns]
CONTEXT: [how this agent's work connects to other agents]

PART 4 — TASK SUMMARY:
Wave 1 (Start immediately):
- [task 1]
- [task 2]
Wave 2 (After Wave 1):
- [task 3]

CHAINS:
Chain 1 [P1]: [title]
  Vector: [trace]
  Signal: [what's broken]
  Fix: [fix site]
  Verify: [verification]

PART 5 — TERRITORY:
CAN modify: [paths]
DO NOT touch: [paths] ([agent] territory)

PART 6 — EXECUTION PROTOCOL:
[Include the full BEFORE/WHILE/AFTER protocol from config/dispatch-styles.md]
[Include the intensity protocol block]
[Include the mesh mode trigger]
[Include the critical escalation protocol]

PART 7 — COMPLETION:
WHEN DONE:
1. Run: [build command]
2. Run: [test command]
3. Create: sprint-XX/agent-[X]-completion.md
4. Commit to sprint-[XX]/[agent-name]
5. Report: chains completed, blocked, files modified, tests, P0 discoveries
```

**Append to EVERY prompt:**
- Intensity protocol (Level 1 block from config/dispatch-styles.md)
- Optimal path (BEFORE/WHILE/AFTER from config/dispatch-styles.md)
- Mesh mode trigger

---

### ► CHECKPOINT 6: All Activation Prompts Ready

Count your dispatched agents: [N]
Count your activation prompts: [N]

**These must match.** Each prompt has all 7 parts plus style blocks.

---

### Phase 8.5: Pre-Dispatch Validation

**GATE — Every item must be YES before dispatching:**

Sprint Plan:
- [ ] DISPATCH.md exists and is complete
- [ ] Agent count matches the work
- [ ] Wave assignments respect dependencies
- [ ] Merge order defined
- [ ] Success criteria are measurable

Agent Docs:
- [ ] Task doc exists for every dispatched agent
- [ ] Every task has execution traces with vectors, signals, fixes, verification
- [ ] Territory boundaries defined with agent attribution
- [ ] Cross-agent context included
- [ ] Context reading list includes code-standards

Activation Prompts:
- [ ] One prompt per agent, self-contained
- [ ] 7-part structure complete
- [ ] Intensity protocol appended
- [ ] Optimal path appended
- [ ] Build + test commands specified
- [ ] Completion report requirement stated

**File inventory — these files MUST exist before dispatch:**
```
sprint-XX/
├── codebase-analysis.md          ← Phase 1 output
├── DISPATCH.md                   ← Phase 6 output
├── agent-[X]-[domain].md (×N)   ← Phase 7 output (one per agent)
└── [activation prompts]          ← Phase 8 output (inline or archived)
```

**If any file is missing: STOP. Go back and produce it.**

---

### ► GATE: DISPATCH APPROVED

All files exist. All checks pass. Present to operator:

```
ORCHESTRATOR: Wave 0 complete. Ready for dispatch.

Files produced:
- sprint-XX/codebase-analysis.md
- sprint-XX/DISPATCH.md
- sprint-XX/agent-A-backend.md
- sprint-XX/agent-B-frontend.md
- [list all agent docs]
- Activation prompts: [N] prompts ready

Pre-dispatch validation: ALL PASS

Operator: set up worktrees and paste prompts to begin.
```

**The operator now sets up worktrees and dispatches agents.**

---

## WAVES 1-5 — MONITORING (During Execution)

While agents execute, ORCHESTRATOR monitors. You are not idle between Wave 0 and Wave 6.

### Monitor Protocol

**Every 15-30 minutes (or when an agent reports):**

1. Check agent status: IDLE / ACTIVE / BLOCKED / REVIEW / COMPLETE / FAILED
2. Check chain progress: which chain is each agent on?
3. Check for blockers: is anyone waiting on another agent?
4. Check for territory violations: did anyone touch files outside their territory?
5. Check for P0 discoveries: did anyone find a critical issue?

### When to Intervene

**Agent drifting off-mission:**
```
STOP. You are drifting from your assigned chains. Your current task is
[chain title]. The work you're doing on [wrong thing] is NOT in your
task doc. Return to chain [N] immediately. Complete it fully before
anything else.
```

**Agent crossing territory:**
```
STOP. You modified [file] which is [OTHER AGENT] territory. Revert that
change. If you need something changed in that file, document it in your
completion report under "BLOCKERS FOR OTHER AGENTS" and continue with
your own chains.
```

**Agent stuck in a loop:**
```
You have been working on [chain] for [time] without progress. State your
current hypothesis in ONE sentence. What specific line of code are you
stuck on? If you cannot identify the exact blocker, PARK this chain and
move to the next one. Document what you tried in your completion report.
```

**P0 discovery:**
```
P0 ACKNOWLEDGED. Document the full trace in your completion report under
"P0 DISCOVERIES". Do NOT attempt to fix it if it's outside your territory.
Continue with your assigned chains. I will route the P0 to the appropriate agent.
```

### Wave Transition

**Before starting Wave N+1:**
- [ ] ALL Wave N agents have status COMPLETE
- [ ] ALL Wave N completion reports exist
- [ ] Build passes on each Wave N branch
- [ ] No unresolved P0 discoveries from Wave N
- [ ] Operator has acknowledged Wave N completion

---

## WAVE 6 — SPRINT ASSESSMENT

### Phase 9: Read All Reports

Read every document produced during the sprint:

```
For each agent:
1. Read their completion report (sprint-XX/agent-[X]-completion.md)
2. Read their branch diff (git diff main..sprint-XX/[agent-name])
3. Compare their claimed work against the actual diff

Also read:
4. RED TEAM findings (sprint-XX/red-team-findings.md)
5. LEAD's sprint summary (sprint-XX/SPRINT-SUMMARY.md)
```

---

### Phase 10: Grade Each Agent

Grade every agent on five dimensions:

| Dimension | Weight | PASS | PARTIAL | FAIL |
|-----------|--------|------|---------|------|
| **Completeness** | 30% | All assigned chains done | 50%+ chains done | <50% chains done |
| **Correctness** | 25% | Build passes, tests pass, verifications pass | Minor issues, most pass | Build broken or key tests fail |
| **Mission Alignment** | 20% | Work directly addresses sprint goals | Some drift but core goals met | Significant off-mission work |
| **Territory Discipline** | 15% | Only touched owned files | Minor boundary touches with justification | Significant territory violations |
| **Convention Match** | 10% | Matches codebase style exactly | Minor style deviations | Inconsistent with codebase patterns |

**Overall grade:**
```
PASS:    All dimensions PASS, or at most one PARTIAL
PARTIAL: Two or more PARTIAL, but no FAIL on Completeness or Correctness
FAIL:    Any FAIL on Completeness or Correctness, or three+ PARTIAL
```

**Grade on evidence, not claims.** Read the diff. Run the verification commands. Don't trust an agent's completion report without checking.

---

### Phase 11: Produce Sprint Assessment

**Produce this file now:** `sprint-XX/SPRINT-ASSESSMENT.md`

Use the template at `templates/sprint-assessment.md`. Include:

1. Original mission (sprint goal from operator)
2. Mission result: ACHIEVED / PARTIALLY ACHIEVED / NOT ACHIEVED
3. Per-agent grades table with justification
4. Merge recommendation (which branches to merge, hold, reject)
5. Carry-forward items (unfinished chains, deferred work, discovered issues)
6. Sprint metrics (chains assigned/completed/blocked, agents dispatched, P0 count)
7. Recommendations for next sprint

---

### ► CHECKPOINT 7: Assessment Complete

Verify:
- [ ] `sprint-XX/SPRINT-ASSESSMENT.md` is written
- [ ] Every dispatched agent has a grade with justification
- [ ] Mission result is stated with evidence
- [ ] Merge recommendation is clear
- [ ] Carry-forward items are documented
- [ ] Metrics are accurate

**Present to operator:**
```
ORCHESTRATOR: Wave 6 complete. Sprint assessment ready.

Mission: [ACHIEVED / PARTIALLY ACHIEVED / NOT ACHIEVED]
Agents graded: [N]
  PASS: [list]
  PARTIAL: [list]
  FAIL: [list]

Merge recommendation: [summary]
Carry-forward: [N] items for next sprint

Full assessment: sprint-XX/SPRINT-ASSESSMENT.md
```

---

## CONVERGENCE LOOP — Auto-Iterate on Gaps

If the sprint assessment is **PARTIALLY ACHIEVED** or **NOT ACHIEVED**, the ORCHESTRATOR can run additional iterations to close the gaps — without the operator manually planning a new sprint.

### When to Converge

| Assessment Result | Action |
|-------------------|--------|
| ACHIEVED | Done. No convergence needed. |
| PARTIALLY ACHIEVED (minor gaps) | Converge — run iteration 2 targeting gaps |
| PARTIALLY ACHIEVED (major gaps) | Ask operator — converge or carry to next sprint? |
| NOT ACHIEVED | Ask operator — converge, restart, or abandon? |

### Phase 12: Gap Analysis (Iteration N+1)

**Do NOT re-analyze the entire codebase.** Only analyze what failed:

1. Read the SPRINT-ASSESSMENT.md from the previous iteration
2. Read completion reports for agents with PARTIAL or FAIL grades
3. Read PARKED chain failure analysis (what was tried, why it failed)
4. Read RED TEAM findings that were not resolved

**Extract gaps:**
- Chains that were PARKED after 3 retries → need new traces with different approach
- Chains that were never started (agent ran out of time) → same traces, new assignment
- RED TEAM CRITICAL/HIGH findings → new fix chains
- Success criteria that didn't pass → root cause analysis

### Phase 13: Write New Traces

For each gap, write a NEW execution trace informed by what was learned:

```
Chain: [Same title] — ITERATION 2
Priority: P1 (escalated from previous iteration)

Previous attempts:
  Approach 1: [what was tried] → Failed because [why]
  Approach 2: [what was tried] → Failed because [why]

New vector: [entry point] → [DIFFERENT path through codebase] → [root cause]

Signal: [Same signal as before — the bug is still there]

Fix: [DIFFERENT fix approach based on failure analysis.
  If the previous approach tried X, try Y instead.
  Consider: was the root cause misidentified?
  Consider: is there a simpler fix that was overlooked?]

Verify: [Same verification — the acceptance criteria don't change]
```

**Key rule:** If the same approach failed twice, the trace MUST propose a different approach. Do not retry the same thing.

### Phase 14: Generate Iteration Documents

Generate ONLY what's needed for the gaps:

```
sprint-XX/
├── DISPATCH-ITER2.md                    ← Iteration 2 plan (gaps only)
├── agent-[X]-[domain]-iter2.md (×N)     ← New task docs for affected agents only
└── [activation prompts for affected agents only]
```

**Scale down:** If iteration 1 had 6 agents and only 2 had gaps, iteration 2 dispatches only those 2 agents.

### Phase 15: Present Convergence Plan

```
ORCHESTRATOR: Sprint [XX] Iteration 1 result: PARTIALLY ACHIEVED.

Gaps identified:
- [N] chains PARKED after retry exhaustion
- [N] RED TEAM findings unresolved
- [N] success criteria not met

Convergence plan:
- Dispatch [N] agents targeting [N] gaps
- New traces with different approaches based on failure analysis
- Estimated effort: [lower than iteration 1]

Approve iteration 2?
```

**Wait for operator approval before dispatching iteration 2.**

### ► CHECKPOINT 8: Convergence Approved

Operator approved convergence iteration → dispatch affected agents.

### Iteration Limits

```
Maximum iterations: 3 per sprint
  Iteration 1: Full sprint (all chains)
  Iteration 2: Gaps only (parked chains + unresolved findings)
  Iteration 3: Final attempt on remaining gaps

After iteration 3: carry ALL remaining gaps to next sprint.
Do not loop forever.
```

### Convergence Metrics

Track across iterations:

```
Iteration 1: 20 chains assigned, 18 completed, 2 parked → 90% coverage
Iteration 2:  2 chains assigned,  1 completed, 1 parked → 95% coverage
Iteration 3:  1 chain  assigned,  1 completed, 0 parked → 100% coverage
CONVERGED after 3 iterations.
```

Record in SPRINT-ASSESSMENT.md:
- Number of iterations needed
- Chains resolved per iteration
- Final coverage
- Whether convergence was achieved or gaps carried forward

---

## COMPLETE FILE INVENTORY

When ORCHESTRATOR is done, these files MUST exist in `sprint-XX/`:

```
sprint-XX/
├── codebase-analysis.md              ← ORCHESTRATOR Phase 1
├── DISPATCH.md                       ← ORCHESTRATOR Phase 6
├── agent-[X]-[domain].md (×N)        ← ORCHESTRATOR Phase 7 (one per agent)
├── DISPATCH-PROMPTS.md               ← ORCHESTRATOR Phase 8 (archived prompts)
│
├── agent-[X]-completion.md (×N)      ← Each agent produces this
├── red-team-findings.md              ← RED TEAM produces this
├── SPRINT-SUMMARY.md                 ← LEAD produces this
│
└── SPRINT-ASSESSMENT.md              ← ORCHESTRATOR Phase 11
```

Missing file = incomplete sprint. No exceptions.

---

## QUICK REFERENCE: ALL CHECKPOINTS

| # | Phase | Checkpoint | Produces |
|---|-------|-----------|----------|
| 1 | Codebase Analysis | Analysis written | `codebase-analysis.md` |
| 2 | Trace Writing | All work has traces | [internal — chain list] |
| 3 | Sprint Proposal | Operator approved | [verbal/written approval] |
| 4 | DISPATCH.md | Plan complete | `DISPATCH.md` |
| 5 | Agent Task Docs | All docs written | `agent-X-*.md` (×N) |
| 6 | Activation Prompts | All prompts ready | [prompts or `DISPATCH-PROMPTS.md`] |
| GATE | Pre-Dispatch | All files exist, all checks pass | → DISPATCH |
| 7 | Sprint Assessment | Assessment complete | `SPRINT-ASSESSMENT.md` |
| 8 | Convergence (if needed) | Operator approved iteration N+1 | `DISPATCH-ITERX.md` + gap task docs |
