# Model alignment audit checklist

Checks 1-7 are structural (Anthropic's Agent Skills best-practices doc, re-fetch each run, this is a baseline not the live source of truth). Checks 8-13 are logic-fit (synthesized from Anthropic's degrees-of-freedom framework plus known current-model characteristics: more agentic, configurable reasoning effort, less need for exhaustive guardrails than older models). Full explanations of each check live in the subagent prompt template below, since that's where they're actually needed.

## Subagent prompt template

```
You are auditing a Claude Code instruction/config file for structural compliance with Anthropic's current Agent Skills guidance AND for whether its workflow logic still fits how current-generation Claude models (Sonnet 5, Opus 5) actually behave.

FILE PATH: {path}
FILE CONTENT:
{content}

CURRENT ANTHROPIC GUIDANCE (fetched this session):
{guidance_summary}

Check the file against these two categories. Cite the specific line/section for each finding.

STRUCTURAL (numbered 1-7):
1. Naming: kebab-case, gerund or noun-phrase form, no reserved words (`anthropic`, `claude`), no vague names (`helper`, `utils`).
2. Description: third person only, states both what the skill/file does and when to use it, specific trigger terms not vague ones, includes negative triggers if narrow or heavy.
3. Frontmatter valid for the file type (skill: `name` + `description`; command: `description` only, no `name`).
4. Size: SKILL.md body under 500 lines; CLAUDE.md/global context file under ~200 lines (this repo's own bloat-check threshold).
5. Progressive disclosure: reference/asset files linked one level deep only from SKILL.md, no nested chains; reference files over 100 lines have a `## Contents` block.
6. No time-sensitive content baked into instructions ("before/after [date]"); deprecated patterns go in a clearly marked old-patterns section instead.
7. Consistent terminology throughout the file.

LOGIC FIT (numbered 8-13):
8. Freedom level matches task fragility: low freedom (exact scripts/wording) only for fragile, sequential, consistency-critical steps; high freedom (heuristics) for judgment calls. Flag low-freedom scaffolding on a task that doesn't need it.
9. Unexplained numeric caps or thresholds ("voodoo constants" in prose form) with no stated rationale. Don't propose a new number, propose either a stated rationale or the flag-if-premature note pattern.
10. Over-explaining: content a current-generation model already knows without being told (generic definitions, obvious reasoning steps). Flag for trimming.
11. Under-specification: a fragile or high-stakes step left to model judgment with no guardrail, script, or validation loop where one would prevent a real failure mode.
12. Reasoning-effort awareness: mechanical/deterministic steps that could call for lower effort but don't; missing where the skill has an obvious mechanical tail (bookkeeping, formatting, checklist steps after the substantive work is done).
13. Any instruction that assumes an older model behavior Anthropic's current docs no longer describe (a deprecated pattern, a workaround for a capability current models now have natively).

For each finding: state the category number, quote the offending text, explain the specific failure mode (not a vague "could be improved"), and propose a concrete fix. For logic-fit findings 8-13 that are guesses about model behavior rather than a clear guidance violation, mark them LOW CONFIDENCE and propose the flag-if-premature note pattern instead of a direct rewrite.

Do not flag a finding if the file's own content (a comment, a decision-log entry you're given, an adjacent rule) explains it's a deliberate choice.

Output: a numbered list of findings, empty list if none, each finding one line.
```
