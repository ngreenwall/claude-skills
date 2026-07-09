---
name: drift-check
description: >-
  Manual context-drift audit. Confirms your global context file (the standing
  instructions loaded on every session) is still loaded and being followed,
  and flags where context dropped. Run ONLY when the user explicitly types
  /drift-check or says "drift check," "context check," "are you still
  following my context," or "check for drift." Do NOT auto-trigger and do NOT
  run during normal task work.
---
# SKILL: Drift Check

A deliberate, manual audit that confirms your **global** context file (the standing instructions your AI tool loads on every session, e.g. `~/.claude/CLAUDE.md`) is loaded and actually shaping behavior, and pinpoints where context dropped (top, middle, or bottom of the file). Long sessions and context compaction silently drop rules; this catches it.

This skill targets the global file specifically because that's usually the long, dense one, personal preferences, workflow habits, formatting rules, accumulated over time. That length is exactly what makes mid-file and end-of-file drift possible. A project-level `CLAUDE.md` is typically short (file-structure or workflow reminders) and gets re-read naturally each session, it doesn't carry the same depth-loss risk, so it's out of scope for this skill by default.

If you ever notice yourself violating a project-specific rule mid-session (e.g. editing a file you were told not to touch), that's a signal you might want a lightweight check for that project file too, but don't set one up preemptively, only add it once you've actually seen it happen.

This skill audits CLAUDE.md specifically, your own written instructions. It's a separate system from Claude Code's auto memory (`~/.claude/projects/<project>/memory/MEMORY.md`), notes Claude writes about itself as it works. Auto memory is out of scope here, there's nothing to audit for drift since Claude, not you, decides what goes in it.

Do NOT pre-read the context file before running the checks. The whole point is to test what is actually loaded in the current context window. Reproduce from memory, then you may verify against the file in step 1.

## Setup (one-time, before first use)

This skill needs three canary tokens embedded in your global context file:

