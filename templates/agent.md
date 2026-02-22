<!--
  TEMPLATE: Per-Agent Task Document
  The dispatcher creates one of these for each agent in a sprint.
  Copy this file, rename to agent-[X]-[domain].md, fill in every [placeholder].
  Remove this comment block before committing.

  This is the document agents read to understand their work. It must be specific enough
  that an AI agent can execute every task without asking questions. Vague instructions
  produce vague results — be precise about current state, required state, and verification.
-->

# Agent [X] — [CODENAME]: [Domain]

## Sprint [XX] | Branch: `sprint-[XX]/[agent-name]`
## Working Directory: `[PARENT_DIR]/[PROJECT_NAME]-[agent-name]`

---

## Role

[2-3 sentences: What this agent does this sprint. What is broken, missing, or being built.
What "done" looks like. Include architectural context — how this agent's work fits into
the larger sprint and what other agents depend on or feed into.]

---

## Context — Read Before Coding

Read these files in this order before writing any code:

1. `[PROJECT_CONTEXT_FILE]` — project patterns, architecture, build commands
2. `[domain-specific doc 1]` — [why this is relevant, e.g., "current schema and migration patterns"]
3. `[domain-specific doc 2]` — [why, e.g., "edge function auth patterns you'll be porting"]
4. `[source file 1]` — [why, e.g., "the auth guard you'll be reimplementing"]
5. `[source file 2]` — [why, e.g., "the algorithm you'll be porting — read carefully"]
6. `[source file N]` — [why]
7. This document — YOUR TASK DOC (follow exactly)

[OPTIONAL: Include grep commands to discover context]
```bash
# Find all callers of the function you're modifying
grep -r "functionName" src/ --include="*.ts"
```

---

## Files Owned

Exact files this agent can modify. No other agent touches these during this sprint.

```
[path/to/file1.ts]
[path/to/file2.ts]
[path/to/directory/]
[path/to/file3.ts] (CREATE — new file)
[path/to/file4.ts] (DELETE)
[path/to/file5.ts] (import path fix only)
```

---

## Tasks

### [X]-01: [Task Title] ([CRITICAL/P1/P2/P3])

**File(s):** `[path/to/file.ts]`

**Current state:**
[Describe what exists now. Include code snippet if relevant:]
```typescript
// What the code looks like RIGHT NOW
@Injectable()
export class WorkspaceAuthGuard implements CanActivate {
  canActivate(_context: ExecutionContext): boolean {
    return true; // STUB: always allow
  }
}
```

**Required implementation:**
[Numbered steps for exactly what needs to change:]
1. [Step 1 — e.g., Extract Bearer token from Authorization header]
2. [Step 2 — e.g., Verify via supabase.auth.getUser(token)]
3. [Step 3 — e.g., Query workspace_members to verify membership]
4. [Step 4 — e.g., Attach { user, workspaceId } to request]

**Reference pattern:** [Point to existing code that shows the pattern to follow]
[e.g., "See ChatController:104-113 for JWT extraction + user verification"]

**Key details:**
- [Implementation detail — e.g., "Use SupabaseService from modules/supabase/, not shared/"]
- [Error handling — e.g., "Return 401 for invalid token, 403 for non-member"]
- [Logging — e.g., "Log auth failures at warn level, no PII in logs"]
- [Dependency — e.g., "After implementing guard, update rss.controller.ts to use guard-provided workspaceId"]

---

### [X]-02: [Task Title] ([P1/P2/P3])

**File(s):** `[path/to/file.ts]`

**Current state:**
[What exists now — describe or show code]

**Required changes:**
[What needs to change — numbered steps or before/after code]

**Key details:**
- [Implementation notes]
- [Edge cases to handle]

---

### [X]-03: [Task Title] ([P1/P2/P3])

**File(s):** `[path/to/file.ts]` + `[path/to/other.ts]`

**Current state:**
[Description or code]

**Required changes:**
[Steps]

**Key details:**
- [Notes]

---

[Continue with [X]-04, [X]-05, etc. as needed]

---

## Wave Organization

Which tasks to do in what order. Tasks within a wave can run in parallel.

### Wave 1 (Start immediately)
- [X]-01: [title]
- [X]-02: [title]
- [X]-03: [title]

### Wave 2 (After Wave 1 complete)
- [X]-04: [title]
- [X]-05: [title]

### Wave 3 (After Wave 2 complete)
- [X]-06: [title]

---

## Territory

**Can modify:** Files listed in "Files Owned" section above.

**Do NOT touch:**
- `[directory/]` — [AGENT_NAME] territory
- `[directory/]` — [AGENT_NAME] territory
- `[directory/]` — [AGENT_NAME] territory
- `[specific file]` — [AGENT_NAME] territory (you USE it, don't modify it)

**Read anything** you need for context. Reading does not violate territory.

---

## Verification Checklist

Run these after completing all tasks. Every check must pass before writing your completion report.

```bash
cd [WORKING_DIRECTORY]

# Build
[exact build command]

# Tests
[exact test command]

# Lint (if applicable)
[exact lint command]

# Task-specific verifications:

# Verify [X]-01: [what to check]
[grep/command that proves the task is done]
# Expected output: [what you should see]

# Verify [X]-02: [what to check]
[grep/command]
# Expected output: [expected]

# Verify [X]-03: [what to check]
[grep/command]
# Expected output: [expected]

# Verify no files touched outside territory
git diff --name-only main..HEAD
# Should only contain files listed in "Files Owned"
```

---

## Commit Strategy

Small, focused commits. One per task or logical group:

1. `[type]: [X]-01 — [description]`
2. `[type]: [X]-02 — [description]`
3. `[type]: [X]-03 — [description]`

Commit types: `fix:`, `feat:`, `refactor:`, `chore:`, `security:`, `docs:`, `test:`

---

## Completion

When all tasks are done and verification passes:

1. Write your completion report: `docs/agent-dispatch/sprint-[XX]/agent-[X]-completion.md`
2. Include: tasks completed, tasks blocked, files modified, tests added, P0 discoveries, blockers
3. Commit all work to your branch: `sprint-[XX]/[agent-name]`
