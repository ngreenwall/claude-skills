---
name: handoff
description: >-
  End-of-session workflow to leave durable project context for the next chat.
  Use whenever the user says "handoff," "wrap up," "end of session," "before we
  end," "close out this session," "let's wrap," "we're done," "wrap this up,"
  "closing out," "starting a new session," or "before I start fresh." Do NOT
  use for new-project bootstrap or missing baseline scaffolding; use
  project-init for that. Do NOT use for general project status questions or
  mid-session context checks.
---
# SKILL: Handoff

Leave durable project context for the next chat. Do not paste a full conversation recap or duplicate what already lives in project context files (CLAUDE.md, AGENTS.md, README.md, etc.).

In **Recent session notes**, write **Shipped** as facts the next agent needs (code behavior, screens, paths, APIs), not narration about editing documentation or session housekeeping.

## PROTOCOL (run in this exact order)

### 0. Early exit for no-op sessions

Judge by session content first: did this session create or edit any files, or make any commits, including ones made mid-conversation outside this flow, and in prior `handoff` runs or a separate context-update pass earlier in the same session? If yes, there's work to log, continue from Step 1 regardless of what git status shows.

Only reach for `git status --porcelain` (terminal available, project is a git repo) as a confirmation check, and only once session content already points to nothing having happened, never as the primary signal. A clean working tree doesn't by itself mean nothing changed, earlier work may have already been committed and pushed outside the handoff flow, and that's real shipped work that still needs logging. Skip to Step 13 with a lightweight report only when session content says nothing happened: state that nothing changed this session, and mark every checklist line as no changes/already up to date.

If no terminal is available, or the project isn't a git repo, git status can't help at all, judge by session content alone.

### 1. Detect bootstrap gaps and route to project-init when needed
Check whether baseline scaffolding is missing:
- `CLAUDE.md`
- `README.md`
- `.cursor/rules/context-router.md`

If two or more of the files above are missing, stop this workflow and recommend running `project-init` first. Resume `handoff` after bootstrap is complete.

### 2. Locate or create WORKLOG.md
Always use `docs/WORKLOG.md`. Save this path as `$WORKLOG_PATH`, every later step uses it.

If `docs/WORKLOG.md` does not exist, create it (creating `docs/` if needed) with this starter template:

```markdown
# Worklog

## Now / Next
<!-- Checklist. "Now" items first (1-3 max), then queued items. -->
- [ ]

### Ideas
<!-- Unscoped, someday. -->

## Session notes
<!-- Newest entry at top. Format: YYYY-MM-DD | Shipped: ... | Next: ... | Blockers: ... -->
<!-- Archive to docs/archive/worklog-YYYY-MM.md when list exceeds 10 entries -->
```

If an existing worklog is missing the **Now / Next** section (older layout), add it at the top of the file before proceeding.

### 3. Read recent entries
Open `$WORKLOG_PATH` → the session notes section. Read the newest 1-2 dated entries for dedup context.

### 4. Archive if needed
If the session list already has 10 entries, archive just enough of the oldest entries to `docs/archive/worklog-YYYY-MM.md` to keep the list at or under 10 after adding the new one, typically just the single oldest entry. Do this before writing so the list never exceeds 10.

### 5. Add or update one session entry
Get today's real date before writing, run `date +%F` (or otherwise confirm the actual date). Do not guess or reuse a date from an older entry, some tools do not receive the current date automatically.

