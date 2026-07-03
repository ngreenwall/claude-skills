# claude-skills

Public, machine-agnostic rewrites of three Claude Code skills (project-init, handoff, drift-check), stripped of any personal or machine-specific setup so anyone can install them.

Check docs/WORKLOG.md for the current task list and recent session notes before starting work.

## What this is
A public repo of installable Claude Code skills. Each skill is a folder with a `SKILL.md`, no build step, no dependency on any private repo. Rewritten from private originals in `~/my-ai-agent` to strip personal/machine-specific setup (names, canary tokens, Cursor linkage scripts, `~/my-ai-agent` paths).

## Key decisions
- [2026-07-03] drift-check scoped to the global context file only, not project-level CLAUDE.md/AGENTS.md, because global files are the long, dense ones where depth-based drift actually happens; project files are short and re-read naturally each session.
- [2026-07-03] drift-check never auto-creates a global context file if one doesn't exist, that's a bigger machine-level decision left to the user.
- [2026-07-03] project-init only creates `CLAUDE.md`, never `AGENTS.md`, but `handoff` still recognizes an existing `AGENTS.md` from another tool.
- [2026-07-03] Every skill rewrite is validated by running its protocol live in a scratch directory before committing, proofreading the text alone isn't enough.
