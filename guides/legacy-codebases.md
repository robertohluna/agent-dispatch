# Agent Dispatch on Legacy Codebases

> How to run multi-agent sprints on codebases with no tests, no docs, spaghetti code, deprecated dependencies, and zero CI/CD. This is where most real-world codebases live.

---

## The Reality Check

Greenfield codebases are a luxury. Most of the time you inherit something else: a PHP monolith from 2009, a Node app where every route is 400 lines, a Python service with database credentials hardcoded into ten different files, or a Go codebase that works but nobody can explain why.

Agent Dispatch was designed for these situations. The core methodology doesn't change — execution traces, chain execution, priority levels — but the **order of operations changes completely**. On a greenfield project, you dispatch agents to build. On a legacy project, you dispatch agents to understand before you do anything else. Moving fast in a legacy codebase without that understanding is how you break things that were somehow working.

This guide covers what that looks like in practice.

---

## 1. Codebase Archaeology

Before any agent writes a single line, you need a map. Not full documentation — just enough to know what you're standing on.

### Dependency Mapping

Understand what calls what. The goal is to find the actual dependency graph, not the one someone told you about.

**JavaScript / TypeScript:**
```bash
# madge: visualize module dependencies
npx madge --circular src/          # Find circular dependencies
npx madge --image graph.png src/   # Visualize the whole graph
npx madge --json src/ > deps.json  # Machine-readable output

# For circular dependency count — high number = deep trouble
npx madge --circular src/ | wc -l
```

**Go:**
```bash
# Built-in tools
go vet ./...                        # Catch suspicious constructs
go list -m all                      # All modules and versions
go mod graph | head -50             # Dependency graph (can be enormous)

# Find import cycles
go build ./... 2>&1 | grep "import cycle"
```

**PHP:**
```bash
# phpstan: static analysis
composer require --dev phpstan/phpstan
vendor/bin/phpstan analyse src/ --level=0   # Start at lowest level
vendor/bin/phpstan analyse src/ --level=5   # Level 5 is already revealing

# dephpend: dependency graph
composer require --dev mihaeu/dephpend
vendor/bin/dephpend text src/
```

**Python:**
```bash
# pydeps: visualize dependencies
pip install pydeps
pydeps your_package/ --max-bacon=3  # Limit depth or it's unreadable

# Find circular imports
pip install isort
isort --check-only --diff .
```

**Ruby:**
```bash
# bundler-audit: known vulnerabilities in dependencies
gem install bundler-audit
bundle-audit check --update

# rubocop for code quality signals
gem install rubocop
rubocop --format json > rubocop-report.json
```

What you're looking for:
- **Circular dependencies** — code that depends on itself. Agents cannot safely modify these without understanding the full loop.
- **God objects / god files** — one file that everything imports. Changes here cascade everywhere.
- **Unexpected cross-layer dependencies** — a handler importing directly from the database layer, skipping the service layer entirely.

### Hotspot Analysis

Which files change most frequently? These are either the heart of the system (important, well-maintained) or the source of constant fires (fragile, poorly understood). Either way, they matter.

```bash
# Files changed most often in the last 6 months
git log --format=format: --name-only --since="6 months ago" \
  | sort | uniq -c | sort -rn | head -20

# Same, but limited to a specific directory
git log --format=format: --name-only --since="6 months ago" -- src/ \
  | sort | uniq -c | sort -rn | head -20

# Files with the most authors (high churn + many authors = instability)
git log --format="%an" --name-only -- . \
  | awk 'NF==1 {file=$0} NF>1 {print file, $0}' \
  | sort -u | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Files changed most in commits that touched many files (big messy commits)
git log --name-only --oneline | awk '
  /^[0-9a-f]/ { count=0; next }
  NF { files[NR]=$0; count++ }
  count>10 { for(i=NR-count+1; i<=NR; i++) print files[i] }
' | sort | uniq -c | sort -rn | head -20
```

The output tells you where to point your BACKEND and DATA agents. High-churn files are where you'll find the most bugs and the most risk. They're also where your characterization tests matter most.

### Dead Code Detection

Legacy codebases accumulate dead code. Functions nobody calls. Routes that were replaced two years ago and forgotten. Entire modules that haven't been imported since a refactor.

**JavaScript / TypeScript:**
```bash
# ts-prune: find unused exports
npx ts-prune

# knip: comprehensive dead code detection
npx knip

# ESLint with no-unused-vars (basic, but ships with most projects)
npx eslint src/ --rule '{"no-unused-vars": "error"}' --format json
```

