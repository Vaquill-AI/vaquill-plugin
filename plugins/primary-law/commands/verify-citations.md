---
description: Verify every legal citation in a file, selection, or pasted text against the official source text.
argument-hint: "[file path, or paste the text]"
---

Verify the citations in: $ARGUMENTS

If no argument was given, use the current selection or ask which file to check.

Procedure:

1. Extract every statutory, regulatory and court-rule citation from the text. Ignore case citations; this corpus does not carry general case law, and say so if the text leans on them.
2. Resolve them in one call with `resolve_statute_citations_batch` rather than one at a time. It is priced per citation either way, so the batch is a pure round-trip win.
3. For each citation that resolved, fetch the section metadata and check `goodLawStatus`.
4. For any citation whose quoted language matters to the argument, fetch the body and compare the quotation against the official text verbatim.

Report as a table: citation as written, what it resolved to, whether it is still good law, and whether any quoted language matches.

Flag these separately and prominently:

- Citations that did not resolve. Say "did not resolve" rather than "does not exist"; an unresolved citation may be a formatting variant.
- Citations pointing at repealed or superseded provisions.
- Quotations that differ from the official text, with both versions shown.

Do not silently correct a citation. Show what was written and what the source says, and let the user decide.
