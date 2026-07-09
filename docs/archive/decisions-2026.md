# Archived decisions

- [2026-07-03] drift-check scoped to the global context file only, not project-level CLAUDE.md/AGENTS.md, because global files are the long, dense ones where depth-based drift actually happens; project files are short and re-read naturally each session.
- [2026-07-03] drift-check never auto-creates a global context file if one doesn't exist, that's a bigger machine-level decision left to the user.
- [2026-07-03] project-init only creates `CLAUDE.md`, never `AGENTS.md`, but `handoff` still recognizes an existing `AGENTS.md` from another tool.
- [2026-07-03] Every skill rewrite is validated by running its protocol live in a scratch directory before committing, proofreading the text alone isn't enough.
- [2026-07-03] SKILL.md auto-invocation is Claude Code specific; Cursor doesn't scan .claude/skills/. Superseded by [2026-07-09]: Cursor has no skills mechanism at all (SKILL.md auto-invocation is Claude Code specific). Claude.ai/Cowork's account-level install path is Customize > Skills > "+" > "+ Create skill" > "Upload a skill".
- [2026-07-03] No separate Cursor-global-rules template was built for drift-check. Content would duplicate example-global-CLAUDE.md, and Cursor's exact User Rules storage mechanism isn't confirmed, so the SKILL.md points users to adapt the existing example instead. Superseded by [2026-07-09]: drift-check is Claude Code only; removed the Cursor adaptation paragraph from Setup, it contradicted the README's own claim that Cursor isn't supported, and kept the skill scoped to one tool it's actually validated against.