**Go:**
```bash
# deadcode: official Go dead code analyzer
go install golang.org/x/tools/cmd/deadcode@latest
deadcode -test ./...

# unused (stricter)
go install honnef.co/go/tools/cmd/unused@latest
unused ./...
```

**Python:**
```bash
pip install vulture
vulture src/ --min-confidence 80

# Also useful: coverage gaps reveal untested (often dead) code
pip install coverage
coverage run -m pytest
coverage report --sort=cover | head -30   # Lowest covered files first
```

**PHP:**
```bash
vendor/bin/phpstan analyse --level=6 src/ 2>&1 | grep "is unused\|is never used\|is never called"
```

**Ruby:**
```bash
gem install debride
debride app/ lib/
```

Do NOT delete dead code in Sprint Zero. Just catalog it. Dead code removal belongs in a later sprint after you have characterization tests. What looks dead is sometimes load-bearing in ways that aren't obvious.

### Secret Scanning

Legacy codebases love hardcoded credentials. Before any agent reads any code — before you yourself read any code — run a secret scanner. If you find live credentials, rotate them immediately. This is non-negotiable.

```bash
# trufflehog: finds secrets with high accuracy (fewer false positives)
# Install
brew install trufflesecurity/trufflehog/trufflehog  # macOS
pip install trufflehog                               # or pip

# Scan the whole repo including git history
trufflehog git file://. --only-verified

# Scan current files only (faster)
trufflehog filesystem . --only-verified

# gitleaks: fast scanner, good for CI integration
brew install gitleaks                # macOS
gitleaks detect --source . -v       # Scan current state
gitleaks detect --source . -v --log-opts="--all"  # Include all history

# detect-secrets: Python-based, good for pre-commit integration
pip install detect-secrets
detect-secrets scan > .secrets.baseline
detect-secrets audit .secrets.baseline
```

Secret scan results go directly to your P0 list. Every live credential found triggers the critical escalation — no other work proceeds until those credentials are rotated and the secrets are removed from code.

---

## 2. Characterization Testing (QA's First Priority)

In a normal sprint, QA writes tests to verify new behavior. In a legacy sprint, QA writes tests to capture *current* behavior — even when that behavior is wrong.

This is called **characterization testing**. The term comes from Michael Feathers' "Working Effectively with Legacy Code," and the concept is simple: before you change anything, document what the system currently does. These tests are your safety net. If they break, you changed behavior. That's not always wrong — but it should always be a conscious decision.

### What QA Captures

**For API endpoints:**
```
Current request → Current response
Including edge cases, even when responses are "wrong"
Status codes, response shapes, error messages (verbatim)
```

**For functions and methods:**
```
Known inputs → Known outputs
Including surprising outputs from surprising inputs
Side effects (what state changes, what gets written to disk/db)
```

**For background jobs and workers:**
```
Input event → Observable outcome
Including timing behaviors if they matter
```

### How to Write Characterization Tests

The rule: **do not assert what should happen. Assert what does happen.**

```typescript
// DO: Characterize current behavior
it('returns 200 with null thumbnailUrl when content has no thumbnail', async () => {
  const res = await api.get('/api/content/test-id-no-thumbnail');
  expect(res.status).toBe(200);
  expect(res.body.thumbnailUrl).toBeNull();
  // Note: this is wrong behavior — should return a default image URL
  // This test exists to prevent silent regressions while we fix it
});

// DO NOT: Assert desired behavior before the fix exists
it('returns default thumbnail when content has no thumbnail', async () => {
  // This test will fail until BACKEND actually fixes it — that's not characterization
});
```

Include comments explaining the known-wrong behavior. Future agents (and future you) need to know whether a failing test means "we fixed it" or "we broke it."

### QA's Legacy Sprint Activation

QA's activation prompt changes significantly for legacy codebases:

