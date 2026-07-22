---
name: token-audit
description: >-
  Audits instruction/config files (SKILL.md, CLAUDE.md, your global context
  file, slash command files) and this repo's skill architecture for token
  bloat, then proposes fixes: rewrite, split, merge, add navigation, or
  remove. Use when the user says "audit tokens," "check token bloat," "shrink
  this file," "audit the skills library," "is this skill too big," "cut
  context cost," "check for updated Anthropic guidance," or "refresh the
  checklist." Do NOT use for finding bugs, contradictions, or dead logic
  (use qa-audit for that). Do NOT use for code files, only instruction/config
  prose and skill structure.
---
# SKILL: Token Audit

A token-bloat audit over instruction and config prose and this repo's skill architecture. Recognized targets: `SKILL.md` files, `CLAUDE.md`, your global context file (the standing instructions your AI tool loads every session, e.g. `~/.claude/CLAUDE.md`), `README.md`, slash command files (`.claude/commands/*.md`), and other instruction/config prose files.

This is NOT:
- **qa-audit**, that reports bug-shaped findings to fix. This skill proposes token-reduction actions (rewrite, split, merge, add nav, remove) instead, a different output shape.
- **drift-check**, that verifies global context actually loaded and is being followed in a *live session*. This skill is a static audit, session state doesn't matter.
- A code minifier. This skill never touches code, only instruction/config prose and skill structure.
- A fix for `docs/WORKLOG.md` or `docs/PROJECT.md` bloat. Those are logs governed by `handoff`'s own per-entry length caps and archive thresholds, not this checklist.

## Modes

**Mode A - single file (default).** One file given, run the single-file checklist only.

**Mode B - full library.** Explicitly requested (e.g. "audit the whole skills library"). Runs the single-file checklist across every file, plus the cross-file checklist and the unused-skills question. This is the expensive path, only run when asked.

**Mode C - refresh guidance.** Explicitly requested (e.g. "check for updated Anthropic guidance," "refresh the checklist"). Requires WebFetch; if it's unavailable in this environment (e.g. blocked by org policy), tell the user and stop rather than proceeding without it. WebFetch these Anthropic doc pages: `https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview` and `https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents`. Compare their current guidance against the Checklist section below, then propose specific additions/edits/removals to the Checklist for confirmation, same before/after format as Step 6. This mode only edits this skill file, never a target file. Skip prompt-caching guidance, already out of scope (see "Dropped, not checked" below).

If the request doesn't make the mode clear, ask before proceeding.

## Checklist

Numbers below match the single-file prompt template's numbering exactly, so a subagent's numbered findings sort directly into these tiers with no renumbering step.

**High weight** (recurring cost paid every session or every trigger, flag and lead the report with these):
1. Length/bloat, single-file: CLAUDE.md-type files over ~200 lines, or sections not needed every session.
2. Monolithic structure, single-file: content that should split into a `reference/` file loaded on demand instead of always-loaded.
3. Hardcoded examples/data instead of scripts, single-file: prose Claude has to read and reason through where a script would produce the same output directly.
4. Verbose reference docs with no navigation aids, single-file: no headers/internal links, forcing full-file reads.
- Duplicate content across files, cross-file/library-mode only: the same instruction living in global CLAUDE.md, project CLAUDE.md, and/or a skill. (Cross-file template item 1, unnumbered here since it never runs in single-file mode.)

**Medium weight, single-file:**
5. Vague or overlapping skill description, causes wrong or redundant triggers.
6. Missing "when NOT to use" guidance, no negative triggers despite being narrow or heavy.
7. Instructions that could be scoped: workflow-specific content sitting in a broadly-loaded file instead of a scoped skill. Single-file mode flags candidates; library mode's cross-file template re-checks the same thing across files, only report it once in Step 4, not once per agent.
8. No conciseness instruction where one would cut response tokens with little downside.
9. Long conversation history left unmanaged, no prompting toward `/compact`/`/clear` at natural breakpoints.
10. Full files/logs pasted instead of relevant excerpts.
11. Thinking budget too high for simple tasks, no call to lower effort for lightweight steps.
12. Vague prompting patterns that send Claude exploring broadly instead of pointing at specific files/paths.

