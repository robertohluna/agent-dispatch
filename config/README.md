# Config

> Dispatch configuration files that control agent behavior, code quality, and prompt assembly.

---

## Files

| File | Purpose |
|------|---------|
| [dispatch-styles.md](dispatch-styles.md) | Execution style blocks appended to every activation prompt — mesh mode triggers, intensity protocol, optimal path (BEFORE/WHILE/AFTER), prompt assembly order, completion doc requirements |
| [code-standards.md](code-standards.md) | Agent coding discipline — write minimal correct code, match codebase conventions exactly, no over-engineering, no under-engineering, conflict prevention between agents |

## How Config Gets Used

The **dispatcher** (the AI that plans the sprint) reads these files and applies them:

1. `dispatch-styles.md` blocks get **appended to every activation prompt** in the order specified by the Assembly Order section
2. `code-standards.md` gets **included in the context reading list** for every agent — agents read it before writing any code

## Customization

### Project-Specific Overrides

Create `config/dispatch-styles-[project].md` for project-specific style blocks. The dispatcher reads the base file first, then applies project overrides.

### Adding Your Own Config

Add any project-specific configuration files here:
- `config/territory-[project].md` — Custom territory mappings
- `config/build-commands.md` — Project-specific build/test commands
- `config/team-conventions.md` — Team-specific coding conventions

The dispatcher reads everything in `config/` and applies it to the sprint plan.

---

**Related Documents:**
- [templates/activation.md](../templates/activation.md) — Where dispatch styles get injected
- [guides/customization.md](../guides/customization.md) — Adapting territories and conventions
- [core/workflow.md](../core/workflow.md) — Sprint lifecycle