```
You are QA agent for [PROJECT] Legacy Sprint Zero.
Your branch is sprint-00/qa.

PRIORITY: Characterization tests. No other work until these exist.

PHASE 1 — Discover test surface:
1. Audit existing tests: how many exist, what they cover, do they pass?
   Run: [test command] and record the result exactly.
2. List all public API endpoints (read routes files, not docs — docs lie).
3. List all public functions in service layer.
4. List all background jobs and scheduled tasks.

PHASE 2 — Write characterization tests:
For each endpoint, write a test that captures current behavior.
Use black-box style: call the endpoint, record the response.
Assert the actual observed output, not what the output should be.
Mark tests that capture known-wrong behavior with:
  // CHARACTERIZATION: Current behavior is [X]. Desired behavior is [Y].
  // Do not remove this test when fixing — update it to reflect the fix.

PHASE 3 — Identify gaps:
List endpoints and functions that have no characterization tests and
explain why (no test harness exists, needs database fixture, etc.).

TERRITORY: tests/, __tests__/, spec/, **/*_test.*, **/*.test.*, **/*.spec.*
CANNOT modify: Application code (read-only for characterization)

CRITICAL ESCALATION PROTOCOL: [standard protocol — see OPERATORS-GUIDE.md]

WHEN DONE:
Write completion report with:
  - Number of characterization tests written
  - Coverage of known API surface (X of Y endpoints covered)
  - List of gaps and why they exist
  - Existing test suite status (pass/fail/count)
  - P0 DISCOVERIES (security holes, data corruption, broken critical paths)
```

### The Safety Net Rule

No other agent should modify business logic until QA's characterization tests exist and pass. This is the hard rule.

**The order is:**
1. QA writes characterization tests
2. Characterization tests pass on current code
3. DATA does data audit
4. Only then do BACKEND, FRONTEND, SERVICES start making changes

If you skip this and an agent "fixes" something that breaks existing behavior, you will not know it until users complain.

---

## 3. The Legacy Sprint Zero

Sprint Zero is not about shipping features. It is about creating the conditions under which features can be safely shipped. Every legacy codebase needs one before real work begins.

### Sprint Zero Objectives

| Agent | Mission |
|-------|---------|
| **QA** | Characterization tests for all critical paths |
| **INFRA** | Get ANY CI/CD running — even if it only runs `build` and the characterization tests |
| **DATA** | Data audit: schema, migrations state, data integrity violations |
| **LEAD** | Secret scan results, dependency audit, archaeology findings, P0 escalations |

### Sprint Zero Wave Order

Wave order is different here because the goal is understanding, not building.

```
Wave 1 (parallel, read-only):
  QA:    Map test surface, write characterization tests
  LEAD:    Run secret scan, run dependency audit, document findings

Wave 2 (after Wave 1 findings are known):
  DATA: Data audit — depends on knowing what secrets/credentials are real
  INFRA: CI/CD setup — depends on knowing what build commands work

Wave 3:
  LEAD:    Compile Sprint Zero findings into a master P0/P1 issue list
```

### Sprint Zero DISPATCH.md — Abbreviated Example

```markdown
# Sprint Zero Dispatch — Legacy Codebase

## Sprint Theme
Archaeology and safety nets. No feature work. No refactoring.
Goal: understand what we have and be able to make changes safely.

## Success Criteria
- [ ] Characterization tests exist for all critical paths
- [ ] CI/CD pipeline runs (even minimal) on every commit
- [ ] Data schema is documented, integrity issues known
- [ ] All P0 issues (secrets, security holes, data corruption) are listed
- [ ] Dependency vulnerabilities are inventoried

## P0 Protocol
Any agent that discovers a live credential, unauthenticated admin endpoint,
or data corruption pattern must STOP and escalate immediately via critical escalation.

## Agents

### QA — Characterization Tests (Wave 1)
[chains for test discovery and writing]

### LEAD — Security Baseline (Wave 1)
[chains for secret scanning and dependency audit]

### DATA — Data Audit (Wave 2)
[chains for schema documentation and integrity checks]

### INFRA — Minimal CI/CD (Wave 2)
[chains for getting any pipeline running]
```

### What Sprint Zero Produces

At the end of Sprint Zero you have:
- A set of characterization tests that pass on current code
- A CI/CD pipeline that runs those tests on every commit
- A documented list of every P0/P1 issue in the codebase
- A data schema map with known integrity violations
- A dependency audit with known CVEs

That is the foundation for every subsequent sprint. Without it, you are working blind.

---

## 4. Incremental Modernization Patterns

Once Sprint Zero is complete and your safety nets exist, real work begins. Do not attempt to rewrite the codebase. Incremental, reversible transformations are how legacy codebases get modernized without catastrophic failures.

### Strangler Fig

The safest pattern for legacy codebases. New code wraps old code at a seam. Old code shrinks over sprints as the new implementation proves itself.

```
Sprint 1:  New /api/v2/payments routes added alongside existing /payments routes
           Both routes serve the same data — new routes backed by new implementation

Sprint 2:  Traffic shifted to v2 routes. Old routes still exist but deprecated.
           Characterization tests compare v1 and v2 outputs for parity.

Sprint 3:  Old routes removed. Old implementation deleted.
```

