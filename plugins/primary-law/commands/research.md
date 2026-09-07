---
description: Find the controlling US law on a question and answer only from the text the tools return.
argument-hint: "<question> [in <state>]"
---

Research: $ARGUMENTS

If no jurisdiction was named, ask which one before spending credits. An unscoped search costs the same and returns worse results.

Run this loop:

1. **Confirm coverage** with `list_statutes_coverage` (free) if the question is about a state or a corpus you have not queried this session. This is what separates "the law does not say that" from "we do not hold that corpus".
2. **Search scoped.** Pass `corpusType` and `state`. Set `excludeRepealed: true` unless the user is specifically asking what was repealed.
3. **Judge the result set by score separation, not by whether rows came back.** Vector search always returns nearest neighbours, so a meaningless query still produces a full page. If the top score is not clearly above the tail, say the corpus does not appear to answer this rather than reporting the best of a bad set.
4. **Fetch the text** of the sections that actually matter with `get_us_statute_section_text`. Use `format=content` on USC so you get the operative text instead of the amendment notes. Never quote the excerpt.
5. **Check the defined terms** with `get_section_definitions` if the answer turns on a word like "person", "employee", "consumer" or "security". Statutes routinely define these against ordinary usage.
6. **Read `goodLawStatus` on everything you cite.**

Then answer. Structure it as:

- The controlling provision, quoted from the body, with its citation and official source link.
- What it requires, in plain language.
- Any defined term that changes the reading.
- Currency: `good_law`, or an explicit caveat.

Rules for the answer:

- Every proposition traces to text a tool returned. If you know something the tools did not return, say it is outside what the corpus confirmed.
- `goodLawStatus: unknown` means unverified, not current. Tell the user to confirm against the official source.
- There is no case law here. If the question actually turns on how courts have read the provision, say so plainly rather than implying the statutory text settles it.
- If nothing solid came back, say so. Do not assemble an answer from adjacent sections.
