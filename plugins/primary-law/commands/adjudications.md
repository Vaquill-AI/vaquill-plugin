---
description: Search federal administrative adjudications, with the overruling caveat stated up front.
argument-hint: "<topic or citation> [body]"
---

Search adjudications for: $ARGUMENTS

Use `corpusType: "AGENCY_ADJUDICATION"`. These are decided cases from executive-branch tribunals.

## Read this before reporting anything

**No publisher in this family ships a machine-readable overruling signal for the general case.** A decision served as issued may have been overruled the week after it was decided, and nothing in this API will tell you.

`goodLawStatus` is `unknown` on every body except the CFPB slice, and that is correct rather than a gap. **Never present one of these as settled law.** Present it as a decision that was issued, with its date, and say the corpus cannot confirm its current standing.

This is also not case law. There is no court-opinion corpus here, and judicial review of these bodies is not published by them, so it is not reflected either.

## The bodies, and what differs about each

Filter with `source`.

| Body | What to know |
| --- | --- |
| **MSPB** | `mspb_precedential` binds Board administrative judges; `mspb_nonprecedential` does not. Separable on the `precedential` boolean. Federal Circuit review under 5 U.S.C. 7703 is not published by the MSPB |
| **OLC** | The executive branch's own controlling legal advice, cited `NN Op. O.L.C. NNN`. The bound series **ends at volume 44 (2020)**; opinions since are unbound and not held. Every point says so in its own `coverageNote` |
| **CFPB** | The only body with a real status signal. Actions the Bureau marks `Expired/Terminated/Dismissed` carry a dead `actStatus` and are **excluded from ordinary search**, which is roughly half the set. Both forums are held and separable on `forum` |
| **SEC** | Commission opinions and adjudicatory orders, cited by release number. **ALJ initial decisions are deliberately not held**; where an opinion names one, the citation is in `relatedCitations` only |
| **BIA / AG** | The `I&N Dec.` reporter. A published Board precedent binds every immigration judge and DHS officer nationwide under 8 C.F.R. 1003.1(g); an Attorney General decision sits above it |

## The BIA is the one place with real doctrinal links

The Board publishes holdings prose stating its effect on earlier decisions, resolved into `supersedes` / `supersededBy` with the publisher's own verb in `dispositionVerb`.

**`overruled_in_part` does not carry a dead `actStatus`**, deliberately: a partly overruled decision is still law for everything else it held. Report the partial overruling rather than treating the decision as dead or as intact.

Three stated limits: the Board only began publishing that prose at volume 19, so a pre-1984 overruling of a pre-1984 decision is invisible; a court of appeals can reject a precedent for its own circuit while it stands elsewhere, and EOIR does not publish that; volumes 1 to 7 are not held.

## SEC proceedings are sequences, not verdicts

Documents sharing a `fileNumber` are one proceeding, linked in date order through `supersedes` / `supersededBy`. Those links record a **sequence**, not an outcome: a later document may affirm the earlier one, schedule briefing, or merely notice an initial decision as final. Do not read a `supersededBy` link here as "overturned".

## Report

For each decision: the body, the citation as the publisher prints it, the date, `decidedBy` where present, and the holding.

Then the caveat, every time: this is the decision as issued, and the corpus carries no verdict on whether it still stands. Tell the user to shepardise before relying on it.