**Signal for SERVICES and BACKEND:** Tasks should be written as "add new implementation alongside existing one" not "replace existing implementation." The strangler fig dies when someone tries to do the replacement in a single sprint.

### Branch by Abstraction

Used when you need to replace a dependency (database, cache, external service) that is embedded throughout the codebase.

```
Phase 1 (BACKEND): Identify all call sites for the old dependency.
                 Extract an interface that both old and new implementations satisfy.
                 Wire the old implementation through the interface (no behavior change).

Phase 2 (DATA): Build the new implementation behind the same interface.
                   Run characterization tests against both implementations.

Phase 3 (BACKEND): Swap the implementation behind the interface.
                 Characterization tests validate behavioral parity.

Phase 4: Delete the old implementation.
```

**Execution trace template:**
```
Chain 1 [P2]: Extract database interface
  Vector: grep -r "db\." src/ → catalog all direct DB calls
  → Identify lowest-common-denominator interface
  → Create interface file
  → Wrap existing DB calls through interface (no logic change)
  Signal: Codebase calls database directly in 47 places with no abstraction.
  Fix: Extract DB interface, wire existing implementation through it.
  Verify: All existing characterization tests still pass. Zero behavior change.
```

### Parallel Run

For high-risk migrations where you cannot afford to be wrong. The new implementation runs alongside the old one, their outputs are compared, and discrepancies are logged. Go live only when the outputs match.

```go
// Example: Parallel run in a service layer
func (s *PaymentService) ProcessPayment(ctx context.Context, req PaymentRequest) (*PaymentResult, error) {
    // Old path — currently live
    oldResult, oldErr := s.legacyProcessor.Process(ctx, req)

    // New path — running in shadow
    newResult, newErr := s.newProcessor.Process(ctx, req)

    // Compare — log discrepancies, don't fail on them yet
    if !resultsMatch(oldResult, newResult) {
        s.logger.Warn("parallel run mismatch",
            "old", oldResult, "new", newResult,
            "old_err", oldErr, "new_err", newErr)
        s.metrics.Inc("payment.parallel_run.mismatch")
    }

    // Serve old result until new implementation is validated
    return oldResult, oldErr
}
```

Add this pattern to SERVICES or BACKEND tasks when migrating external integrations or high-risk service logic. Use it for at least one sprint's worth of production traffic before cutting over.

### Database-First

When the application code is a mess but the data model is relatively sane, start with the database. Clean data models constrain the chaos in the application layer.

**Sprint ordering for database-first:**

```
Sprint 1 (DATA):
  - Audit schema: missing indexes, nullable columns that shouldn't be, missing constraints
  - Write migration scripts (reversible — always reversible)
  - Add data integrity checks
  - Document the schema (what each table is, what each column means)

Sprint 2 (DATA + BACKEND):
  - Add model-layer validation that mirrors database constraints
  - Wrap raw DB queries in repository pattern
  - Eliminate N+1 queries identified in audit

Sprint 3 (BACKEND):
  - Clean service layer now that data layer is solid
```

Do not start on the database-first path without DATA's data audit from Sprint Zero. You need to know what you have before you start normalizing it.

---

## 5. Legacy-Specific Agent Adaptations

The agent roles do not change, but their behavior, tempo, and priorities shift significantly on legacy codebases.

### DATA — Data Layer

**Normal sprint:** Fix data bugs, optimize queries, write migrations.

**Legacy sprint:** DATA runs first, before anyone else touches anything.

DATA's audit must answer:
- What does the schema actually look like (not what migrations say — what `\d+ tablename` shows right now)?
- Are there orphaned records? Foreign keys without constraints? Nullable columns storing empty strings instead of NULL?
- Are migrations reversible? Is migration history complete?
- Are there direct-to-database calls bypassing the ORM/model layer?
- Is there any raw SQL with string concatenation (injection risk)?

```bash
# DATA audit commands — PostgreSQL
psql $DATABASE_URL -c "\dt"                        # List all tables
psql $DATABASE_URL -c "\d+ tablename"              # Full column info
psql $DATABASE_URL -c "SELECT relname, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC;"  # Row counts
psql $DATABASE_URL -c "SELECT * FROM pg_indexes WHERE tablename NOT LIKE 'pg_%';"  # All indexes
psql $DATABASE_URL -c "SELECT conname, contype, conrelid::regclass FROM pg_constraint;"  # All constraints

# Find orphaned foreign key records (example)
psql $DATABASE_URL -c "
  SELECT count(*) FROM orders o
  LEFT JOIN users u ON o.user_id = u.id
  WHERE u.id IS NULL;
"

# MySQL equivalent
mysql -e "SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = 'your_db';"
```

