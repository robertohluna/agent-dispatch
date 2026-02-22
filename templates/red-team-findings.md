# RED TEAM Findings Report — Sprint [XX]

> Adversarial review of all agent branches
> Reviewed by: RED TEAM (Agent R)
> Date: [YYYY-MM-DD]

---

## Summary

- **Branches reviewed:** [N] of [N]
- **Total findings:** [N]
- **CRITICAL:** [N] (blocks merge)
- **HIGH:** [N] (blocks merge)
- **MEDIUM:** [N]
- **LOW:** [N]
- **Merge recommendation:** [CLEAR TO MERGE / CONDITIONAL / BLOCKED]

---

## Findings by Agent Branch

### sprint-[XX]/[agent-name]

#### RT-[XX]-001: [Finding Title]

| Field | Value |
|-------|-------|
| **Severity** | [CRITICAL / HIGH / MEDIUM / LOW] |
| **Category** | [Security / Edge Case / Regression / Territory Violation / Data Integrity] |
| **File** | `[path/to/file.go:line]` |
| **Description** | [What's wrong and why it matters] |
| **Reproduction** | [Steps to trigger the issue] |
| **Impact** | [What happens if this ships — data loss, auth bypass, etc.] |
| **Adversarial Test** | `[path/to/test_file.go:TestName]` (if written) |
| **Recommendation** | [How to fix] |

<!-- Repeat RT-[XX]-NNN blocks for each finding on this branch -->

<!-- Copy this entire "### sprint-[XX]/[agent-name]" section for each branch reviewed -->

---

## Regression Test Results

| Branch | Tests Run | Passed | Failed | New Failures |
|--------|-----------|--------|--------|--------------|
| sprint-[XX]/data | [N] | [N] | [N] | [N] |
| sprint-[XX]/design | [N] | [N] | [N] | [N] |
| sprint-[XX]/backend | [N] | [N] | [N] | [N] |
| sprint-[XX]/services | [N] | [N] | [N] | [N] |
| sprint-[XX]/frontend | [N] | [N] | [N] | [N] |
| sprint-[XX]/infra | [N] | [N] | [N] | [N] |
| sprint-[XX]/qa | [N] | [N] | [N] | [N] |

---

## Territory Violations

| Agent | File Modified | Expected Territory | Violation? |
|-------|--------------|-------------------|------------|
| [agent-name] | `[path/to/file]` | [which agent owns this file/dir] | [YES / NO] |

<!-- Add one row per suspicious cross-territory modification. Remove this comment when filling in. -->

---

## Merge Recommendations

| Branch | Recommendation | Blocking Findings | Notes |
|--------|---------------|-------------------|-------|
| sprint-[XX]/data | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/design | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/backend | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/services | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/frontend | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/infra | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |
| sprint-[XX]/qa | [CLEAR / CONDITIONAL / BLOCKED] | [RT-XX-NNN, ...] | [any context] |

---

## Severity Reference

| Severity | Meaning | Merge Impact |
|----------|---------|-------------|
| CRITICAL | Security vulnerability, data corruption, auth bypass | **BLOCKS merge.** Must fix before shipping. |
| HIGH | Race condition, missing validation, unhandled error on critical path | **BLOCKS merge.** Fix or accept with documented risk. |
| MEDIUM | Edge case not handled, missing test coverage, suboptimal error message | Does not block. Fix this sprint or carry to next. |
| LOW | Code style, minor improvement opportunity, documentation gap | Does not block. Note for future. |
