# claude-skills

> Public skills for Claude Code, portable to Cursor and Claude.ai/Cowork, no dependency on any private repo or personal setup.

A "skill" is a folder with a `SKILL.md` file inside it (YAML frontmatter with `name` and `description`, then markdown instructions) that tells Claude how to run a specific workflow. Skills can optionally bundle scripts or other resources:

```
my-skill/
├── SKILL.md
├── scripts/       # executable code
├── references/    # documentation
├── assets/        # templates, resources
```

None of the five skills here use `scripts/` or `references/`, no code, no build step. `handoff` and `project-init` each ship an `assets/` folder with starter templates (worklog, CLAUDE.md, Cursor context-router content) that `SKILL.md` loads on demand instead of inlining.


## Contents

- [How to install (Claude Code)](#how-to-install-for-claude-code)
- [Using these in Cursor](#using-these-in-cursor)
- [Using these in Chat or Cowork](#using-these-in-chat-or-cowork)
- [Skills](#skills)
  - [project-init](#project-init)
  - [handoff](#handoff)
  - [drift-check](#drift-check) (Claude Code only)
  - [qa-audit](#qa-audit) (Claude Code only)
  - [token-audit](#token-audit) (Claude Code only)
- [License](#license)


## How to install for Claude Code

Copy a skill's folder into your project's `.claude/skills/` directory (or `~/.claude/skills/` for a skill you want available everywhere), keeping the folder name and the `SKILL.md` inside it.

## Using these in Cursor

Cursor won't auto-discover a `SKILL.md`, but you can get the same workflow by turning it into a Cursor project rule:

1. Copy a skill's `SKILL.md` content.
2. Save it as `.cursor/rules/<skill-name>.mdc` in your project.
3. Keep the existing `description:` line in the frontmatter (that's what lets Cursor's agent decide when it's relevant), and add `alwaysApply: false` so it only loads when needed rather than on every message.
4. Invoke it by typing `@<skill-name>` in a Cursor chat, or let the agent attach it automatically when your request matches the description.

Note: `drift-check` isn't built for Cursor yet, only the Claude Code version ships here, though the mechanism could work since Cursor's User Rules auto-load every session too.

Note: `qa-audit` and `token-audit` both rely on spawning a separate subagent (an `Agent`-tool call) per file, that's a Claude Code mechanism a Cursor project rule can't reproduce. Converting either would collapse to a single self-audit pass with no parallelization, not the same workflow.

## Using these in Chat or Cowork

**Claude.ai Chat** can't do any of this: no persistent access to a project folder, only files you manually attach to one conversation.

**Cowork** has folder access and subagents, so `project-init` and `handoff` could work there. `drift-check` can't: it needs a global `CLAUDE.md` to auto-load every session, and Cowork doesn't do that. `qa-audit` and `token-audit` aren't offered there either, they ship Claude Code only for now.

To install a skill: Customize > Skills > "+" > "+ Create skill" > "Upload a skill" (zip the folder first).


## Skills

### [project-init](project-init/SKILL.md)

> One-time setup that gives a brand-new repo its starting docs and files. Works in Cursor too.

Asks two questions: simple or in-depth project? Will it live on git? Then creates only what that scope needs, `.gitignore` + MIT `LICENSE` if git, plus `README.md`, `CLAUDE.md`, and a Cursor context router. In-depth projects also get `docs/WORKLOG.md` for a task checklist and session notes.

- **Use it:** first thing in a brand-new repo, before your first `handoff`.
- **Don't use it:** for end-of-session updates in a repo that's already set up, that's `handoff`.

What it creates, simple project (git yes):

```
your-project/
├── .gitignore
├── LICENSE
├── README.md
├── CLAUDE.md
└── .cursor/
    └── rules/
        └── context-router.md
```

What it creates, in-depth build (git yes):

```
your-project/
├── .gitignore
├── LICENSE
├── README.md
├── CLAUDE.md
├── docs/
│   └── WORKLOG.md
└── .cursor/
    └── rules/
        └── context-router.md
```

Skip the git question with "no" and `.gitignore`/`LICENSE` are left out. `.cursor/rules/` is created either way, it's just a router file that's harmless if you don't use Cursor.

### [handoff](handoff/SKILL.md)

> End-of-session wrap-up that logs progress and commits your work. Works in Cursor too.

Logs one dated session entry (`Shipped` / `Next` / `Blockers`) in `docs/WORKLOG.md`, trims the Now/Next task checklist, checks the README is still accurate, and commits (pushes if a remote exists). Creates `docs/WORKLOG.md` if it doesn't exist yet. If there's meaningful carryover, it also writes a one-paragraph starter prompt you can paste into your next chat, skipped when there's nothing worth carrying over.

- **Use it:** when wrapping up a work session on a project that already has `project-init`'s scaffolding. Also good when your session is getting long and you want to start a fresh one, running it first preserves state before context gets compacted or lost.
- **Don't use it:** for new-project bootstrap, or general "what's the status" questions mid-session.


### [drift-check](drift-check/SKILL.md)

> On-demand audit that checks whether your global context file is still being followed. Claude Code only.

Checks your **global** context file (the standing instructions your AI tool loads every session, e.g. `~/.claude/CLAUDE.md`) and pinpoints where in the file it dropped: top, middle, or bottom. Long sessions and context compaction silently drop rules; this catches it.

Scoped to the global file on purpose: it's usually the long, dense one that accumulates preferences over time, so it's most likely to drift mid- or end-of-file. A project-level `CLAUDE.md` is short enough to get re-read naturally each session, so it's out of scope by default. If you catch yourself violating a project rule mid-session, consider a lightweight check there too, but don't set one up ahead of need.

- **Use it:** only when you explicitly ask for it, e.g. "run a drift check." It should never trigger on its own.
- **Don't use it:** as part of normal task work, it's a deliberate, occasional audit, not a background check.

**One-time setup required before first use.** This skill can't run until your global context file is prepared:

1. **Find or create your global context file.** Claude Code: `~/.claude/CLAUDE.md` (plain markdown, edit directly). Cursor: your User Rules (in Cursor Settings). Don't have one? This skill won't create it for you, use the example at [`drift-check/example-global-CLAUDE.md`](drift-check/example-global-CLAUDE.md) (note at the top marks what not to copy; everything below the divider is the template).
2. **Generate three canary tokens.** Pick three short, random, unguessable strings, e.g. `sunfish-quartz-14`, `pebble-orchard-08`, `willow-basalt-71`. Use a different one for each position (top, middle, bottom) so a missing token tells you exactly where the drop happened.
3. **Place and label them in your global context file.** Label each clearly: `Canary (top):`, `Mid-file canary:`, `Canary (bottom):`. Put the top one next to whatever rule you consider most important. Tell your context file not to output these tokens during normal work, only when a drift check runs.
4. **Pick your own "always-on" rules to check every time.** A common one is a response-format prefix, if you want your assistant to start every reply with a tag (e.g. "Agent:", "Bot:", your own name, or nothing at all), name that convention in your context file and list it as one of the core rules in `drift-check/SKILL.md`'s Check 2 section. Skip this if you don't use a prefix convention.
5. **Note the file path** so the skill knows what to re-read when checking. Only needed if your global context file lives somewhere other than the standard location (e.g. `~/.claude/CLAUDE.md`).


### [qa-audit](qa-audit/SKILL.md)

> Static-analysis QA pass over instruction/config prose, not code. Claude Code only.

Spawns one Opus subagent per file (`SKILL.md`, `CLAUDE.md`, your global context file, `README.md`) to check for internal contradictions, dead or unreachable logic, unstated dependencies, style violations, and cross-file consistency. It reads and reasons about the file, it never runs the skill or file against real prompts. Proposes fixes and waits for confirmation before editing anything.

Uses `model: "opus"` by default for the subagent calls. To use a different model, edit the `model: "opus"` line in `qa-audit/SKILL.md`.

- **Use it:** when you want a second, skeptical pass on instruction/config prose you wrote, e.g. "QA this skill" or "audit CLAUDE.md."
- **Don't use it:** to check whether global context actually loaded in a live session (that's `drift-check`), to grade a skill's real-prompt outputs (that's skill-creator's eval feature), or on code diffs (use `/code-review`).


### [token-audit](token-audit/SKILL.md)

> Audits instruction/config prose and this repo's skill architecture for token bloat. Claude Code only.

Spawns one Opus subagent per file (`SKILL.md`, `CLAUDE.md`, your global context file, `README.md`) to flag token-bloat patterns, oversized files, missing navigation, duplicate content across files, and unscoped instructions, then proposes fixes: rewrite, split into a reference file, merge, add navigation, or remove. Single-file mode by default; full-library mode also checks for duplicate content across files and asks which skills you actually use.

Uses `model: "opus"` by default for the subagent calls. To use a different model, edit the `model: "opus"` line in `token-audit/SKILL.md`.

- **Use it:** when you want to cut the token cost of a file or the whole skills library, e.g. "audit tokens on CLAUDE.md" or "is this skill too big."
- **Don't use it:** to find bugs, contradictions, or dead logic (that's `qa-audit`), or on code files.


## License

MIT, see [LICENSE](LICENSE).