**DATA's output from Sprint Zero is a prerequisite for every other agent.** Until DATA confirms the data layer is understood (not fixed — understood), no other agent can safely plan changes that touch data.

### QA — QA / Security

**Normal sprint:** Write tests for new behavior after code is written.

**Legacy sprint:** Characterization tests before code is touched. This is QA's entire Sprint Zero mission.

After Sprint Zero, QA's subsequent sprint role expands:
- Every bug fix by BACKEND or DATA must have a characterization test updated to reflect the corrected behavior (not removed — updated)
- QA maintains a "known behavior changelog" that tracks which characterization tests were intentionally changed and why
- Security audit is continuous, not one-time: every sprint QA runs a fresh dependency scan

**The characterization test lifecycle:**
```
Sprint 0: Test captures WRONG behavior, marked with // CHARACTERIZATION comment
Sprint N: BACKEND fixes the behavior
Sprint N: QA updates the test to assert CORRECT behavior, removes the comment
```

### BACKEND — Backend Logic

**Normal sprint:** Fix handler bugs, implement service logic, clean routing.

**Legacy sprint:** BACKEND does not refactor until safety nets exist. Full stop.

The temptation on legacy codebases is to "clean up while you're in there." A 400-line handler with three bugs is crying out to be split into smaller functions. Resist this completely in the first two sprints. Fix the bugs. Do not restructure. Restructuring changes the behavior surface and breaks characterization tests.

After Sprint Zero and Sprint One (bug fixes with characterization), Sprint Two is when BACKEND can begin structural improvements — but only on files that have characterization test coverage.

**BACKEND's execution traces constraint for legacy sprints:**
```
You are tracing a bug, not refactoring a file.
Fix the specific failure point identified in the vector.
Make the minimum change that resolves the bug.
If the fix requires restructuring, document it as a follow-up chain.
Do not restructure in the same chain as a bug fix.
```

### FRONTEND — Frontend UI

**Normal sprint:** Fix UI bugs, optimize network requests, clean components.

**Legacy sprint:** FRONTEND may not have a framework to work with. If the frontend is jQuery-era code with inline scripts, FRONTEND does not rewrite to React. FRONTEND works within the existing paradigm.

Common legacy frontend situations:
- **Inline `<script>` tags with global variables**: Work with the globals, don't try to eliminate them in Sprint Zero
- **jQuery everywhere**: Fix the jQuery bugs with jQuery, not with a "let me just add React to this one component"
- **Server-rendered templates**: The component model doesn't apply — think in template partials
- **Vanilla JS, no bundler**: FRONTEND cannot use import statements or modern module syntax until INFRA sets up a build pipeline

FRONTEND's Sprint Zero task is documentation, not modification: catalog what frontend code exists, what it does, and what's broken. The fixes come in subsequent sprints once INFRA has established a build pipeline.

### INFRA — Infrastructure

**Normal sprint:** Optimize Docker, improve CI/CD, manage environment config.

**Legacy sprint:** INFRA's first priority is getting ANY CI/CD running. "Any" means exactly that. A GitHub Actions workflow that runs `npm install && npm run build` and exits 0 is infinitely better than no CI/CD.

The minimal viable pipeline for Sprint Zero:
```yaml
# .github/workflows/ci.yml — absolute minimum
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: [your build command]
      - name: Test
        run: [your test command]  # Will run characterization tests from QA
```

From this baseline, INFRA iterates. Linting in Sprint One. Security scanning in Sprint Two. Deployment automation in Sprint Three. Build a pipeline no one has time to argue about in Sprint Zero, then add to it.

INFRA also takes ownership of the dependency audit findings from LEAD. CVEs in direct dependencies get patched. CVEs in transitive dependencies get documented and tracked. Dependencies that haven't been updated in 4 years get flagged for replacement in the modernization backlog.

### SERVICES — Specialized Services

**Normal sprint:** Clean integrations, deduplicate clients, improve error handling.

**Legacy sprint:** SERVICES often can't start until DATA and BACKEND have run. Legacy integration code is usually mixed with business logic and data access — there's no clean boundary to work within.

SERVICES's Sprint Zero task: map the integrations. What external services does this codebase call? How are credentials managed (hint: they're probably hardcoded)? Is there retry logic? Timeout handling? Circuit breaking? Log the findings. Don't fix them yet.

