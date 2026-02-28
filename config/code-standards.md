# Agent Code Standards

> How agents write code. Non-negotiable discipline for every agent in every sprint.

---

## The Rule

Write the **shortest correct code** that solves the problem while **maintaining the codebase's existing architecture and patterns**.

That's it. Everything below is elaboration.

---

## Write Minimal, Not Minimal-ist

### DO: Shorter code that achieves the same result

```go
// GOOD — direct, clear, minimal
func (s *Store) GetUser(ctx context.Context, id string) (*User, error) {
    var u User
    err := s.db.QueryRowContext(ctx, "SELECT id, name, email FROM users WHERE id = $1", id).
        Scan(&u.ID, &u.Name, &u.Email)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    return &u, nil
}
```

```go
// BAD — same result, twice the code, zero additional value
func (s *Store) GetUser(ctx context.Context, id string) (*User, error) {
    if ctx == nil {
        return nil, fmt.Errorf("context cannot be nil")
    }
    if id == "" {
        return nil, fmt.Errorf("id cannot be empty")
    }

    query := "SELECT id, name, email FROM users WHERE id = $1"
    s.logger.Debug("executing query", "query", query, "id", id)

    row := s.db.QueryRowContext(ctx, query, id)

    var u User
    if err := row.Scan(&u.ID, &u.Name, &u.Email); err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            s.logger.Info("user not found", "id", id)
            return nil, fmt.Errorf("user not found: %s", id)
        }
        s.logger.Error("failed to scan user", "error", err, "id", id)
        return nil, fmt.Errorf("failed to get user %s: %w", id, err)
    }

    s.logger.Debug("user retrieved successfully", "id", id)
    return &u, nil
}
```

The first version handles the actual requirements. The second adds logging, nil-context guards, empty-string checks, and error-type branching that the caller didn't ask for and the codebase doesn't use elsewhere.

### DON'T: Cut corners on architecture for brevity

```go
// BAD — shorter, but skips the service layer the codebase uses everywhere
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    var u User
    h.db.QueryRow("SELECT ...", id).Scan(&u.ID, &u.Name)  // bypasses service + store layers
    json.NewEncoder(w).Encode(u)
}
```

```go
// GOOD — follows the handler → service → store pattern the codebase already uses
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    u, err := h.userService.GetByID(r.Context(), id)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(u)
}
```

Shorter is not better when it breaks architectural patterns the rest of the codebase depends on.

---

## The Standards

### 1. Match the Codebase Exactly

Before writing anything, read 3-5 files in the same directory. Then match:

- **Naming** — camelCase, snake_case, PascalCase — whatever they use, you use
- **Error handling** — their wrapping pattern, their error types, their return style
- **Import style** — their grouping, their alias conventions, their path resolution
- **Function signatures** — their parameter ordering, their return types, their nil handling
- **File organization** — their section ordering, their comment style, their export patterns

**If you can't tell your code from theirs, you wrote it right.**

### 2. No Over-Engineering

| Over-Engineering | What to Do Instead |
|------------------|-------------------|
| Abstract base class for one implementation | Write the concrete implementation |
| Factory pattern for one object type | Construct the object directly |
| Event bus for two components talking | Direct function call |
| Configuration file for one value | Hardcode it (extract later if needed) |
| Generic `<T>` for one type | Use the concrete type |
| Interface with one implementation | Skip the interface, use the struct |
| Utility function used once | Inline the logic |
| Error types for internal functions | Return `fmt.Errorf()` with context |
| Middleware for one endpoint | Put the logic in the handler |
| Retry/backoff for internal calls | Let it fail, caller handles it |

**The test:** If you're adding abstraction, ask: "Does a second consumer of this exist today?" If no, don't abstract.

### 3. No Under-Engineering

Don't cut these for brevity:

- **Error handling** — Every error path must be handled. No swallowed errors.
- **Input validation at boundaries** — User input, API input, external service responses: validate.
- **The architectural pattern** — If the codebase uses handler → service → store, you use handler → service → store. Every time.
- **Resource cleanup** — Close connections, cancel contexts, release locks.
- **Concurrency safety** — If the code runs concurrently, protect shared state.