**Library-mode only, interactive, not a subagent check, not numbered in either template:**
Unused or rarely-triggered skills: no usage telemetry exists in this repo, so ask the user directly which skills they've actually used rather than inferring. Unconfirmed skills are removal candidates, not auto-removed.

**Dropped, not checked:** prompt caching (harness-managed, not file content to edit) and redundant MCP server tool descriptions (third-party server content, not editable here).

**Guardrail on every check above:** a fix that saves tokens but reads worse for a human, not just one that loses an exact string, still counts as a quality cost. Flag it "uncertain" the same as a nuance-loss risk, don't propose it as a plain "safe" fix.

## PROTOCOL

### Step 1: Resolve target(s) and mode, guard against code

Identify the file(s) and mode (see Modes above).

**Bare invocation (no file named, mode unclear):** check the current directory for anything to audit first, don't default to asking an open-ended question.
- If `SKILL.md` files, `CLAUDE.md`, or a global context file exist here: present a 3-option menu (single file / full library / refresh guidance) instead of free-text back-and-forth.
- If none of those exist: there's nothing for Mode A or B to point at. Skip the menu, tell the user in one line ("No instruction/config files found here, offering a guidance refresh instead"), and go straight to Mode C.

Before anything else, check whether the target looks like code: it's a code file if the extension isn't `.md`/`.mdc`. If so, stop immediately, don't spawn any agent, and tell the user in one line: "That's a code file, not instruction/config prose, use /code-review for that."

If the extension is `.md`/`.mdc` but the path isn't one of the recognized instruction/config locations (a `SKILL.md` inside a skills folder, `CLAUDE.md`, your global context file, `README.md`, `.claude/commands/*.md`), ask the user to confirm it's instruction/config prose before proceeding.

If the file is managed by a sync tool that generates copies elsewhere (e.g. a tool that regenerates `.claude/` or `.cursor/` files from a single source of truth), always resolve to that source file, never a generated copy. Skip this check if you don't use such a tool, most projects edit `SKILL.md`/`CLAUDE.md` directly.

**Skip check (Mode A only):** if the target is under ~100 lines and a quick read shows no obvious bloat (headers/nav present, no repeated content, no wall-of-prose section), skip spawning the audit agent and report "already lean, no findings" instead of running the full checklist. Mode B always runs the full pass per file, this shortcut is single-file only.

### Step 2: Gather context

Read related docs (README, decision log, sibling skills) so the subagent doesn't flag deliberate, documented choices (repeated canary tokens, intentional emphasis, required verbatim command blocks) as bloat.

### Step 3: Spawn audit agents

Mode A: one `Agent` call per target file, `model: "opus"`, launched in parallel. If auditing 4+ files, batch small files (`SKILL.md` under ~100 lines) 2-3 per call; keep larger/higher-risk files (your global context file, `CLAUDE.md`, anything over ~100 lines) on their own call.

Mode B: run the same per-file calls as Mode A, plus one additional `Agent` call (`model: "opus"`) given the full set of file paths/contents for the cross-file checklist.

