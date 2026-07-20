# Decision log and worklog maintenance

Loaded only when one of these rare conditions actually fires during Step 4 (worklog) or Step 6 (spec/decisions). The common path (add one worklog entry, no corrections, log under 10/15 entries) doesn't need this file.

## Worklog archiving (Step 4)

If adding the new entry would push the session list past 10 entries, archive just enough of the oldest entries to `docs/archive/worklog-YYYY-MM.md` to keep the list at or under 10 after adding the new one, typically just the single oldest entry. Do this before writing so the list never exceeds 10.

**Ordering:** `docs/archive/worklog-YYYY-MM.md` files use newest-at-top, matching the live worklog. Insert newly archived entries at the top of the archive file, in the same newest-to-oldest order they appeared in the live worklog.

## Correcting an existing decision entry (Step 6)

When a past decision turns out to be wrong or incomplete, don't append a second "(Correction, ...)" onto an entry that already has one. Instead, mark the original entry superseded (e.g. `Superseded by [YYYY-MM-DD entry above].`) and write a fresh entry stating the current understanding as one coherent fact. This keeps each entry scannable and means the outdated one archives out immediately under the rule below, rather than accumulating corrections indefinitely.

## Decision log archiving (Step 6)

If the decision log already has 15 entries, archive the oldest to `docs/archive/decisions-YYYY.md` (year of the archived entries) before adding a new one, mirroring the worklog's archive-at-10 rule. Also archive any entry already marked superseded, regardless of count, once a newer decision replaces it. The live log should never exceed 15 non-superseded entries. Include archive moves in the proposed-changes list for confirmation, not as a silent follow-up.

**Ordering:** `docs/archive/decisions-YYYY.md` files use newest-at-top, matching the live decision log. Insert newly archived entries at the top of the archive file, in the same newest-to-oldest order they appeared in the live log.

## Escalation: CLAUDE.md outgrowing its spec/decision sections (Step 6)

If the "What this is" and "Key decisions" sections in `CLAUDE.md` exceed roughly a screen of content, propose moving them to `docs/PROJECT.md` (spec section + decision log section), leaving a one-line pointer in `CLAUDE.md`. Apply only with user confirmation, and note the move in the completion checklist.
