<!--
  TEMPLATE: Agent Completion Report
  Every agent writes one of these when their work is done.
  Copy, rename to agent-[X]-completion.md, fill in all sections.
  Remove this comment block before committing.

  This report is read by:
  - LEAD (to decide merge readiness)
  - RED TEAM (to know what changed and where to look)
  - The operator (to verify quality and track progress)
  - Other agents (to understand what changed in your territory)

  Be precise. "Fixed the bug" is not useful. "Wrapped map write in mutex at store/workspace.go:88
  to eliminate DATA RACE under concurrent access" is useful.
-->

# Agent [X] — [CODENAME]: Sprint [XX] Completion Report

> Branch: `sprint-[XX]/[agent-name]`
> Date: [YYYY-MM-DD]
> Status: **COMPLETE** | **PARTIAL** | **BLOCKED**

---

## Summary

[2-3 sentences: What was accomplished. What wasn't and why. Any surprises or discoveries.]

---

## Tasks Completed

| Task ID | Title | Priority | Status | Files Changed |
|---------|-------|----------|--------|---------------|
| [X]-01 | [title] | P1 | COMPLETE | [count] |
| [X]-02 | [title] | P1 | COMPLETE | [count] |
| [X]-03 | [title] | P2 | COMPLETE | [count] |
| [X]-04 | [title] | P2 | PARKED — [reason] | 0 |

---

## Task Details

### [X]-01: [Title] — COMPLETE

**What was wrong:** [Root cause — one sentence]

**What changed:**
- `path/to/file.ts` — [specific change, e.g., "Implemented JWT validation in canActivate(), replacing stub that returned true"]
- `path/to/other.ts` — [specific change, e.g., "Updated to read workspaceId from guard instead of request body"]

**Verification:**
- Build passes: `[command]` — PASS
- Tests pass: `[command]` — PASS
- [Task-specific check]: [result]

---

### [X]-02: [Title] — COMPLETE

**What was wrong:** [Root cause]

**What changed:**
- `path/to/file.ts` — [change]

**Verification:**
- Build: PASS
- Tests: PASS
- [Specific check]: [result]

---

### [X]-03: [Title] — PARKED

**Reason:** [Why this task was not completed — dependency, blocker, time, discovered larger scope]
**Recommended next step:** [What the next agent or sprint should do]

---

## P0 Discoveries

Issues discovered during work that were NOT in the original task list. These are critical
findings that the operator and LEAD must review before merging.

| ID | Description | File | Line | Risk | Recommended Agent |
|----|-------------|------|------|------|-------------------|
| P0-[X]-01 | [what you found] | [path] | [line] | [why it's critical] | [who should fix] |

> If no P0 discoveries, write: "None discovered."

---

## Issues for Other Agents

Non-critical findings that other agents should be aware of.

| Issue | Priority | Recommended Agent | Notes |
|-------|----------|-------------------|-------|
| [description] | P1/P2/P3 | [AGENT_NAME] | [context — e.g., "Found during [X]-02, outside my territory"] |

> If none, write: "None."

---

## Files Modified

Complete list of every file changed, created, or deleted:

```
MODIFIED  [path/to/file1.ts]
MODIFIED  [path/to/file2.ts]
CREATED   [path/to/new-file.ts]
DELETED   [path/to/dead-file.ts]
```

---

## Verification Results

```bash
# Build
$ [build command]
# Result: PASS / FAIL

# Tests
$ [test command]
# Result: PASS — [N] tests passed, [N] failed, [N] skipped

# Lint
$ [lint command]
# Result: PASS / FAIL

# Territory check
$ git diff --name-only main..HEAD
# All modified files are within declared territory: YES / NO
```

---

## Metrics

- Tasks assigned: [N]
- Tasks completed: [N]
- Tasks parked: [N]
- Files modified: [N]
- Files created: [N]
- Files deleted: [N]
- Lines changed: +[added] / -[removed]
- Tests added: [N]
- Build status: PASS / FAIL
- Test status: PASS / FAIL ([N] total, [N] passed)
