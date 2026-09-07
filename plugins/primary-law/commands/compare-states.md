---
description: Compare how different states word the same rule, starting from one provision or a topic.
argument-hint: "<citation or act_id, or a topic and a list of states>"
---

Compare across states: $ARGUMENTS

Two routes. Pick by what you were given.

**Given one provision** (a citation or `act_id`):

1. Resolve it to an `act_id` if needed (`resolve_statute_citation`, 2 credits).
2. Call `get_section_cross_state` (6 credits). It returns at most one provision per state, most similar first.
3. Fetch the body of each hit worth reporting.

**Given a topic and a list of states:**

1. Call `search_us_statutes` once with `state` as a **list**, for example `["ca","ny","tx"]`. One call, not one per state.
2. Take the best hit per state, then fetch bodies.

Either way, verify before you compare. `similarity` is a retrieval score, not a legal opinion: a high score means two provisions cover the same ground, never that they impose the same obligation, and the differences are usually the entire point of the question.

Report as a table, one row per state: citation, the operative requirement in a short phrase, and `goodLawStatus`.

Then, below the table, write the part that matters:

- **Where they actually differ.** Thresholds, deadlines, who is covered, what triggers the duty, what the remedy is.
- **Any state that did not come back**, and which of the two reasons applies. Fewer states than requested means the rest did not clear the similarity floor, not that they are missing from the corpus. Check `list_statutes_coverage` (free) before saying a state has no such law.
- **Any state whose `goodLawStatus` is not `good_law`**, called out rather than folded into the table.

Do not present the comparison as legal advice about which state is stricter. Present what each text says and let the differences speak.
