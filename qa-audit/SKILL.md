---
name: qa-audit
description: >-
  Static-analysis QA pass over instruction/config files (SKILL.md, CLAUDE.md,
  your global context file), not code. Spawns an Opus subagent per file to
  check for contradictions, dead logic, unstated dependencies, and style
  violations, then proposes fixes for confirmation before applying. Use when
  the user says "QA this skill," "audit this file," "run a QA pass on X," "QA
  SKILL.md," "audit CLAUDE.md," or "check this file for consistency issues."
  Do NOT use for auditing whether global context loaded correctly in the live
  session (use drift-check instead). Do NOT use for running a skill against
  real prompts to grade its outputs (that's skill-creator's eval feature,
  dynamic testing). Do NOT use for reviewing code diffs (use /code-review
  instead); this skill never touches code, only instruction/config prose.
---
# SKILL: QA Audit

A static-analysis QA pass over instruction and config prose. Recognized targets: `SKILL.md` files, `CLAUDE.md`, your global context file (the standing instructions your AI tool loads every session, e.g. `~/.claude/CLAUDE.md`), `README.md`, and other instruction/config prose files. It never executes or evaluates the target against real prompts, it reads it and reasons about it.

This is NOT:
- **drift-check**, that audits whether global context actually loaded and is being followed in a *live session*. This skill runs a static read of a file anytime, session state doesn't matter.
- **skill-creator's eval feature**, that runs a skill against real prompts and grades its outputs, dynamic testing. This skill never executes the target.
- **/code-review**, that's for code diffs. This skill never touches code, only instruction/config prose.

## PROTOCOL

### Step 1: Resolve target(s) and guard against code

Identify the file(s) to QA.

Before doing anything else, check whether the target looks like code: it's a code file if the extension isn't `.md`/`.mdc` (e.g. `.ts`, `.py`, `.js`). If so, stop immediately, don't spawn any agent, and tell the user in one line: "That's a code file, not instruction/config prose, use /code-review for that."

If the extension is `.md`/`.mdc` but the path isn't one of the recognized instruction/config locations (a `SKILL.md` inside a skills folder, `CLAUDE.md`, your global context file, `README.md`), don't auto-reject it, ask the user to confirm it's instruction/config prose before proceeding.

If the file is managed by a sync tool that generates copies elsewhere (e.g. a tool that regenerates `.claude/` or `.cursor/` files from a single source of truth), always resolve to that source file, never a generated copy, editing a generated copy gets silently overwritten on the next sync run. Skip this check if you don't use such a tool, most projects edit `SKILL.md`/`CLAUDE.md` directly.

### Step 2: Gather context

Read related docs (README, decision logs, sibling skills) so deliberate, documented choices aren't misflagged as bugs.

### Step 3: Spawn QA agents

Launch one `Agent` tool call per target file, `model: "opus"`, using the prompt template below with placeholders filled in for that file. If multiple files, launch all of them in the same message so they run in parallel.

**QA prompt template (fill in placeholders per file):**
```
You're QAing [file(s)/system] in [repo/location]. Purpose: [what it's supposed to do].

Check for:
1. Unstated dependencies, a step/section that assumes something exists or was already read/produced, but nothing upstream actually does that.
2. Internal contradictions, one part assumes X, another assumes not-X.
3. Dead or unreachable logic, steps that can never trigger, or reference something renamed/removed.
4. Style violations, [house style rules, e.g. no em dashes, no vague hedge words].
5. Cross-file consistency, if multiple files reference each other, confirm paths/claims/section names still match.
6. Don't trust any line numbers given, find the real match by content first.

Read [related docs, e.g. README, decision log] for context so deliberate, documented choices aren't flagged as bugs.

Output one finding per issue: location, quoted text, why it's a problem, proposed fix. Don't apply fixes, just report. Rank by likelihood of causing real bad behavior vs. just reading oddly. End with a summary count by severity.
```

If the target carries YAML frontmatter (any `SKILL.md`, or a global/rule source file), append this 7th check item to the prompt:
```
7. Frontmatter/convention violations: kebab-case name (for skills), pushy description with named trigger phrases, negative triggers present if the skill is narrow or heavy, all required frontmatter fields present (name, description).
```

### Step 4: Compile findings

Merge results across all target files into one list, ranked by severity (likelihood of causing real bad behavior vs. just reading oddly). End with a summary count by severity.

### Step 5: Propose fixes, wait for confirmation

Show the proposed fix for each finding. Wait for explicit user confirmation before editing anything.

### Step 6: Apply fixes

On confirmation, apply the fixes using the current session's running model, no need to re-invoke Opus for mechanical edits.

### Step 7: Regenerate if sync-managed

If the edited file is managed by a sync tool that generates copies elsewhere (see Step 1), rerun that tool's generate/sync command so the copies pick up the change. Skip this step entirely if the file isn't sync-managed, that's the common case.

### Step 8: Commit

Single-contributor repo convention: commit (and push, if this repo is push-direct-to-main). For a shared or multi-contributor repo, propose a branch and PR instead.
