---
description: Find federal agency guidance on a topic, and say clearly what weight it does and does not carry.
argument-hint: "<topic> [agency or source]"
---

Find agency guidance on: $ARGUMENTS

Use `corpusType: "AGENCY_GUIDANCE"`. This is **sub-regulatory** material: what an agency says about how it reads the law. Handling that honestly is the whole job of this command.

## Narrow with `source`, always

The token folds 51 named sources across 26 agencies. Unscoped, a query for harassment guidance can return NLRB advice memos.

The `source` vocabulary is published on the search endpoint, and **a bad value returns a 422 naming every valid code**, so the API is the authoritative list. Ask it rather than guessing. `list_statutes_coverage` (free) also carries the full legend with per-source scope notes.

## Say what it is before you say what it says

Guidance is not law, and several sources carry harder limits than that:

- **IRS written determinations** (private letter rulings, TAM, chief counsel advice) carry `actStatus: non_precedential` because **26 U.S.C. § 6110(k)(3) says they "may not be used or cited as precedent"**. They are evidence of how the Service analysed one taxpayer's facts. Say so every time you surface one.
- **The Internal Revenue Manual** is the IRS's own instruction manual for its employees. It is agency procedure, it does not bind the agency against a taxpayer, and it must never be listed alongside statutes and regulations as though it were.
- **The DOL Field Operations Handbook** is WHD internal enforcement procedure, on the same footing.
- **CMS Internet-Only Manuals** are instructions to Medicare contractors, not regulations.

## Coverage is bounded per source, and the bounds are real

Several sources are deliberately partial. Check the scope note before reporting an absence:

- NLRB Division of Advice memoranda: a **rolling 10-year window**, not the full archive.
- CPSC advisory opinions: historical, roughly 1970s to 2012.
- OLC opinions: the official bound volumes, ending at volume 44 (2020). Opinions issued since are not bound and are not held.
- CFTC staff letters: 2008 to present.

## Currency

Most of this corpus reports `goodLawStatus: unknown`, and that is correct rather than a gap: agencies rarely publish a machine-readable withdrawal signal. Where an item carries `rescinded`, `superseded` or `withdrawn`, lead with it.

`unknown` here means unverified. Do not present a 2011 advisory opinion as the agency's current position because nothing marked it dead.

## Report

For each item: the issuing agency and source, the citation or document number, the date, and what it says.

Then, in one line, what it is worth: binding regulation, non-binding interpretation, internal procedure, or non-precedential determination. That sentence is the most useful thing in the answer.

If the question is really about what the law requires rather than how an agency reads it, say so and point at the statute or regulation instead.
