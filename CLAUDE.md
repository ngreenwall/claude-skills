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
- [2026-07-09] What gates these three specific skills isn't install path, it's file access and CLAUDE.md auto-load per surface. Claude.ai chat has no file access, so none of the three can do anything there. Cowork has file access via a connected folder, so project-init and handoff could genuinely work there. drift-check still can't: its mechanism depends on CLAUDE.md auto-loading every session, which Cowork doesn't do automatically. (Correction, 2026-07-09: chat does support manually attaching files to one conversation; what it actually lacks is persistent access to a local project folder.)
- [2026-07-09] Verified README's external claims against primary sources: the official Claude support article (support.claude.com) for the Claude.ai/Cowork skill-upload path, and Cursor's rules docs for .mdc frontmatter behavior. Confirmed Cursor still has no native Claude Skills support (open community feature request, unresolved). Fixed a stale TOC anchor and two markdown links that had gotten wrapped in backticks (rendering as literal text instead of links).
- [2026-07-09] Cursor has no skills mechanism at all (SKILL.md auto-invocation is Claude Code specific). Claude.ai/Cowork's account-level install path is Customize > Skills > "+" > "+ Create skill" > "Upload a skill".
- [2026-07-09] drift-check is Claude Code only; removed the Cursor adaptation paragraph from Setup, it contradicted the README's own claim that Cursor isn't supported, and kept the skill scoped to one tool it's actually validated against.
- [2026-07-09] Project-vs-global CLAUDE.md conflicts resolve as explicit-override-only: a project `CLAUDE.md` rule beats a global rule only when it explicitly states it's overriding that specific global rule; an unmarked conflict is treated as accidental drift and the global rule stays authoritative. Rejected "project always wins" (too permissive, would swallow real drift) and "global always wins" (breaks legitimate per-project overrides). Applied to `context-router.md`'s canonical template (`handoff`, `project-init`, and the live `.cursor/rules/context-router.md`) and to `drift-check`'s Check 2. Not yet tested against a real project-vs-global conflict in this repo, no example currently exists here.