### 4. Three Lines > One Abstraction

```typescript
// GOOD — three similar lines, immediately readable
const name = req.body.name?.trim() || '';
const email = req.body.email?.trim() || '';
const phone = req.body.phone?.trim() || '';
```

```typescript
// BAD — premature abstraction for three fields
const sanitize = (field: string) => (obj: Record<string, any>) =>
  obj[field]?.trim() || '';
const [name, email, phone] = ['name', 'email', 'phone'].map(f => sanitize(f)(req.body));
```

Duplication is cheaper than the wrong abstraction. Extract only when:
- The pattern repeats 4+ times
- The logic is complex enough that a name adds clarity
- The codebase already has a utility for this

### 5. No Gold-Plating

Agents must implement **exactly what was assigned**. Not more.

- Assigned to fix a null check? Fix the null check. Don't refactor the surrounding function.
- Assigned to add an endpoint? Add the endpoint. Don't add caching, rate limiting, and observability.
- Assigned to write tests? Write tests for the specified behavior. Don't add benchmarks and fuzz tests.
- See dead code while fixing a bug? Leave it. It's not in your chain.

**Exception:** If the dead code or adjacent issue is a P0 security/data-integrity problem, document it in P0 DISCOVERIES. Don't fix it — document it.

### 6. Comments: Only When Non-Obvious

```go
// GOOD — explains WHY (not obvious from code)
// Release lock before notification call — holding the mutex during HTTP
// calls caused the 504 timeouts reported in DISPATCH chain 1.
mu.Unlock()
notifyWebhook(event)
```

```go
// BAD — explains WHAT (obvious from code)
// Unlock the mutex
mu.Unlock()
// Send webhook notification
notifyWebhook(event)
```

Add comments for:
- **Why** something unusual is done
- **Business rules** that aren't obvious from code
- **Workarounds** with links to issues/bugs

Don't add comments for:
- What the code does (if the code is clear)
- Function signatures (the types document themselves)
- Section headers (use file organization instead)

---

## Agent Conflict Prevention

Multiple agents work on the same codebase simultaneously. These rules prevent them from breaking each other.

### Territory Is Law

- You modify ONLY files in your declared territory
- You read anything for context — reading doesn't violate territory
- If you need a change in another agent's territory, document it in "Issues for Other Agents" in your completion report
- Shared files (package.json, go.mod, app.module.ts) are LEAD's responsibility

### Import Boundaries

- Import FROM other territories: allowed (read-only dependency)
- Modify files IN other territories: never
- Add exports TO other territories: never (document the need instead)

### Interface Contracts

When your code depends on another agent's work:

1. **Check if the interface exists** — If the function/type is already defined, use it as-is
2. **If it doesn't exist yet** — Code against the expected interface from the task doc, add a clear comment noting the dependency
3. **Never stub another agent's work** — Don't write placeholder implementations in their territory

### Build Verification

After completing all chains, every agent must run:

```bash
[build command]    # Must succeed — your code compiles with the rest
[test command]     # Must pass — your code doesn't break existing tests
```

If the build fails because of another agent's incomplete work (Wave 1 vs Wave 2 timing), document the dependency and mark the chain as BLOCKED. Don't hack around it.

---

## Quick Checklist

Before reporting completion, every agent verifies:

- [ ] Code matches codebase conventions (naming, patterns, style)
- [ ] No unnecessary abstractions or utility functions
- [ ] No code outside assigned territory
- [ ] All error paths handled
- [ ] No swallowed errors or TODO placeholders
- [ ] Architectural patterns followed (handler → service → store, etc.)
- [ ] Build command passes
- [ ] Test command passes
- [ ] Completion report filled out completely

---

**Related Documents:**
- [core/methodology.md](../core/methodology.md) — Chain execution protocol (trace → fix → verify)
- [core/anti-patterns.md](../core/anti-patterns.md) — AP-09: The Refactor Trap
- [config/dispatch-styles.md](dispatch-styles.md) — Optimal Path block (BEFORE/WHILE/AFTER)
- [templates/completion.md](../templates/completion.md) — Completion report template
