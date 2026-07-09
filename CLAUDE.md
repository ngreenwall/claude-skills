# claude-skills

Public, machine-agnostic rewrites of three Claude Code skills (project-init, handoff, drift-check), stripped of any personal or machine-specific setup so anyone can install them.

Check docs/WORKLOG.md for the current task list and recent session notes before starting work.

## What this is
A public repo of installable Claude Code skills. Each skill is a folder with a `SKILL.md`, no build step, no dependency on any private repo. Rewritten from private originals in `~/my-ai-agent` to strip personal/machine-specific setup (names, canary tokens, Cursor linkage scripts, `~/my-ai-agent` paths).

## Key decisions
- [2026-07-03] No separate Cursor-global-rules template was built for drift-check. Content would duplicate example-global-CLAUDE.md, and Cursor's exact User Rules storage mechanism isn't confirmed, so the SKILL.md points users to adapt the existing example instead.
- [2026-07-03] SKILL.md auto-invocation is Claude Code specific; Cursor doesn't scan .claude/skills/. README now separates what Cursor inherits for free (context-router.md, CLAUDE.md, WORKLOG.md) from what requires manual conversion to a .cursor/rules/*.mdc project rule. (Correction, 2026-07-09: this is only true of the local-folder install path. Claude.ai and Cowork have a separate account-level install path, Settings > Customize > Skills > Upload, and skills work identically there once installed. Cursor genuinely has no skills mechanism at all, that part of the original decision stands.)
- [2026-07-03] drift-check scoped to the global context file only, not project-level CLAUDE.md/AGENTS.md, because global files are the long, dense ones where depth-based drift actually happens; project files are short and re-read naturally each session.
- [2026-07-03] drift-check never auto-creates a global context file if one doesn't exist, that's a bigger machine-level decision left to the user.
- [2026-07-03] project-init only creates `CLAUDE.md`, never `AGENTS.md`, but `handoff` still recognizes an existing `AGENTS.md` from another tool.
- [2026-07-03] Every skill rewrite is validated by running its protocol live in a scratch directory before committing, proofreading the text alone isn't enough.
- [2026-07-09] What gates these three specific skills isn't install path, it's file access and CLAUDE.md auto-load per surface. Claude.ai chat has no file access, so none of the three can do anything there. Cowork has file access via a connected folder, so project-init and handoff could genuinely work there. drift-check still can't: its mechanism depends on CLAUDE.md auto-loading every session, which Cowork doesn't do automatically.