For integrations embedded in spaghetti (a Stripe API call sitting in the middle of a handler function alongside DOM manipulation and database queries), SERVICES uses extraction before optimization:

```
Phase 1 (Sprint 1, SERVICES): Extract the integration call to its own function.
                           Do not change the logic. Just move it.
                           Characterization tests must still pass.

Phase 2 (Sprint 2, SERVICES): Now the integration logic is isolated,
                           optimize it: add retry, timeout, circuit breaker.
                           Write integration tests against the isolated code.

Phase 3 (Sprint 3, SERVICES): Move the isolated function to its proper home
                           in an integrations/ or clients/ directory.
```

### LEAD — Orchestrator

**Normal sprint:** Merge management, documentation, ship decisions.

**Legacy sprint:** LEAD takes on an additional role: **triage coordinator**.

Legacy sprints produce more P0 discoveries than greenfield sprints. LEAD maintains the P0/P1 issue list and helps the operator decide what to handle now versus what to park. When three agents simultaneously discover security holes, LEAD is the single point of triage — not because LEAD has special knowledge, but because one coordinate point for critical escalations is better than three agents each trying to reach the operator.

**LEAD's legacy Sprint Zero deliverable:**

```markdown
# Sprint Zero: Master Issue List

## P0 — Fix Before Any Other Work

### P0-001: Live AWS credentials in config/database.yml (LEAD discovery)
File: config/database.yml, lines 14-16
Credentials appear active. Rotate immediately.
Agent to fix: INFRA (move to environment variables)

### P0-002: Admin endpoint /admin/users has no authentication (QA discovery)
File: routes/admin.js, line 89
Any unauthenticated user can list, modify, and delete all users.
Agent to fix: BACKEND (add auth middleware)

### P0-003: SQL injection in user search (QA discovery)
File: models/UserModel.php, line 203
Raw string concatenation: `"WHERE username LIKE '%" . $_GET['q'] . "%'"`
Agent to fix: DATA (parameterize query)

## P1 — Fix This Sprint

[list continues]

## P2 — Fix in Next Sprint

[list continues]

## Known-Bad Behavior (Characterization Tests Exist)

[list of wrong behaviors that are now documented and tested]
```

This document becomes the single source of truth for all future sprint planning on the codebase.

---

## 6. Red Flags in Legacy Code

These patterns warrant P0 or P1 escalation when agents encounter them. Include this list in every agent's activation prompt for legacy sprints so they know what to surface.

### Execution Risk (P0)

```
eval(), exec(), system(), shell_exec(), passthru()
Any function that executes arbitrary strings as code or system commands.

Raw string concatenation into SQL queries:
  "SELECT * FROM users WHERE id = " + userId
  f"SELECT * FROM users WHERE id = {user_id}"
  "SELECT * FROM users WHERE id = #{user_id}"

File path construction from user input without validation:
  File.read("/uploads/" + filename)
  open(base_dir + request.path)
```

### Secret Exposure (P0)

```
Hardcoded strings that match patterns:
  AKIA[0-9A-Z]{16}              # AWS access key
  sk_live_[a-zA-Z0-9]{24,}     # Stripe live key
  ghp_[a-zA-Z0-9]{36}          # GitHub personal access token
  AIza[0-9A-Za-z\-_]{35}       # Google API key
  postgres://user:password@     # DB connection string with credentials

Credentials in comments:
  // password: admin123 (old password, may still work)
  # API_KEY = "abc123" -- remove before commit (was not removed)
```

### Authentication and Authorization Failures (P0)

```
Routes that should require authentication but don't:
  app.get('/admin/users', getAllUsers)   // no auth middleware
  router.delete('/data', deleteAll)     // no auth, no confirmation

Direct object references without ownership checks:
  const record = await db.find(req.params.id)
  // Missing: if (record.userId !== req.user.id) throw Forbidden
  return record

Role checks based on user-supplied input:
  if (req.body.role === 'admin') { /* grant access */ }
```

### Data Integrity Risk (P1)

```
Global mutable state:
  var currentUser = null  // global, modified by multiple handlers
  $GLOBALS['db']          // PHP global
  global db               // Python global

Missing transactions on multi-step writes:
  await db.update('orders', { status: 'paid' }, id)
  await db.update('inventory', { count: count - 1 }, itemId)
  // If second update fails, order is paid but inventory is not decremented

Race conditions on read-modify-write:
  const count = await db.get('user_count')
  await db.set('user_count', count + 1)  // non-atomic, breaks under concurrency
```

