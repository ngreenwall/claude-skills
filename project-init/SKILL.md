---
name: project-init
description: >-
  One-time project bootstrap workflow for new repos. Use whenever the user says
  "initialize project," "set up this repo for Cursor and Claude," "new Cursor
  project," "set up Cursor," "bootstrap docs/context," "new project setup," or
  "prepare handoff files." Prefer this before first handoff in a repo. Do NOT
  use for end-of-session updates in active projects; use handoff for that.
---
# SKILL: Project Init

Initialize a project so future sessions and handoffs work consistently in both Cursor and Claude Code. Default to the minimum: three docs plus git. A project earns more files by outgrowing these, not on day 0.

## MISSION
Create the minimum durable scaffolding a project needs on day 0:
- Git repo initialized with a `.gitignore`, and `LICENSE` when the project is meant to be shared
- `CLAUDE.md` for project-facing AI guidance, including a short "What this is" and "Key decisions" section
- `docs/WORKLOG.md` with the task checklist (Now / Next) up top and rolling session notes below
- `README.md` baseline sections so agents and humans can pick up the repo quickly
- `.cursor/rules/context-router.md` for Cursor startup routing

Do NOT create SPEC.md, DECISIONS.md, ROADMAP.md, or PROJECT.md. Spec and decision content starts as sections inside CLAUDE.md; the escalation path (below) covers projects that outgrow that.

## PROTOCOL (run in this exact order)

Invoking this skill is the user's confirmation for its standard outputs: create the files for the chosen scope without per-file approval. Only two questions need answers at either scope: the scope question and the git question. This overrides any general "state proposed changes and wait for confirmation" habit for files this skill is defined to create.

### Step 1: Determine project scope
Ask the user:

> "Is this a simple, smaller project (a script, a one-off page, a config, a quick artifact), or an in-depth build (an app or tool you'll develop over time)?"

Do not infer the answer from file types or folder contents, only the user knows how deep this project goes. Exception: if the user already described the project's depth earlier in the conversation, state the scope you inferred and ask them to confirm instead of asking from scratch.

**Simple project:**
1. Ask one setup question: "Will this live on git?"
   - **Yes:** run `git init` if not already a repo, create a minimal `.gitignore` (`.DS_Store`, `.env`, plus stack-specific entries if detected), and create an MIT `LICENSE`, all automatically, no further questions. Use a different license only if the user names one. Do NOT ask where it will be pushed or create a remote; the completion report covers connecting to GitHub later. After all files are created, make an initial commit (`Initial scaffold`) so the folder is push-ready.
   - **No:** skip git, `.gitignore`, and `LICENSE` entirely.