**Single-file prompt template (fill in placeholders per file):**
```
You're auditing [file] in [repo/location] for token bloat. Purpose: [what it's supposed to do].

Goal: flag ways to reduce the token cost of loading this file, and propose a fix for each. This is not a bug hunt (that's qa-audit); don't flag contradictions or dead logic.

Check for:
1. Length/bloat: is this file (if CLAUDE.md-type) over ~200 lines, or does it carry sections not needed every session?
2. Monolithic structure: is there content that should live in a reference/ file loaded on demand instead of always-loaded?
3. Hardcoded examples/data instead of scripts: prose Claude has to read and reason through where a script would produce the same output directly.
4. Verbose reference content with no navigation aids: long sections with no headers/internal links forcing full reads.
5. Vague or overlapping description (if this file has skill frontmatter): would this trigger on requests it shouldn't, or fail to trigger when it should?
6. Missing "when NOT to use" guidance (if this file has skill frontmatter): no negative triggers despite being narrow or heavy.
7. Scoping: instructions here that look workflow-specific but live in a broadly-loaded file instead of a scoped skill.
8. No conciseness instruction where one would cut response tokens with little downside.
9. No guidance toward /compact or /clear at natural breakpoints, if this is a long-running workflow skill.
10. Full-file/log-paste patterns: instructions that cause a whole doc or log to load when only a section is needed.
11. No effort/thinking-budget guidance for lightweight steps in an otherwise heavy workflow.
12. Vague prompting patterns: instructions that send Claude exploring broadly instead of pointing at specific files/paths.

Never touch: verbatim strings that must stay exact (commands, code blocks, canary tokens, file paths, frontmatter field names, exact trigger phrases). If a cut risks losing a nuance you're not fully sure is safe to drop, flag it "uncertain" instead of proposing a fix. Same for a cut that saves tokens but makes the file harder for a human to read or scan quickly, even if no exact string is lost, that's a quality cost too, not a free win. Flag it "uncertain" rather than "safe."

Read [related docs] first so deliberate, documented choices aren't flagged as bloat.

Output one finding per issue: category number, location, quoted text (if applicable), proposed fix (rewrite / split into reference file / add nav headers / add negative trigger / add conciseness instruction / etc.), estimated impact, confidence (safe / uncertain-flag-for-review). End with a summary count grouped by high/medium weight.
```

**Cross-file prompt template (Mode B only):**
```
You're auditing token bloat ACROSS these files: [list of paths]. This is a whole-library pass, not a single-file check.

Check for:
1. Duplicate content: the same instruction/rule appearing in more than one file (e.g. global CLAUDE.md, a project CLAUDE.md, and/or a skill). For each duplicate, name all locations and recommend one source of truth.
2. Scoping: instructions sitting in a broadly-loaded file (your global context file, root CLAUDE.md) that are actually specific to one workflow/skill and could move into that skill instead. Only flag this if it requires comparing multiple files to see, the single-file pass already flags same-file scoping candidates on its own.

Output one finding per issue: category, all locations involved, recommended consolidation/move, confidence. End with a summary count.
```

### Step 4: Compile findings

Merge across all agent outputs into one list. Sort high-weight items (template numbers 1-4, plus cross-file duplicate content) first, then medium-weight, each sorted by estimated impact within its tier. If both the single-file and cross-file agents flagged the same scoping issue (item 7), report it once, keep the cross-file version since it names all locations involved.

On a full-library run (Mode B), suggest `/compact` here before Step 5, the raw agent transcripts are no longer needed once findings are compiled.

**Report format cap:** one line per finding (issue + proposed fix), no restated subagent reasoning. On Mode B, show the top 15 findings by impact in full and summarize the rest as a count by tier (e.g. "12 more medium-weight findings, same categories").

### Step 5: Library-mode, ask about unused skills

List every skill in your skills directory and every slash command under `.claude/commands/` (name + one-line description) and ask the user directly which they've used. Anything not confirmed goes into the report as a removal candidate, listed separately from the subagent findings, not applied automatically.

### Step 6: Propose fixes, wait for confirmation

Show original vs. proposed for each finding, with uncertain-flagged items called out separately. Structural actions (split a file, remove a skill, merge duplicates) get explicit confirmation each, since they're more invasive than a text rewrite. Wait for explicit confirmation before editing anything.

### Step 7: Apply

On confirmation, apply using the current session's running model, including splitting into `reference/` files or removing a skill folder where confirmed.

### Step 8: Regenerate if sync-managed

If any edited file is managed by a sync tool that generates copies elsewhere (see Step 1), rerun that tool's generate/sync command so the copies pick up the change. Skip this step entirely if no file is sync-managed, that's the common case.

### Step 9: Commit

Single-contributor repo convention: commit (and push, if this repo is push-direct-to-main). For a shared or multi-contributor repo, propose a branch and PR instead.