Follow the template and ordering rules in that section:
- Format: `YYYY-MM-DD | Shipped: ... | Next: ... | Blockers: ...` (omit Blockers if none)
- Newest entry stays at the top
- **Same-day entries are separate by default.** Multiple distinct changes on the same date get their own entries (same date repeated is fine), don't merge unrelated topics into one line just because they share a date. Only edit the latest entry in place when it's a true near-duplicate, the same unfinished topic continuing (e.g. two consecutive `handoff` runs before the work is done), not merely the same day.
- Write Shipped as facts, not narration
- **Shipped is a semicolon-separated list of short clauses, not prose paragraphs.** Each clause is a fact: what changed, a file/path, a behavior, or a commit hash, not a multi-sentence explanation.
- **No time-of-day sub-narration.** Don't split one day's work into "LATER SAME DAY" / "EVENING" / "LATE EVENING" sections. Describe the end state and what changed, not the sequence of how it got there.
- **Soft length guardrail:** if a drafted Shipped field runs past roughly 500 characters, cut iteration detail first (approaches tried and abandoned within the session) and keep final behavior, paths touched, and any commit hash. The worklog is next-session context, not a project diary.
- If something from this session is being logged as a PROJECT.md decision (Step 6), reference it in one clause instead of restating its rationale, the worklog carries what happened, PROJECT.md carries why

### 6. Update spec and decision content

Find where spec and decision content lives: `docs/PROJECT.md` (merged spec + decision log) if it exists, otherwise the "What this is" and "Key decisions" sections in `CLAUDE.md`.

If neither exists, skip this step.

Review the session for anything that should be added or changed in whichever location exists:

**Spec content:** flag if any of these changed this session:
- What the project does or why it exists
- Who it's for
- Goals or success criteria
- What's explicitly out of scope
- Key constraints (stack, platform, team, timeline)

**Decision content:** flag if any of these occurred this session:
- A key architecture, design, or scope decision was made
- An alternative was explicitly ruled out
- A previous decision was reversed or updated

**Correcting an existing entry:** when a past decision turns out to be wrong or incomplete, don't append a second "(Correction, ...)" onto an entry that already has one. Instead, mark the original entry superseded (e.g. `Superseded by [YYYY-MM-DD entry below].`) and write a fresh entry stating the current understanding as one coherent fact. This keeps each entry scannable and means the outdated one archives out immediately under the rule below, rather than accumulating corrections indefinitely.

**Ordering:** new decision entries append at the bottom (oldest-first), unlike the worklog's newest-at-top ordering, that's what makes "Superseded by [YYYY-MM-DD entry below]" accurate.

**Decision log archiving:** if the decision log already has 15 entries, archive the oldest to `docs/archive/decisions-YYYY.md` (year of the archived entries) before adding a new one, mirroring the worklog's archive-at-10 rule. Also archive any entry already marked superseded, regardless of count, once a newer decision replaces it. The live log should never exceed 15 non-superseded entries.

For each proposed change, show it in this format before writing, including any archive move (which file it's moving to and what's being removed from the live log) as one of the proposed changes, not something applied silently after confirmation:

> **[file path]** Action (add|update|remove|archive): proposed content
> *Why:* one-line rationale

If nothing qualifies, say "No spec or decision updates needed" and move on.

Wait for confirmation before writing. If the user approves, apply all at once. Use today's date on new decision entries.

**Escalation:** if the "What this is" and "Key decisions" sections in `CLAUDE.md` exceed roughly a screen of content, propose moving them to `docs/PROJECT.md` (spec section + decision log section), leaving a one-line pointer in `CLAUDE.md`. Apply only with user confirmation, and note the move in the completion checklist.

### 7. Update the task checklist

Open the **Now / Next** section at the top of `$WORKLOG_PATH` (Step 2 guarantees it exists).