2. Create or patch `README.md` with just: What this is, How to install/use it, and any key configuration. Skip "How to run it" / "How to test or preview" if they don't apply.
3. Create a minimal `CLAUDE.md` so return sessions start oriented (it's the file AI tools auto-load):

   ```markdown
   # <Project name>

   <One line: what this is and why it exists.>

   Check README.md for how to use and run it. Keep this project simple, no docs/ scaffolding unless it graduates to an in-depth build.
   ```

4. Create `.cursor/rules/context-router.md` so Cursor loads CLAUDE.md too:

   ```markdown
   For this repository, read `CLAUDE.md` before any substantive work to load project conventions and constraints.

   Treat `CLAUDE.md` as supplemental project context alongside system/developer instructions and `.cursor/rules`. If guidance conflicts, prioritize system/developer instructions first, then `.cursor/rules`, then `CLAUDE.md`.
   ```

5. Do not create WORKLOG.md or a `docs/` folder, a simple project doesn't need session scaffolding. If it grows into a real build later, rerun `project-init` and answer "in-depth"; it patches in the missing files.
6. Report completion using the "Simple project" checklist in Step 7 and stop.

**In-depth build:** continue to Step 2.

### Step 2: Verify workspace and detect current state
Confirm you are in the project root and inventory whether these files already exist:
- `CLAUDE.md`
- `docs/WORKLOG.md`
- `README.md`
- `.cursor/rules/context-router.md`

Prefer patching existing files over replacing them.

### Step 3: Ask the git question
Ask: "Will this live on git?" (Skip the question if the project root is already a git repo, treat that as a yes.)

- **Yes:** run `git init` if not already a repo. If `.gitignore` does not exist, create one with sensible defaults for the detected stack (check for `package.json`, `requirements.txt`, `Gemfile`, etc.), at minimum `.DS_Store`, `.env`, `node_modules/` (if JS/TS), and editor/IDE cruft. Create an MIT `LICENSE` automatically (use a different license only if the user names one). No further git questions. Do not ask where the repo will be pushed or create a remote; after all scaffolding steps finish, make an initial commit (`Initial scaffold`) so the folder is push-ready, and the completion report's follow-up block covers connecting to GitHub later.
- **No:** skip git, `.gitignore`, and `LICENSE`. Note in the report that `handoff`'s commit step won't apply until git is initialized.

If `.gitignore` or `LICENSE` already exist, leave them untouched.

### Step 4: Create CLAUDE.md and docs/WORKLOG.md
Ensure the `docs/` folder exists.

If `CLAUDE.md` is missing, create it (fill in what you know from the conversation; leave honest placeholders otherwise):

```markdown
# <Project name>

<One-line description of what this repo is and why it exists.>

Check docs/WORKLOG.md for the current task list and recent session notes before starting work.

## What this is
<!-- One short paragraph: what it does, who it's for, what's out of scope. -->

## Key decisions
<!-- One line per decision: [YYYY-MM-DD] Decision, and why. Newest at top. -->
```

If `CLAUDE.md` exists, patch in the worklog check line and any missing section headers, don't rewrite existing content.

If `docs/WORKLOG.md` is missing, create it:

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

### Step 5: Create or patch README baseline
Ensure `README.md` includes these minimum sections, adapting or omitting any that don't apply:
- What this is
- How to run it
- How to test or preview
- Key configuration
- Project structure (lightweight, top-level paths)

If `README.md` already exists, preserve project-specific details and only add missing sections.

### Step 6: Create/repair Cursor context router
Ensure `.cursor/rules/context-router.md` exists with:

```markdown
For this repository, read `CLAUDE.md` before any substantive work to load project conventions and workflow constraints.

Then read `docs/WORKLOG.md` for the current task list, latest shipped work, and blockers.

Treat `CLAUDE.md` and context notes as supplemental project context alongside system/developer instructions and `.cursor/rules`. If guidance conflicts, prioritize system/developer instructions first, then `.cursor/rules`, then `CLAUDE.md` and context files.
```

### Step 7: Report completion checklist (required)

For the **simple** branch, return:

```markdown
Project init checklist (simple project):
- [x] Git repo + `.gitignore` + MIT `LICENSE` (created|skipped-no-git|already good)
- [x] `README.md` (created|updated|already good)
- [x] Minimal `CLAUDE.md` (created|updated|already good)
- [x] `.cursor/rules/context-router.md` (created|already good)
- [x] WORKLOG.md, docs/: not applicable at this scope

Manual follow-up:
- <only include items that still need user action>
```

If git was initialized (either scope) and no remote exists, always append this to Manual follow-up. Substitute the real values before printing (folder name from the project root, username via `gh api user -q .login`), the command must be copy-pasteable exactly as shown, no `<placeholders>`:

```markdown
- This folder is committed and ready to push whenever you want it on GitHub:
  - Fastest (creates the repo and pushes in one step, paste as-is): `gh repo create ACTUAL-FOLDER-NAME --private --source=. --push` (use `--public` to share it)
  - Or create the repo on github.com first, then connect it:
    ```bash
    git remote add origin git@github.com:ACTUAL-USERNAME/ACTUAL-FOLDER-NAME.git
    git branch -M main
    git push -u origin main
    ```
```

For the **in-depth** branch, return:

```markdown
Project init checklist:
- [x] Git repo + `.gitignore` + MIT `LICENSE` (created|skipped-no-git|already good)
- [x] `CLAUDE.md` (created|updated|already good)
- [x] `docs/WORKLOG.md` (created|updated|already good)
- [x] `README.md` baseline sections (created|updated|already good)
- [x] `.cursor/rules/context-router.md` (created|updated|already good)

Manual follow-up:
- <only include items that still need user action>
```

## ESCALATION PATH (when a project outgrows the defaults)
Not part of day-0 init. Apply later, during handoff or when the user asks:
1. When the "What this is" and "Key decisions" sections outgrow CLAUDE.md (roughly a screen of content), move them to a single `docs/PROJECT.md` (spec section + decision log section) and leave a pointer line in CLAUDE.md.
2. That's the last tier, do not split PROJECT.md into separate SPEC/DECISIONS files.
3. The task checklist always stays in `docs/WORKLOG.md`, never split it into a separate ROADMAP.md.

## DEFAULTS AND GUARDRAILS
- Never guess scope, always ask (Step 1). Folder contents are not a reliable signal for how deep a project will go.
- Both scopes ask the git question once; a yes covers `.gitignore` and MIT `LICENSE` automatically, a no skips all three. Never create a remote at scaffold time.
- Never delete existing project content to satisfy this skill; patch minimally.
- Never rewrite a full `README.md` if only sections are missing.
- Keep file paths in output explicit and copy-pasteable.
- This skill only creates `CLAUDE.md`, not `AGENTS.md`. If you already use `AGENTS.md` from another tool, `handoff` will still recognize and update it, project-init just doesn't create one itself.

## Tool-specific notes

**Cursor:** `context-router.md` points Cursor at `CLAUDE.md` and `docs/WORKLOG.md`. If you maintain a separate global-context file shared across projects, link or reference it from your own Cursor setup; that's outside this skill's scope.

**Claude Code:** if you keep global instructions in `~/.claude/CLAUDE.md`, that's a one-time, machine-level setup, not per-project. This skill does not configure it.

## BOUNDARY
- `project-init` is for one-time bootstrap or repairing missing scaffolding.
- `handoff` is for recurring session updates (session notes, Now/Next checklist, README health check, durable doc deltas) and owns the CLAUDE.md-to-PROJECT.md escalation.
- Do not move recurring handoff logging into `project-init`.
