# Worklog

## Now / Next
<!-- Checklist. "Now" items first (1-3 max), then queued items. -->
- [ ]

### Ideas
<!-- Unscoped, someday. -->

## Session notes
<!-- Newest entry at top. Format: YYYY-MM-DD | Shipped: ... | Next: ... | Blockers: ... -->
2026-07-22 | Shipped: Ported private my-ai-agent edits into handoff/SKILL.md, worklog-template.md, decision-log-maintenance.md: archive thresholds 10→6/15→8; length checks now hard-enforced; README relevance + CLAUDE.md bloat checks re-added; /compact-at-low-effort line added. token-audit/SKILL.md: 2 new triggers; new Mode C (WebFetch Anthropic docs, propose checklist updates); bare-invocation menu step. README.md: project-init structure diagrams (simple/in-depth); Works-in-Cursor-too tags on project-init/handoff. | Next: none pending from this session. | Blockers: none

2026-07-20 | Shipped: completed the `~/my-ai-agent` token-audit comparison (commit c05d345): `handoff` gained a closing `/clear` line, trimmed Step 5, and new `handoff/references/decision-log-maintenance.md` (rare-path archiving/correction/escalation rules, newest-at-top ordering); `drift-check` gained a ~40-line output guardrail, condensed intro, table-format remediation mapping; `qa-audit` gained a shorter "This is NOT" pointer; `project-init`'s duplicate `ESCALATION PATH` merged into `BOUNDARY`. Also fixed a pre-existing bug: `handoff` Step 6's decision-ordering rule contradicted `CLAUDE.md`'s real newest-at-top log. | Next: none pending from this session.

2026-07-20 | Shipped: ran a full-library token-audit pass (8 Opus subagents) across all 5 skills, CLAUDE.md, and README.md; applied 3 safe cuts (qa-audit's frontmatter Do-NOT clauses collapsed; token-audit's Step 4 gained a `/compact` suggestion for library mode, matching qa-audit's existing pattern; project-init's `CLAUDE.local.md` aside trimmed from ~60 to ~20 words) plus 1 confirmed-uncertain fix (handoff's duplicate "Shipped" guidance at top-of-file folded into Step 5's own wording). Declined a proposed README/CLAUDE.md dedup after verifying the flagged duplicate didn't actually exist in README.md (false positive from the cross-file audit agent). | Next: review ~/my-ai-agent's own token-audit pass (commit c05d345) against this repo's skills for anything worth porting, in progress when this session ended.

2026-07-13 | Shipped: Added a "When to run this" section to `handoff/SKILL.md`, clarifying it's not just end-of-task: also the right move at a safe checkpoint when context is running high mid-task (using the Next: field for remaining work), preferred over `/compact` since it's a reviewed capture rather than an automatic lossy summary. Ported from the equivalent change made to the private `~/my-ai-agent` handoff skill. README's existing "Use it" line already covered the long-session case, left as-is. | Next: none pending from this session.

2026-07-09 | Shipped: added `token-audit` as fifth public skill (rewritten from private, README/CLAUDE.md updated); ported private token-optimization pass into `handoff`/`project-init`: inline templates moved to `assets/*.md` files loaded on demand (worklog starter, CLAUDE.md starters, Cursor context-router content), kept this repo's override-marking sentence that private had dropped; added a human-readability guardrail to `token-audit`; merged `drift-check`'s duplicate output-format blocks into one; fixed `qa-audit`'s `/compact` wording; added a skill-folder-structure diagram to README's intro. Commits 4c0d757, 2489043. | Next: none pending from this session.

2026-07-09 | Shipped: qa-audit scoped Claude Code only: dropped Cowork claims from README (TOC, Cowork section, heading) and CLAUDE.md's decision entry; fixed README's `drift-check`/`qa-audit` headings, moved their `(Claude Code only)` suffix out of the heading into the blockquote subtitle so GitHub's auto-generated anchor slug matches the TOC link (was broken); `qa-audit/SKILL.md` Step 3 now batches 2-3 small SKILL.md files (<100 lines) per Opus agent call on 4+-file audits to cut token spend, larger/higher-risk files (global context file, CLAUDE.md, >100 lines) stay 1:1, mirrors the same change already made to the private skill. | Next: none pending from this session.

<!-- Archive to docs/archive/worklog-YYYY-MM.md once the list reaches 6 entries -->