- Review the current items. For each item that matches something in the Shipped field of the session entry you just wrote, delete it (don't just check it off), so the list stays clean. The session notes already have the history.
- Keep 1-3 active items at the top; pull queued items up as active ones clear.
- Add any new next actions from this session's Next field.
- Leave **Ideas** untouched unless the user explicitly moved something.

### 8. README health check
(If terminal access is unavailable, use read/list file tools instead of shell commands for this step and Step 9.)

The README should give any agent (or human) enough to pick up the project cold. Check for and update these sections:

- **What this is:** one paragraph on what the project does and why it exists
- **How to run it:** exact commands to install dependencies and start the project locally
- **How to test or preview:** how to verify changes work (test command, dev server URL, preview steps)
- **Key configuration:** any env vars, config files, or credentials needed to run it

Also run a directory listing of the project root and compare against any existing project structure section. If the structure has changed, update it.

If a section is missing or clearly stale, add or update it. If the README is accurate and complete, say "README is up to date" and move on.

### 9. Update other project docs
First, check whether the project's CLAUDE.md (or AGENTS.md if no CLAUDE.md exists) already contains a line telling the agent to check WORKLOG.md at session start. If not, add this line to the appropriate file:

```
Check $WORKLOG_PATH for the current task list and recent session notes before starting work.
```

Then verify a Cursor context router rule exists at `.cursor/rules/context-router.md`. If it is missing or stale, create or replace it with this canonical content:

```markdown
For this repository, read `CLAUDE.md` before any substantive work to load project conventions and workflow constraints.

Then read `docs/WORKLOG.md` for the current task list, latest shipped work, and blockers.

Treat `CLAUDE.md` and context notes as supplemental project context alongside system/developer instructions and `.cursor/rules`. If guidance conflicts, prioritize system/developer instructions first, then `.cursor/rules`, then `CLAUDE.md` and context files. If this project's `CLAUDE.md` conflicts with your global context file, the project rule wins only when it explicitly states it's overriding that specific global rule; an unmarked conflict is treated as accidental drift, and the global rule stays authoritative.
```

Then update CLAUDE.md, AGENTS.md, or other context files only when something else durable changed this session, new conventions, removed patterns, renamed paths. Do not update these for session housekeeping or doc edits. If nothing else applies, say "No additional doc updates needed."

Before adding any convention to CLAUDE.md, check it is not already present and skip if it is. This keeps the write idempotent so a duplicate cannot appear if a separate context-update pass already logged the same thing.

### 10. Noise check
Search `$WORKLOG_PATH` for vague language. Substitute the resolved path for `$WORKLOG_PATH` in the command below, it is a placeholder, not a real shell variable, so the literal command will not work unexpanded (e.g. use `docs/WORKLOG.md`):
```bash
grep -Ein "maybe|investigate later|brainstorm" docs/WORKLOG.md
```
Fix any matches in dated session entries only. Ignore matches elsewhere in the file.

### 11. Starter prompt (optional)
If carryover context matters for the next chat, write a one-paragraph starter prompt the user can paste into a new thread. Include: what was just shipped, the immediate next step, and any active blocker. Do not include instructions about where to find context, CLAUDE.md handles that automatically. Otherwise omit it.

### 12. Commit and push all changes
Claude Code / terminal only, if terminal access is unavailable (e.g. Cursor without shell), skip this step and note it as manual follow-up.

If the project is not a git repo (no `.git`), skip this step and note it in the checklist; the user chose no-git at init. Otherwise check for uncommitted changes with `git status --porcelain`. If there are none, skip. If there are, stage everything and commit in one push, this is a single-contributor personal repo workflow. If no remote is configured, commit but skip the push and note it. For a shared or multi-contributor repo, propose a branch and PR instead of pushing straight to main.

```bash
git add -A && git commit -m "Handoff: <YYYY-MM-DD> session notes" && git push
```

### 13. Report completion checklist (required)
Return this exact structure at the end:

```markdown
Handoff checklist:
- [x] Session note in `<WORKLOG_PATH>` (added|updated)
- [x] Spec and decision content reviewed (updated|no updates needed|escalated-to-PROJECT.md|skipped-not-found)
- [x] Task checklist updated (yes|nothing to change)
- [x] README health check (updated|already up to date)
- [x] Context routing docs (`CLAUDE.md`/`AGENTS.md` and `.cursor/rules/context-router.md`) (updated|already good)
- [x] Noise check complete
- [x] Committed and pushed (yes|no changes|skipped-no-terminal)

Manual follow-up:
- <only include items that still need user action>
```

## Tool-specific notes

**Both tools:** run this skill for project session state. If you also keep a separate end-of-session pass for capturing lasting preferences or patterns (not project-specific), run that afterward, they cover different things and complement each other.

This skill should keep `.cursor/rules/context-router.md` present and current so Cursor reliably loads `CLAUDE.md` and `$WORKLOG_PATH`.