1. Generate three short, unique, unguessable strings (e.g. `anchor-word-word-NN` style, or any random phrase you won't naturally type in conversation). Use a different token for each position, don't reuse one token in all three spots.
2. Place one token near the **top** of the file (alongside your most important always-on rule, so recalling the rule and the token happen together), one in the **middle**, and one at the **bottom**. Label each clearly, e.g. `Mid-file canary: <token>` and `Canary (bottom): <token>`, and instruct the file's reader not to output the token during normal work, only when running this check.
3. Note the file path so Check 1 knows what to re-read. Only needed if your global context file lives somewhere other than the standard location (e.g. `~/.claude/CLAUDE.md`).

Skip setup if tokens already exist in the file from a previous run.

If you use an always-on convention worth checking every time (e.g. a response-format prefix), name it in your context file, then replace the placeholder examples in this file's own Check 2 "Core rules" list (below) with your actual rules, so scorecards stay comparable across runs.

If you don't have a global context file yet and want one, create it yourself, this skill won't create it for you, that's a bigger, machine-level decision than an audit tool should make on your behalf.

**Using this in Cursor instead of Claude Code:** the canary/labeling mechanics are the same regardless of tool, only the file changes. Cursor's equivalent of a global context file is its User Rules (configured in Cursor Settings), place the same three labeled canaries there instead of `~/.claude/CLAUDE.md`. You can reuse `drift-check/example-global-CLAUDE.md`'s content as a starting point, just adapt it to whatever format Cursor's rules expect. Note: step 3 (noting the file path) doesn't apply here, User Rules aren't a plain file with a path, so Check 1's re-read may need to happen by you pasting the current contents back in rather than the skill reading it directly.

## PROTOCOL

Produce a short scorecard with all three checks below. Keep it tight.

### Check 1, Proof of load + depth

Reproduce the three depth canaries from the context file **from memory**, then confirm each:

- **Top:** the token near your top-level always-on rule.
- **Mid-file:** the token labeled as the mid-file canary.
- **Bottom:** the token labeled as the bottom canary.

For each, mark ✅ if you can reproduce it verbatim, ❌ if you cannot. A missing token tells you which depth dropped:
- All three ✅ = file loaded end to end.
- Top ✅, mid/bottom ❌ = middle/end dropped (classic long-context loss).
- All ❌ = file not loaded at all.

After scoring from memory, read the context file and confirm the tokens match. Flag any mismatch.

### Check 2, Behavior audit

Grade the last several assistant replies in this session against the context file's rules. One row per rule, ✅/❌, with a one-line verbatim evidence quote from a recent reply. If a drift-check already ran this session, grade only the replies since it; earlier violations were already reported and corrected.

This check is self-assessment, so trust it less than Check 1. The same drift that corrupts output can corrupt this grading (a drifted model may rate itself ✅). Check 1 (canary recall) is the objective anchor. If Check 1 fails, distrust a clean Check 2, the model likely cannot grade itself reliably either.

Audit in two passes:

**Core rules (always, fixed list, keeps scorecards comparable across sessions):** pick 3-5 rules from your context file that rarely change and are easy to verify from a single reply, for example a response-format prefix, a formatting constraint, or a "confirm before editing files" policy. Write them into this list once during setup and keep the list stable across runs so scorecards stay comparable over time. Example placeholders to replace with your own:
- Follows the required response-format rule (if any)
- Follows the required formatting constraints (if any)
- States proposed changes and waits for confirmation before editing files (if that's your policy)

**Derived rules (keeps the audit current as your context file changes):** after the Check 1 file read, grade every other testable rule in the file that isn't covered by the core list above. Derive these from the file you just read, not from memory, the fresh read is the ground truth. Skip rules that didn't apply this session (e.g. a recommendation-first rule when nothing was recommended) and mark them `n/a` rather than ✅.

For any ❌, quote the offending line so the drift is concrete.

### Check 3, Compaction flag

State whether context was compacted this session (a `/compact` ran, or a context-summary/handoff appears earlier in the thread). Compaction is a top drift trigger, but it doesn't hit every check the same way:
- A project-root `CLAUDE.md` (or `CLAUDE.local.md`) is re-read from disk and re-injected automatically after compaction, so a Check 1 ❌ on that file after compaction is a real finding, not expected fallout, the content should have survived.
- A `CLAUDE.md` in a subdirectory is not auto-reinjected, it only reloads the next time Claude reads a file in that subdirectory, so a ❌ there after compaction can be expected fallout until that happens.
- Check 2 (behavior) ❌s are always expected fallout after compaction regardless of file scope, past replies aren't retroactively fixed by a reload, only future ones are.
- If unsure which of the above applies: say so rather than guessing.

## Output format

Lead with the verdict, then three lines. Only expand a line into per-item detail when that check found a failure, a clean check stays a single summary line.

**All clean:**
```markdown
Drift check: ✅ on-track
- Canaries: top ✅ / mid ✅ / bottom ✅ (loaded end to end)
- Behavior: <N>/<N> rules followed
- Compaction: <yes | no | unsure>
```

**Drift found (expand only the failing check(s)):**
```markdown
Drift check: ⚠ drifting
- Canaries: top ✅ / mid ❌ / bottom ❌ (middle/end dropped)
- Behavior: <N>/<total> rules followed
  - <rule>: ❌  "<evidence quote>"
  - ... (one line per failing rule only; skip ✅ and n/a rows entirely)
- Compaction: <yes | no | unsure>
```

`<total>` for the Behavior line is the count of rules actually graded (core + derived, excluding `n/a`). Compaction only expands past one line if it's flagging fallout, e.g. "Compaction: yes, treat any ❌ above as expected fallout."

If drift is found, end with one plain-language sentence naming who acts and when, no shorthand tags, no jargon:
- Behavior fix (Check 2): say "I'll self-correct this starting with my next reply, you don't need to do anything."
- Load/file fix (Check 1): say "This needs your OK before I edit `<file>` to fix it."
- Compaction fallout: say "Re-run this check whenever it's convenient, no rush."

If this check is confirming a fix from an earlier drift-check in the same session, say so in full sentences, e.g. "Last check flagged <rule>; every reply since then followed it, so that's resolved, nothing else needed from you." Don't reuse a tag or shorthand from the earlier check's output out of context, it won't carry meaning on its own.

No other wrap-up.

## Auto-remediation (on drift only)

Remediation depends on which check failed. They fail differently: a load failure is fixable now by re-reading; a behavior failure lives in past replies and can only be corrected going forward.

**Check 1 ❌ (load/depth):** attempt one self-heal pass before stopping:
1. **Re-read the context file** (the path noted during setup). This is a read, not an edit, so do it without asking.
2. **Re-run all three checks once** and print a second, shorter scorecard.
3. **Report the result:**
   - Resolved → say so in one line.
   - Still ❌ → state which check still fails and give the manual fix.

**Check 2 ❌ only (behavior, Check 1 clean):** do NOT re-run the checks, past replies cannot be regraded. Instead:
1. State each broken rule and the correction in one line each.
2. Apply the corrections from the next reply onward.
3. The retest is the next `/drift-check` run, recommend one later in the session if the drift was more than one rule.

Hard limits:
- Re-read only. Do NOT edit any file to "fix" drift; that needs confirmation.
- One retry max. Do not loop.
- If a canary fails to load even after re-reading (file-not-loaded case), stop and tell the user to check the path/filename, you cannot fix that yourself.
- If drift recurs across sessions, flag it: the file may be too long, and the real fix is shortening it, not re-reading. Anthropic's own guidance targets under 200 lines per CLAUDE.md file, longer files consume more context and measurably reduce adherence.
