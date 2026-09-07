---
description: Report what Vaquill AI actually holds for a jurisdiction, before you read an empty result as an absence of law.
argument-hint: "[state, or 'federal', or leave blank for everything]"
---

Report corpus coverage for: $ARGUMENTS

Call `list_statutes_coverage`. It is free, so call it rather than inferring coverage from search results.

Read three things off the response, and keep them separate:

1. **`corpora`** - which corpus types this jurisdiction holds, and the document count for each. Coverage is per `(jurisdiction, corpusType)` pair, never per jurisdiction: a state can be complete in statutes and hold nothing in regulations.
2. **`freshness`** - notices naming a pair that is no longer refreshed. `paused` means the data is complete and queryable but no longer updated. `retired` means that corpus is not offered there and its count is 0.
3. **`retrievalStatus`** - `current`, `stale`, `failing` or `never_retrieved`. This records when we last **checked**, not when the law last changed, and it is not a claim that the law is current.

Then answer the question the user actually has, which is almost always one of these:

- **"Is this a gap or an absence?"** An empty search inside a corpus we hold means the provision is probably not there. An empty search for a corpus we do not hold means nothing at all about the law and must never be reported as "no such provision".
- **"How current is this?"** Freshness covers **statutes only**. Regulations, court rules, constitutions and agency guidance each refresh on their own schedule and are not represented there, so an absent freshness notice is not a freshness guarantee for those corpora.

Two things to state rather than assume:

- **Read `measuredAt`.** This response is cached and can legitimately be hours old. If the user is making a decision on freshness, tell them when it was measured.
- **A section count is not a statute count.** The per-jurisdiction total spans every corpus type, so a jurisdiction can hold tens of thousands of documents and zero statutes.

If the user landed here because a search came back empty, say explicitly which of the two cases they are in. That is the whole reason this command exists.
