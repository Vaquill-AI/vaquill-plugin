---
description: Sweep a corpus or jurisdiction for what has changed since a date.
argument-hint: "<YYYY-MM-DD> [corpus and/or state]"
---

Report what changed since: $ARGUMENTS

Use `search_us_statutes` with `changedSince`.

1. **Scope it before you call.** Pass `corpusType` and `state`. A `changedSince` window matching more than 20,000 sections is **refused with a 422**, not truncated, because a partial change set would look complete to a sync job. If you get that 422, narrow the corpus or the date range; do not retry unchanged.
2. Run the search. It costs 4 credits, and **an empty result is charged**: "nothing moved since Tuesday" is the answer you paid for.
3. For anything that matters, call `get_section_changes` on the `act_id` (1 credit) to see the event history, then fetch the body if the user needs the current text.

Report as a list grouped by corpus: citation, what kind of change, when it was detected.

State these three caveats explicitly. They are the difference between a useful report and a misleading one:

- **These are observed changes, not effective dates.** `detectedAt` is when we saw the publisher's text differ from our copy. It is an upper bound on when the change took effect, never the effective date.
- **An empty result means no captured change in the window, never that nothing was amended.** Change capture began long after the corpus did, runs per source, and events are swept at 24 months. A date range older than 24 months will look quiet because the events were swept, not because the law was still.
- **A detected change is not necessarily an amendment.** Publishers re-typeset and republish. If the user needs to know whether the substance moved, the diff has to be read.

If they ask "has anything changed" about one specific section rather than a corpus, skip all of this: `get_section_changes` on that `act_id` costs 1 credit and answers it directly.