### Structural Warning Signs (P2)

```
Files over 2000 lines:
  Models that contain routing logic.
  Controllers that contain SQL.
  Utilities that import from the application layer.

goto statements (PHP, C, older Go):
  goto retry;  // almost always signals confused control flow

Massive switch/case blocks (>20 cases) doing business logic:
  Indicates missing polymorphism or a state machine that wasn't designed.

Commented-out code spanning more than 20 lines:
  Either dead code or a failed attempt to fix something.
  Both need to be understood before the surrounding code is touched.

Version checks for long-dead versions:
  if (PHP_VERSION < '5.3') { ... }
  if (node.version < 'v0.12') { ... }
  These conditions are unreachable and indicate unmaintained code.
```

### The Red Flag Protocol in Activation Prompts

Add this block to every agent's activation prompt on legacy sprints:

```
RED FLAG PROTOCOL:
If you encounter any of the following during your work, stop the current chain
and document it as a P0 or P1 discovery before continuing:

P0 (stop everything, escalate immediately):
  - eval/exec/system with any user-controlled input
  - SQL queries built with string concatenation
  - Hardcoded live credentials (API keys, passwords, tokens)
  - Admin or destructive endpoints with no authentication

P1 (document before continuing, finish current chain):
  - Global mutable state in request handlers
  - Multi-step writes with no transaction
  - Direct object references without ownership checks
  - Files over 2000 lines in your territory

Document findings under "RED FLAG DISCOVERIES" in your completion report.
Include: file path, line number, what you found, why it's dangerous.
```

---

## 7. Communication Pattern for Legacy Sprints

Legacy sprints produce more P0 discoveries than greenfield sprints. Plan for it.

### The Triage Rhythm

In a greenfield sprint, you dispatch Wave 1, come back in a few hours, and everything is fine. In a legacy sprint, Wave 1 agents are likely to surface multiple P0 issues within the first 30 minutes. You cannot go make coffee and come back — you need to be available to triage.

**Recommended monitoring cadence for Sprint Zero:**
- Check QA and LEAD completion reports every 15-30 minutes during Wave 1
- Have a working P0 list open (a text file, a Notion doc, whatever you use)
- Each P0 finding gets: a file path, a line number, an assigned agent, and a priority order for fixing

**The operator's role during legacy Sprint Zero:**
```
You are not a passive observer. You are an active triage coordinator.
Agents will find things. You decide: fix now, fix this sprint, document and carry forward.
Not every P0 is equally urgent. A hardcoded API key with no network access
is less urgent than an unauthenticated endpoint on a live production system.
Use judgment. Make explicit decisions. Record them.
```

### The P0 Decision Tree

When an agent surfaces a P0:

```
Is the system currently in production?
  YES → Is the vulnerability actively exploitable right now?
    YES → Rotate credentials / take endpoint offline BEFORE any other agent work.
          This takes priority over the entire sprint.
    NO  → Add to P0 list, assign to appropriate agent, fix in this sprint.
          Do not proceed to Sprint 1 until all production P0s are resolved.
  NO  → Document it, assign it to Sprint 1 P1 list.
        Continue Sprint Zero for other findings.
```

### Adjusting the Critical Escalation for Legacy

The standard critical escalation protocol says: stop, document, wait for operator decision. In a legacy codebase during Sprint Zero, this may fire 10-15 times per agent. You cannot interrupt 10-15 times per agent per sprint.

Adjust the protocol for Sprint Zero:

```
LEGACY SPRINT ZERO CRITICAL ESCALATION PROTOCOL:
For P0 findings during Sprint Zero:
  - DO NOT stop your current work.
  - Document the finding immediately in your completion report under "P0 DISCOVERIES".
  - Continue your current chain.
  - After completing your CURRENT chain (not all chains — just the current one), check:
    if any P0 discovery is in your territory → fix it before starting the next chain.
    if P0 discovery is outside your territory → leave it for the assigned agent.

This modified protocol applies ONLY to Sprint Zero, where discovery is the primary goal.
In Sprint 1 and beyond, the standard critical escalation protocol applies: stop, document, wait.
```

### What LEAD's Sprint Zero Summary Looks Like

LEAD's final report for Sprint Zero is not a standard completion report. It is a master handoff document that the operator uses to plan all subsequent sprints.

