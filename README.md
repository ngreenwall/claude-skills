# claude-skills

Public, machine-agnostic Claude Code skills anyone can install, no dependency on any private repo or personal setup.

## Skills

- **project-init/** — one-time bootstrap for a new repo: git, `.gitignore`, `LICENSE`, `README.md`, `CLAUDE.md`, and a Cursor context router.
- **handoff/** — end-of-session workflow that logs a dated session entry, updates the task checklist, and commits/pushes.
- **drift-check/** — manual audit that confirms your context file is still loaded and being followed, using canary tokens you generate yourself.

## How to install

Copy a skill's folder into your project's `.claude/skills/` directory (or `~/.claude/skills/` for a skill you want available everywhere), keeping the folder name and the `SKILL.md` inside it. No build or compile step.

## Key configuration

`drift-check` requires a one-time setup step per context file: see the "Setup" section in `drift-check/SKILL.md`.
