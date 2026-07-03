# claude-skills

Public, machine-agnostic Claude Code skills anyone can install, no dependency on any private repo or personal setup.

A "skill" here is just a folder with a `SKILL.md` file inside it, plain markdown instructions that tell Claude Code how to run a specific workflow. No code, no build step. `SKILL.md` with trigger-phrase auto-invocation is a Claude Code mechanism specifically, Cursor doesn't scan `.claude/skills/` or fire skills off trigger phrases.

That said, `project-init` and `handoff` also write and maintain files Cursor reads natively (`.cursor/rules/context-router.md`, `CLAUDE.md`, `docs/WORKLOG.md`), so once either has run once (in Claude Code), Cursor picks up the same project conventions and worklog automatically. `drift-check` has no Cursor-side output, it's Claude Code only.

## Contents

- [How to install (Claude Code)](#how-to-install-claude-code)
- [Using these in Cursor](#using-these-in-cursor)
- [Skills](#skills)
  - [project-init](#project-init)
  - [handoff](#handoff)
  - [drift-check](#drift-check)
- [License](#license)

## How to install (Claude Code)

Copy a skill's folder into your project's `.claude/skills/` directory (or `~/.claude/skills/` for a skill you want available everywhere), keeping the folder name and the `SKILL.md` inside it.

## Using these in Cursor

Cursor won't auto-discover a `SKILL.md`, but you can get the same workflow by turning it into a Cursor project rule:

1. Copy a skill's `SKILL.md` content.
2. Save it as `.cursor/rules/<skill-name>.mdc` in your project.
3. Keep the existing `description:` line in the frontmatter (that's what lets Cursor's agent decide when it's relevant), and add `alwaysApply: false` so it only loads when needed rather than on every message.
4. Invoke it by typing `@<skill-name>` in a Cursor chat, or let the agent attach it automatically when your request matches the description.

## Skills

### [project-init](project-init/SKILL.md)

One-time bootstrap for a new repo. Asks two questions: simple or in-depth project? Will it live on git? Then creates only what that scope needs, `.gitignore` + MIT `LICENSE` if git, plus `README.md`, `CLAUDE.md`, and a Cursor context router. In-depth projects also get `docs/WORKLOG.md` for a task checklist and session notes.

- **Use it:** first thing in a brand-new repo, before your first `handoff`.
- **Don't use it:** for end-of-session updates in a repo that's already set up, that's `handoff`.

### [handoff](handoff/SKILL.md)

End-of-session workflow. Logs one dated session entry (`Shipped` / `Next` / `Blockers`) in `docs/WORKLOG.md`, trims the Now/Next task checklist, checks the README is still accurate, and commits (pushes if a remote exists). Creates `docs/WORKLOG.md` if it doesn't exist yet. If there's meaningful carryover, it also writes a one-paragraph starter prompt you can paste into your next chat, skipped when there's nothing worth carrying over.

- **Use it:** when wrapping up a work session on a project that already has `project-init`'s scaffolding. Also good when your session is getting long and you want to start a fresh one, running it first preserves state before context gets compacted or lost.
- **Don't use it:** for new-project bootstrap, or general "what's the status" questions mid-session.

### [drift-check](drift-check/SKILL.md)

Manual audit that proves your **global** context file (the standing instructions your AI tool loads every session, e.g. `~/.claude/CLAUDE.md`) is still loaded and actually being followed. The audit pinpoints where in the file it dropped: top, middle, or bottom. Long sessions and context compaction silently drop rules; this catches it.

This is scoped to the global file on purpose, it's usually the long, dense one that accumulates personal preferences and workflow habits over time, which is exactly what makes mid- or end-of-file drift possible. A project-level `CLAUDE.md` is normally short enough to get re-read naturally each session, so it's out of scope here by default. If you notice yourself violating a project-specific rule mid-session, that's a signal to consider a lightweight check for that file too, but don't set one up ahead of need.

- **Use it:** only when you explicitly ask for it, e.g. "run a drift check." It should never trigger on its own.
- **Don't use it:** as part of normal task work, it's a deliberate, occasional audit, not a background check.

**One-time setup required before first use.** This skill can't run until your global context file is prepared:

1. **Find or create your global context file.** In Claude Code, that's `~/.claude/CLAUDE.md`, a plain markdown file, edit it directly with any editor. In Cursor, it's your User Rules (in Cursor Settings) instead. If you don't have one and want one, create it yourself, this skill won't create it for you.
2. **Generate three canary tokens.** Pick three short, random, unguessable strings, e.g. `sunfish-quartz-14`, `pebble-orchard-08`, `willow-basalt-71`. Use a different one for each position (top, middle, bottom) so a missing token tells you exactly where the drop happened.
3. **Place and label them in your global context file.** Label each clearly: `Canary (top):`, `Mid-file canary:`, `Canary (bottom):`. Put the top one next to whatever rule you consider most important. Tell your context file not to output these tokens during normal work, only when a drift check runs.
4. **Pick your own "always-on" rules to check every time.** A common one is a response-format prefix, if you want your assistant to start every reply with a tag (e.g. "Agent:", "Bot:", your own name, or nothing at all), name that convention in your context file and list it as one of the core rules in `drift-check/SKILL.md`'s Check 2 section. Skip this if you don't use a prefix convention.
5. **Note the file path** so the skill knows what to re-read when checking.

A filled-in example is at [`drift-check/example-global-CLAUDE.md`](drift-check/example-global-CLAUDE.md). The file has a note at the top marking what not to copy, everything below the divider is the actual template: swap in your own preferences and rules, generate your own three tokens, and save it as `~/.claude/CLAUDE.md`.

See the "Setup" section in `drift-check/SKILL.md` for the full walkthrough.

## License

MIT, see [`LICENSE`](LICENSE).
