---
description: Trace the chain between a statute and the regulations that implement it, in either direction.
argument-hint: "<citation or act_id>"
---

Trace the authority chain for: $ARGUMENTS

Statutes grant power; regulations exercise it. This command walks the link, which is a different question from "what does this say".

1. Resolve the input to an `act_id` if it is a citation string.
2. Fetch the section and read the link fields.

## The fields, and which direction each points

| Field | On | Points to |
| --- | --- | --- |
| `implementingRegulations` | USC | The CFR parts issued under this statute |
| `statutoryAuthority` | CFR | The statute granting the rulemaking power |
| `lawImplemented` | CFR and some state regs | The statute the rule actually **implements** |

**`statutoryAuthority` and `lawImplemented` are not the same thing**, and conflating them is the mistake this command exists to prevent. One is the power to write the rule, the other is the substantive law being carried out. Florida populates both and they routinely differ.

## Then widen with the citation graph

- `crossReferencesUsc` and `crossReferencesCfr` are the outbound references in the text. `'self:<section>'` denotes another section of the same title.
- `relatedCitations` carries outbound references that are neither USC nor CFR. On several corpora `crossReferencesUsc`/`Cfr` are 0% populated, so this is the only outbound set they have.
- `get_section_cited_by` (2 credits) is the **inbound** direction: which sections point at this one. USC and CFR only; state codes and Federal Register rules carry no cross-reference index and return empty with a `note`, refunded.

## Two honesty constraints

**Inbound citations are evidence, not proof.** A citation the publisher never recorded in machine-readable form cannot appear. `total` is the size of the returned list capped by `limit`, not a corpus-wide count; `truncated` says more exist.

**A chain is not an argument.** That a regulation was issued under a statute says nothing about whether it is valid, current, or applies to the user's facts. Report the links, not a conclusion about authority.

## For federal statutes, consider the enacting text too

`publicLawCites` on a USC section carries structured Statutes at Large citations, each with `statVolume`, `statPage` and a `display` string. Those resolve into `corpusType: "SESSION_LAW"`, which holds federal law **as enacted** rather than as codified.

Worth fetching when the question turns on what Congress actually passed rather than how it was compiled. Note that acts before 1957 carry no public law number and are cited by chapter.

## Report

Draw the chain as a short list, statute at the top, regulations beneath, with the field that established each link named. Where a link is absent, say the corpus records none rather than implying no such relationship exists.
