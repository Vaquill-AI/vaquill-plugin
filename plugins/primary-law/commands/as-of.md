---
description: Read a statute section as it stood on a given date, with the observation caveat stated honestly.
argument-hint: "<citation or act_id> <YYYY-MM-DD>"
---

Reconstruct a section as of a date: $ARGUMENTS

Procedure:

1. If given a citation string rather than an `act_id`, resolve it first with `resolve_statute_citation`. Never assemble an `act_id` by hand; the chapter component is not derivable from a citation.
2. Call `get_section_changes` (1 credit) to see the observed timeline for that section.
3. Call `get_us_statute_section_text` with `asOf=<date>`.

Report the text, then the provenance, in this order:

- The reconstructed body.
- The change events that bracket the requested date, with their dates.
- The `asOf.isBounded` flag, translated into plain language.

Translate `isBounded` carefully, because it is the part users misread:

- `isBounded: true` means we observed no change affecting that date. It is evidence, not proof.
- `isBounded: false` means the reconstruction is not bounded by an observation at all, and should be presented as our best reconstruction rather than as the text in force.

Never phrase either case as "this is what the law said on that date" without the qualifier.
If the section has no recorded change events, say that the reconstruction rests on no observation history.
