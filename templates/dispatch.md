# Sprint [XX] Dispatch — [Theme]

> [One-line description of sprint goals]
> Created: YYYY-MM-DD

## Sprint Goals

1. Goal 1
2. Goal 2
3. Goal 3

## Execution Traces

### Chain 1: [Title] (P1)
```
[entry point] → [function] → [function] → [root cause]
Signal: [what's broken and observable evidence]
```

### Chain 2: [Title] (P1)
```
[entry point] → [function] → [root cause]
Signal: [what's broken]
```

### Chain 3: [Title] (P2)
```
[trace path]
Signal: [observable evidence]
```

## Wave Assignments

> Each agent's focus should reference the chain(s) above it is resolving.

### Wave 1 — Foundation (no dependencies)

| Agent | Focus | Est. Complexity |
|-------|-------|-----------------|
| DATA | [Data layer tasks] | [1-10] |
| QA | [Test/security tasks] | [1-10] |
| INFRA | [Infrastructure tasks] | [1-10] |
| DESIGN | [Design system/token/a11y tasks] | [1-10] |

### Wave 2 — Backend (depends on Wave 1)

| Agent | Focus | Est. Complexity |
|-------|-------|-----------------|
| BACKEND | [Handler/service tasks] | [1-10] |
| SERVICES | [Integration/service tasks] | [1-10] |

### Wave 3 — Frontend (needs DESIGN specs + stable backend)

| Agent | Focus | Est. Complexity |
|-------|-------|-----------------|
| FRONTEND | [Frontend tasks] | [1-10] |

### Wave 4 — Ship (depends on all)

| Agent | Focus | Est. Complexity |
|-------|-------|-----------------|
| LEAD | [Merge + docs] | [1-10] |

## Merge Order

> Run merge validation (build + test) after EVERY merge before proceeding.

```
1. DATA → main
2. DESIGN   → main
3. BACKEND   → main
4. SERVICES   → main
5. FRONTEND   → main
6. INFRA → main
7. QA    → main
8. LEAD    → main
```

## Success Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Worktree Setup

```bash
SPRINT="sprint-XX"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

for agent in backend frontend infra services qa data lead design; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done

# Install dependencies (customize for your stack)
# for agent in backend frontend infra services qa data; do
#   (cd "$PARENT_DIR/${PROJECT_NAME}-${agent}" && npm install)
# done
```

## Post-Sprint Cleanup

```bash
for agent in backend frontend infra services qa data lead design; do
  git worktree remove "$PARENT_DIR/${PROJECT_NAME}-${agent}" 2>/dev/null
  git branch -d $SPRINT/$agent 2>/dev/null
done
```

---

**Sprint Planning Source:** [Your progress tracker]
