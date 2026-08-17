---
name: model-alignment-audit
description: >-
  Audits skills (SKILL.md files), your global context file, and
  CLAUDE.md/AGENTS.md files against Anthropic's current Agent Skills guidance
  and the behavior of current models (e.g. Sonnet 5, Opus 5). Use when the
  user says "audit for new models," "audit my skills against latest
  Anthropic guidance," "are my skills still built right for current models,"
  "model alignment audit," or "refresh skills for the new model." Do NOT use
  for pure token-bloat concerns or to refresh token-audit's own checklist
  (use token-audit). Do NOT use for logic contradictions/dead code unrelated
  to model fit (use qa-audit).
---
# SKILL: Model Alignment Audit

Checks whether skills and context files are still well-built for the models actually running them today, not just structurally valid. Two questions per file: does it follow Anthropic's current authoring guidance, and does its workflow logic (freedom levels, caps, hand-holding) still match what current models need, or is it carrying scaffolding tuned for older, less capable models.

This is NOT:
- **token-audit**, that targets token-bloat cost (rewrite/split/merge). This skill targets guidance-and-model fit instead.
- **qa-audit**, that finds contradictions and dead logic unrelated to model behavior. Overlap is possible, run both if both concerns apply.

## Requires web access

This skill fetches live Anthropic documentation. If WebFetch/WebSearch are unavailable in this environment (e.g. blocked by org policy), tell the user and stop rather than proceeding without current guidance.

## PROTOCOL

### Step 1: Resolve targets

Default (no file named): audit everything in scope.
- If run from a skills-library repo (a repo whose primary content is `SKILL.md` files): every `SKILL.md` in it, your global context file (if accessible from here), and this repo's `CLAUDE.md`.
- If run from any other project: that project's `CLAUDE.md` (or `AGENTS.md`) plus any project-local skill files.

If the user names a specific file or skill, audit just that one instead. If the named file isn't a `SKILL.md`, `CLAUDE.md`/`AGENTS.md`, or a rules file, ask before proceeding.

If the file is managed by a sync tool that generates copies elsewhere (e.g. a tool that regenerates `.claude/` or `.cursor/` files from a single source of truth), always resolve to that source file, never a generated copy. Skip this check if you don't use such a tool, most projects edit `SKILL.md`/`CLAUDE.md` directly.

### Step 2: Fetch current guidance

WebFetch `https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices` for the structural checklist. WebSearch for recent Sonnet/Opus release notes or changelogs, at most two fetches, summarize in under 200 words, to catch model-specific behavior changes relevant to freedom-level tuning, e.g. configurable reasoning effort, agentic capability jumps. Prefer anthropic.com/platform.claude.com sources over third-party summaries. If a fetch fails or returns nothing new, say so and fall back to `references/checklist.md` as-is rather than substituting search results. Summarize what's new since the checklist was last confirmed current, don't re-paste the full fetched pages into context.

### Step 3: Gather context

Check your project's decision log (if one exists) for entries about the target files, so a subagent doesn't flag a deliberate, documented choice (an exact-wording requirement, a canary token, a tuned cap) as a defect.

### Step 4: Spawn audit agents

One `Agent` call per target file, `model: "opus"`, launched in parallel. If auditing 4+ files at once, batch small files (under ~100 lines) 2-3 per call; keep larger or higher-risk files (your global context file, `CLAUDE.md`) on their own call. These batching thresholds are tuning guesses, not measured limits, flag it if batching produces a noticeably worse result than 1:1 would have.

Fill in `references/checklist.md`'s prompt template with: the file's content, this session's summary of current Anthropic guidance from Step 2, and the two check categories (structural, logic/model-fit).

### Step 5: Compile findings

Merge into one list, split into two sections: **Structural** (format/frontmatter/naming/progressive-disclosure issues) and **Logic fit** (freedom-level mismatches, unexplained caps, over- or under-specification). Rank each section by confidence, an unvalidated guess about model behavior is lower confidence than a clear guidance violation, label accordingly. One line per finding: issue plus proposed fix.

For any logic-fit finding that's a guess rather than an observed problem (e.g. "this cap might be too tight for Sonnet 5"), don't propose changing the number outright. Propose adding a short in-skill note asking the model to flag it if the cap fires prematurely in real use, the same flag-if-premature note pattern used elsewhere in this repo (e.g. `handoff`'s length checks). Only propose changing a number directly when there's a stated reason (e.g. it contradicts a concrete Anthropic recommendation).

On a full-library run, suggest `/compact` before Step 6, the raw agent transcripts aren't needed for the apply phase.

### Step 6: Propose fixes, wait for confirmation

Show original vs. proposed per finding. Wait for explicit confirmation before editing anything.

### Step 7: Apply

Apply confirmed fixes using the current session's running model. Steps 7-9 are mechanical, run them at low reasoning effort. If a proposed fix turns out to need real judgment mid-apply, stop and raise it rather than pushing through at low effort.

### Step 8: Regenerate if sync-managed

If any edited file is managed by a sync tool that generates copies elsewhere (see Step 1), rerun that tool's generate/sync command so the copies pick up the change. Skip this step entirely if no file is sync-managed, that's the common case.

### Step 9: Commit

Single-contributor repo convention: commit (and push, if this repo is push-direct-to-main). For a shared or multi-contributor repo, propose a branch and PR instead.

## BOUNDARY

- Use this for periodic re-checks after a new Claude model ships, or when the user asks whether skills/context still fit current model behavior. Not a substitute for `token-audit` (bloat) or `qa-audit` (contradictions/dead logic); run those separately if that's the actual concern.
- Don't run this reflexively on every skill edit, it's a guidance-refresh audit, not a pre-commit check.
