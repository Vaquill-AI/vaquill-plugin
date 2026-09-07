---
description: "Track a Federal Register rulemaking: what an agency proposed, what became final, and when comments close."
argument-hint: "<topic or agency> [proposed|final] [since YYYY-MM-DD]"
---

Track rulemaking: $ARGUMENTS

This is a different question from "what does the law say". The CFR is the rule in force; the Federal Register is the rule *moving*. Use `corpusType: "FEDERAL_REGISTER"`.

## Filters that only work here

- `agency` takes the FR agency slug, e.g. `environmental-protection-agency`. A list matches any of several. Combined with a non-FR corpus it returns **empty**, not an error.
- `documentType` is `final`, `proposed` or `presidential`. Ask which the user means: "what did EPA do" is usually final, "what is coming" is proposed.
- `publishedFrom` / `publishedTo` are the **publication** date. This is not `yearFrom`, which filters a section's version year.

Each result carries `agencySlugs`, and those are exactly the values `agency` accepts, so a facet round-trips straight back into a narrower query.

## Report the dates carefully, because there are four of them

| Field | Meaning |
| --- | --- |
| `publicationDate` | When it appeared in the Register |
| `frEffectiveOn` | When the rule takes effect |
| `frCommentsCloseOn` | The comment deadline on a proposed rule |
| `datesText` | The publisher's own prose, ~98% populated |

**Prefer `datesText` when a rule has staged or conditional effective dates.** The single `frEffectiveOn` flattens that away, and a compliance reader acting on the flattened date can be wrong about when an obligation starts.

A rule is routinely published well before it is effective. Never report `publicationDate` as when the rule took effect.

## Follow the rulemaking, not just the document

A single rulemaking is usually several documents: a proposed rule, its final rule, and any corrections. They are joined by the **Regulation Identifier Number**.

`frRelatedDocuments` groups them, but it is **only populated on `get_us_statute_section`**, never on a search result, because computing it needs a follow-up lookup. So: search to find one document, then fetch it by `act_id` to see the rest of its family.

Also useful: `frRegulationIdNumbers` (the RIN itself), `frDocketIds`, `frCorrectionOf` and `frCorrections`, `frRegulationsDotGovUrl` for the comment docket, and `frStartPage`/`frEndPage`, which give the page range the citation cannot express.

## Notices are a curated slice, not the whole series

`FEDERAL_REGISTER_NOTICE` is a separate token holding roughly 2.4% of notices: the ones that act on a rulemaking (withdrawals, comment-period extensions and reopenings, significant guidance, negotiated rulemaking). Every notice carries `frNoticeClass` saying which rule admitted it.

Paperwork Reduction Act collections, meeting notices and permit applications are **not held**. If the user is looking for one of those, say so rather than reporting an empty result as an absence.

## Report

Group by rulemaking, not by document. For each: agency, stage, citation, publication date, the effective or comment-close date, and the `abstract` (populated on 99.9% of rules) rather than your own summary. Note `action` verbatim, since "interim final rule", "direct final rule" and "withdrawal" are legally different things.
