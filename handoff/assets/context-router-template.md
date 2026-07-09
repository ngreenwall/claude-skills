# Cursor context-router canonical content

Used in Step 9 when `.cursor/rules/context-router.md` is missing or stale. The block already uses the resolved path (e.g. `docs/WORKLOG.md`).

```markdown
For this repository, read `CLAUDE.md` before any substantive work to load project conventions and workflow constraints.

Then read `docs/WORKLOG.md` for the current task list, latest shipped work, and blockers.

Treat `CLAUDE.md` and context notes as supplemental project context alongside system/developer instructions and `.cursor/rules`. If guidance conflicts, prioritize system/developer instructions first, then `.cursor/rules`, then `CLAUDE.md` and context files. If this project's `CLAUDE.md` conflicts with your global context file, the project rule wins only when it explicitly states it's overriding that specific global rule; an unmarked conflict is treated as accidental drift, and the global rule stays authoritative.
```