```markdown
# Sprint Zero Summary — [Project Name]

## Executive Assessment
[2-3 sentences: how bad is this, roughly]

## P0 Issues Requiring Immediate Action (before Sprint 1)
| ID | Issue | File | Line | Agent to Fix | Status |
|----|-------|------|------|--------------|--------|
| P0-001 | Live Stripe key hardcoded | config/stripe.js | 3 | INFRA | OPEN |
| P0-002 | SQL injection in search | lib/search.php | 47 | DATA | OPEN |

## P1 Issues for Sprint 1
[table of P1 issues]

## Safety Net Status
- Characterization tests: [X] tests written, covering [Y]% of known API surface
- CI/CD: [running / not running] — [what it runs]
- Known-bad behaviors documented: [X] behaviors

## Data Layer Summary (DATA)
- Schema documented: [yes / partial / no]
- Integrity violations found: [list or count]
- Migration state: [clean / diverged / unknown]

## Security Baseline (QA)
- Dependency CVEs: [critical: X, high: Y, medium: Z]
- Authentication gaps: [count and description]
- Injection risks: [count and description]

## Hotspot Files (Top 10 by churn)
[list with change frequency and why it matters]

## Recommended Sprint 1 Theme
[what the data suggests should come first]

## Modernization Backlog
[patterns identified for incremental modernization — strangler fig candidates, etc.]
```

This document is the starting point for every sprint planning session until the codebase is stable.

---

## Quick Reference

### Sprint Zero Checklist

```
Before dispatching any agent:
  [ ] Secret scan completed (trufflehog or gitleaks)
  [ ] Any live credentials found → ROTATED (not logged, rotated)
  [ ] Hotspot analysis run (git log churn)
  [ ] Dependency mapping done (madge / go list / phpstan)

Sprint Zero waves:
  [ ] Wave 1: QA (characterization tests) + LEAD (secret scan + dep audit) dispatched
  [ ] Wave 1 P0s triaged and assigned
  [ ] Wave 2: DATA (data audit) + INFRA (minimal CI/CD) dispatched
  [ ] All Wave 2 completion reports collected

Before Sprint 1:
  [ ] All production P0s resolved
  [ ] Characterization tests passing in CI
  [ ] Sprint Zero master issue list complete
  [ ] Sprint 1 theme chosen based on findings
```

### Legacy Sprint Wave Order

```
Sprint Zero (discovery):
  Wave 1: QA (characterization) + LEAD (security baseline) — parallel
  Wave 2: DATA (data audit) + INFRA (minimal CI/CD) — parallel
  Wave 3: LEAD (master issue list compilation)

Sprint 1 and beyond (standard wave order, but QA still goes first):
  Wave 1: QA (update characterization tests for planned changes) + DATA + INFRA
  Wave 2: BACKEND + SERVICES
  Wave 3: FRONTEND
  Wave 4: LEAD
```

### Modernization Pattern Selection Guide

```
What's the primary problem?

  "Everything depends on everything" (circular deps, spaghetti)
    → Branch by Abstraction: extract interface, swap implementation

  "Need to replace a core dependency" (old DB driver, legacy auth system)
    → Branch by Abstraction + Parallel Run

  "Old routes/features coexist with new ones"
    → Strangler Fig: new routes alongside old, migrate traffic

  "The database schema is the mess, not the code"
    → Database-First: normalize schema before touching application

  "High-risk migration, can't be wrong"
    → Parallel Run: run old and new simultaneously, compare outputs
```

### Execution Pace for Legacy Sprints

| Agent | Normal Tempo | Legacy Sprint Zero Tempo | Why |
|-------|-------------|--------------------------|-----|
| DATA | Slow, careful | Slower — audit mode | Data corruption risk is highest here |
| QA | Broad, scanning | Broad, non-modifying | Write tests, read code, don't change anything |
| INFRA | Fast, mechanical | Slow — understand first | The existing build process may be fragile |
| BACKEND | Moderate | Read-only in Sprint Zero | Do not touch business logic without safety nets |
| SERVICES | Moderate | Read-only in Sprint Zero | Integration side effects are unpredictable |
| FRONTEND | Fast, iterative | Read-only in Sprint Zero | Frontend may depend on fragile build process |
| LEAD | Deliberate | Deliberate — triage mode | Sprint Zero P0 list requires careful judgment |

---

**Related Documents:**
- [OPERATORS-GUIDE.md](operators-guide.md) — Full tutorial for running sprints
- [METHODOLOGY.md](../core/methodology.md) — Execution traces, chain execution, priority levels
- [agents/](../agents/) — Agent role definitions
- [WORKFLOW.md](../core/workflow.md) — Technical workflow details
- [CUSTOMIZATION.md](customization.md) — Adapt agent territories for your project
