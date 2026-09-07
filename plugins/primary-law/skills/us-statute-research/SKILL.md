---
name: us-statute-research
description: Research US primary law with the Vaquill AI tools - find a controlling section, resolve a citation to an exact provision, read a section as it stood on a past date, scope a search to the right corpus, and report currency honestly. Use whenever a question turns on what a US statute, regulation, court rule, constitution or agency guidance document actually says, or when a citation needs verifying against the official text.
---

# Researching US primary law

This corpus is enacted and promulgated text from official government publishers.
Every answer must rest on text a tool returned.

A plausible fabricated citation is the worst output this tool set can produce, and it is the easiest to produce by accident: **a nonsense query still returns rows.** Vector search always returns nearest neighbours. Separation is the signal, not presence. A meaningless query topped out at `relevanceScore` 0.47 where a real one scored 0.95.

Read this before the first call. Most of what follows is the difference between an answer and a confidently wrong answer.

---

## 1. The corpus

### Do not hardcode what it holds

The published docs say it plainly: do not hardcode credit costs, jurisdiction counts, or corpus lists.
Section counts differ between the search documentation, the coverage endpoint and the marketing page, and all three go stale.

`list_statutes_coverage` is **free** and is the only live answer. It returns a `corpusTypes` legend, per-jurisdiction counts, freshness notices and retrieval status.
Call it rather than quoting a number from memory or from this file.

### The corpus tokens

Roughly 19 `corpusType` tokens. The live spec enum and the docs disagree on the exact set, so treat this as orientation and the coverage legend as truth.

**Federal:** `USC`, `SESSION_LAW` (Statutes at Large, as enacted), `STATUTE_COMPILATION`, `CFR`, `CFR_ANNUAL`, `FEDERAL_REGISTER`, `FEDERAL_REGISTER_NOTICE`, `EXECUTIVE_ACTION`, `AGENCY_GUIDANCE`, `AGENCY_ADJUDICATION`, `US_TAX_TREATY`, `SENTENCING_GUIDELINES`, `FEDERAL_RULES`, `CONSTITUTION`.

**State:** `STATE`, `REGULATION`, `STATE_RULES`, `STATE_AGENCY_GUIDANCE`, `STATE_CONSTITUTION`.

A federal token needs only `corpusType`. **A state token needs `state`**, or the search spans every jurisdiction holding that corpus and costs exactly the same.

`source` narrows the folded tokens (`AGENCY_GUIDANCE` alone covers dozens of named sources across many agencies). A bad `source` returns a 422 that names every valid code, **so the API itself is the authoritative list.** Ask it rather than guessing.

### `CFR_ANNUAL` is excluded from the default pool

Historical CFR editions are not merely demoted, they are excluded from the default candidate pool. An unfiltered search never returns them.
To reach a historical edition you must ask for it by token.

### What it does not hold

**No general case law, no court dockets.** The roadmap puts them explicitly out of scope. There is no judicial-opinion search here.
If a question needs holdings rather than enacted text, say so instead of reaching for a statute tool that cannot answer it.

**United States only.** A question about another country's law returns empty rather than erroring, which reads as a gap in US coverage. Say the corpus does not cover it.

---

## 2. `act_id`: the single most common failure

> "You never construct an `actId` by hand."

It looks assemblable. It is not.

```text
USC_T42_C21_S1983                       42 U.S.C. § 1983
CFR_T17_P240_S240_10b5_1                17 C.F.R. § 240.10b5-1
STATE_TX_Cpr_C93_S93.005                Tex. Property Code § 93.005
STATE_LA_Crevised-statutes_T14_S112.11  La. R.S. § 14:112.11
```

The title and section are derivable from a citation. **The chapter is not.**
In the Texas example the `Cpr` segment exists only in the data. Ids are case-sensitive and mixed-case.

Assembling ids happens to work for a few jurisdictions whose hierarchy sits in the citation, which is exactly what makes it dangerous: the failures look like missing coverage rather than wrong ids.

| You have | Call | Cost |
| --- | --- | --- |
| A citation string | `resolve_statute_citation` | 2 |
| Many citations | `resolve_statute_citations_batch` | 2 each |
| A topic | `search_us_statutes` | 4 |
| A place in the hierarchy | `list_statute_divisions` | 1 |

### The batch endpoint tries to diagnose a bad id, but do not lean on it

`get_sections_batch` (up to 50 ids) returns `notFound` plus `notFoundDetail` with a `reason`:

- **`assembled_id`** - the id does not exist but the section does, under the ids in `didYouMean` (at most 3).
- **`not_in_corpus`** - malformed beyond recovery, or genuinely absent.

That distinction is the point: read as "not in corpus", an assembled id looks like missing coverage, which is how a working integration gets abandoned.

**Measured 2026-09-07, it is unreliable in both directions:**

```text
USC_T42_S1983          -> not_in_corpus, didYouMean []
                          USC_T42_C21_S1983 exists. This IS an assembled id
                          and was not diagnosed as one.

STATE_TX_C93_S93.005   -> assembled_id,
                          didYouMean [..._Cag_..., ..._Cbc_..., ..._Cfi_...]
                          The correct code is Cpr. It is not in the list.
```

So `not_in_corpus` does **not** mean the section is absent, and a `didYouMean` list does **not** necessarily contain the right id. The spec says it plainly and means it: treat it as a hint rather than a contract.

**Never resolve a bad id by picking from `didYouMean`.** Go back to `resolve_statute_citation` or `search_us_statutes`, which return an id confirmed to exist. Misses are free, so the only cost of guessing here is a wrong answer.

---

## 3. Cost and billing

Credits are $0.01 each. **This table is a snapshot; `get_pricing` is free, unauthenticated and live.**

| Cost | Operations |
| --- | --- |
| Free | `list_statutes_coverage`, `get_pricing` |
| 1 | `list_statute_divisions` (refunded on an empty level), `get_section_changes` |
| 2 | `get_us_statute_section`, `resolve_statute_citation`, `get_section_neighbors`, `get_section_cited_by`, `get_sections_batch` per section **returned** |
| 4 | `search_us_statutes`, `get_section_definitions` |
| 6 | `get_us_statute_section_text`, `get_section_cross_state` |

### Read `creditsConsumed`, never compute it

> "Read it rather than assuming the list price: failed and refunded work bills 0, and batch endpoints charge per item returned."

A batch of 50 ids resolving 47 costs 94 credits, not 100.

### Charged on a negative answer, deliberately

Not bugs. A confident negative is the answer you were paying for.

- An unresolved citation on `resolve`, single or batch. Only backend errors refund.
- A search returning zero results, including an empty `changedSince` window. "Nothing moved since Tuesday" is the answer.
- An empty but real change history for a section that exists.
- A `changes` response for a `removed` section. Learning that a provision was repealed is the point.
- A point-in-time request where the section demonstrably did not exist yet.
- `cited_by` returning `[]` with **no** `note`, meaning genuinely nothing cites it.

### Refunded

404s everywhere; `body` when `available: false`; `asOf` with `source: "unavailable"`; out-of-scope corpora on `cited_by` / `definitions` / `cross_state`; definitions that could not be parsed; an empty `divisions` browse; batch misses; `includeBody` rows that returned `body: null`; every validation rejection; **and any backend outage**.

### A 200 with no results is a real answer, not an outage

This is load-bearing. When the corpus is unreachable you get a **503 and a refund**, never `200 results: []`.

Measured against production on 2026-08-15 while the corpus was down, the API *used to* return `200 results: []` and charge 4 credits. That is fixed. So today, an empty 200 means the corpus genuinely has nothing matching, and you should report it as such rather than suggesting a retry.

---

## 4. Search

Hybrid semantic plus keyword ranking. There is deliberately no keyword-only mode.

### Scope it, always

> "Unscoped queries can surface keyword matches from unrelated corpora; a corpus filter sharply improves precision."

- `state` takes a 2-letter code or `federal`, and a **list** works: `["ca","ny","tx"]` is one call instead of three.
- `code` is the only way to scope below a jurisdiction. `state=tx` alone searches all ~15 Texas codes. Values are the `identifier` from `list_statute_divisions`, fed straight back.
- `chapter` and `part` **must** be paired with `titleNumber` (USC/CFR) or `code` (state). Unpaired is rejected, because chapter numbers repeat across titles.
- `titleNumber` is only valid with `USC` or `CFR`. Without a corpus scope it is rejected, because an unscoped numeric title mixes state statutes with USC rows.
- Every result carries a `parent` object. **Feed it back verbatim** to search that hit's neighbours; it always carries one of the required pairings.

### Caps

| Limit | Value |
| --- | --- |
| `limit` | max 50, default 10 |
| `offset` | max 70 |
| **Deepest reachable result** | **#120** |
| `query` | 2 to 500 chars |
| Batch `actIds` / `citations` | 50 |

Do not page past `offset` 70. Narrow the query instead.

`count` is the size of the current page. **`total` is a deprecated alias for `count`** whose name reads as a corpus-wide total, which it never was. Read `count` in new code.
There is no corpus-wide match total: ranking runs over a bounded 120-candidate pool. `total == count` with `hasMore: true` is correct.

### `relevanceScore`

Roughly 0 to 1, a **relative rank within one response**. Never compare across queries, never use a fixed threshold.
Only an exact citation match returns 1.0. It is **null** on endpoints that did not rank (fetch by id, batch fetch): null means "not ranked", not "bad match".

Judge a result set by the *separation* between top and tail, not by whether rows came back.

### `fields` projection

A result carries 40+ fields, most null on any row. `fields` trims the payload; `actId` and `citation` are always included.
An unknown field name is **rejected with 422** so a typo does not silently drop a field you needed.

### `includeBody` is a cost multiplier

The 4-credit search **plus the full 6-credit body price for every row that returns text**.

- 10 rows with text: 4 + 60 = **64 credits**
- `limit: 50`: **304 credits in a single call**

Rows with `body: null` are not charged, so the real cost is never computable from `limit`.
It buys latency, not a discount. Running out of credits for the bodies does **not** void the search: rows serve with `body: null` and you are charged only for the search.

### The excerpt is not quotable

> "`excerpt` is a ranking preview windowed around the match, so it can begin mid-section and drop a leading subsection marker."

Measured 2026-09-02: four of five California rows were exactly 4 characters short, and the missing 4 were the leading `(a) `. `body.startswith(excerpt)` is **false**.
For statutory text that is a misquote, not a formatting nit.

**Never quote `excerpt` to a user.** Use it to choose which section to fetch, then fetch the body. Raising `excerptChars` does not help; a bigger excerpt is still an excerpt.

### Currency filters are not point-in-time

- `yearFrom` / `yearTo` filter the **most recent** amendment year. A section amended in 2025 is excluded by `yearTo=2024` even though it existed in 2024.
- Sections carrying no date at all are **silently excluded** the moment either bound is set, so an unbounded search returns strictly more.
- The corpus holds one current text per citation. For a past text use `asOf`.

### `changedSince` is the sync filter

> "These are observed changes, not effective dates. The date is when we SAW it, an upper bound on when it took effect."
> "An empty result means no captured change in the window, never that nothing was amended."

Events are swept at **24 months**. A window matching more than 20,000 sections is **refused, not truncated**, because a partial change set would look complete to a sync job. Narrow with `corpusType` and `state`.

---

## 5. Reading a section

### Metadata is not text

`get_us_statute_section` (2) returns citation, hierarchy, breadcrumb, status and format links.
`get_us_statute_section_text` (6) returns the body.
Do not pay 6 to answer "does this exist" or "what is it called".

### On USC, the body is mostly not the law

The highest-impact fact in this skill.

> "The operative text is a median 46% of it across USC, and as little as 3.4% on `17 U.S.C. § 107`, where 28,529 of 29,775 characters are commentary."

| `format` | Returns |
| --- | --- |
| `both` | html + plain (default; the same text twice) |
| `plain` / `html` | one of them; roughly halves the payload |
| `content` | **operative text only**, with `sourceCredit` and `notes` split out |

**If you are feeding sections to a model, use `content`.** The full body invites quoting a 1992 amendment note as the law in force. On `17 U.S.C. § 107` that is ~1 KB instead of ~30 KB, same price.

**`content` is USC only.** CFR and state corpora carry no equivalent structure, so all three split fields are null there and `plain` remains the whole body.
Null means "we cannot split this reliably", never "this section is empty". Check for null and fall back to `plain`.

### `structured: true`

Adds `markdown` and a `subsections` tree at no extra cost. Nodes carry `label` (`(b)`), `pincite` (`(b)(2)`), a full pinpoint `citation`, and `text` **excluding nested children**.
Use it when the user needs a specific subsection.

### Which URL to cite

- `sourceUrl` on the body, and `externalUrl` on a result, are the government's published address. **Cite these.**
- `source` names our storage backend. It is **not a citation**.
- `stateHtmlUrl` is the publisher's page for most state corpora, but for mirrored corpora it is our archived copy. Prefer `externalUrl`.
- **A null source URL is often correct, not missing data.** Some state regulations and court rules are published through commercial platforms where the licence covers sourcing but not republishing the link, so a `licenseNote` says a link exists and is withheld. We never substitute a third-party compiler.

---

## 6. Point-in-time (`asOf`)

`get_us_statute_section_text` takes `asOf=YYYY-MM-DD`, same 6 credits. There is no versions endpoint; `get_section_changes` is the timeline.

> "It is a reconstruction, not an archive. The corpus holds one current text per citation; an earlier one is rebuilt from the before-side of the first change we OBSERVED after your date."

`html` is **never** returned on this path. Synthesising markup the publisher never printed would be a wrong answer on the endpoint whose job is fidelity.

| Field | Meaning |
| --- | --- |
| `source: live` | No change captured after your date, so today's text is what stood then **as far as we ever saw** |
| `source: reconstructed` | Rebuilt from the before-side of a captured change |
| `source: unavailable` | We know it changed and cannot rebuild it. No text. **Refunded** |
| `existed: false` | Earliest capture is its ADDITION, postdating your date. A real answer. **Charged** |
| `observedFrom` | Earliest change ever captured. A property of when capture began, not of the age of the law |
| `basisChangeId` | The change the reconstruction rests on. Null when `source` is `live` |
| `coverage` | Prose, written to be shown to a reader verbatim |

### `isBounded` is load-bearing

> "TRUE means: we observed no change affecting your date, and that is NOT the same as there having been none. Surface it. Rendering a bounded answer as authoritative point-in-time law makes a claim we did not make."

Never say "this is what the law said on that date" for a bounded answer. Say what we can attest to, and show the `coverage` prose.

### Published-edition corpora are enumerations

`CFR_ANNUAL` and some compendia hold discrete published editions. `editionsObserved` **is an enumeration, never a range**: an edition list of 1996 and 2011 says nothing about 2004.

---

## 7. Currency: the fields that look alike

Getting this wrong tells a lawyer that dead law is live.

### `year` is a trap

> "The year this row was ingested, not the year of the law. Most ingesters stamp the capture year, so 1,575,300 statute sections currently read 2026. Do not filter or reason about currency with it."

### `actStatus` is raw and filterable; `goodLawStatus` is derived and not

`actStatus` is the publisher's own word as recorded, stored and indexed.
`goodLawStatus` is computed at read time from status, jurisdiction, publisher and category. It is **not** filterable.

> "Filtering on the status we store and reporting the verdict we derive are deliberately separate, because only the first is indexable."

### The `goodLawStatus` vocabulary

| Value | Meaning |
| --- | --- |
| `good_law` | **Two claims**: the publisher marks it in force, AND we have verified this publisher actually prints repeals where we can read them, so an absent repeal marker is evidence |
| `not_good_law` | Checked, and dead |
| `not_operative` | Exists, no operative force right now (reserved, vacant, dormant, unfunded, not yet effective). It may become law later |
| `unknown` | **Unverified.** Not a soft yes and not an alarm |
| `null` | Currency checking is switched off server-side |

**`unknown` is a statement about our measurement, not about the law.** It conflates three genuinely different situations the API cannot currently tell apart:

1. A section its own legislature prints as in force, in a jurisdiction whose repeal signal nobody has verified. California statutes and a large share of state regulations sit here.
2. A status the currency module does not classify: `enacted` (session laws), `snapshot` (mirrors), `pending`, `non_precedential`.
3. A document that is not current law at all: a frozen enactment record, an internal procedure, an adjudication a later panel may have overruled.

> "Treat `unknown` as 'verify against the official source,' not as 'current.'"

`good_law` is the only affirmative currency claim. **Never infer currency from the absence of `not_good_law`.**

Be especially careful under `AGENCY_ADJUDICATION`: a decision served as issued may have been overruled the week after it was decided, and nothing in this API will tell you.
`SESSION_LAW` and `STATUTE_COMPILATION` carry `unknown` by design; they are records of enactment, not of current force.

### `excludeRepealed` does not promise good law

> "This removes what we KNOW is dead. It does not promise the remainder is good law. A section survives when its status is `in_force` OR when we hold no trustworthy repeal signal for its jurisdiction, and those two are not the same thing."

Filter with it, then **read `goodLawStatus` on every survivor**.

To find what was lost rather than what remains, use `actStatus: "repealed"`. Combining a dead `actStatus` with `excludeRepealed: true` contradicts itself and is rejected with 422.

Two filter asymmetries worth knowing:

- **`non_precedential` is filterable but never excludable.** Excluding it would drop tens of thousands of documents from an "exclude repealed law" query, which is not what that asks for.
- **`enacted`, `snapshot` and `pending` are rejected as filter values** even though they exist on hundreds of thousands of points, because "offering a filter for a status we cannot interpret would sell certainty we do not have."

Dead law is **demoted, not dropped**, from default search results. It will appear unless you exclude it.

### IRS written determinations

They carry `actStatus: "non_precedential"` and by statute "may not be used or cited as precedent" (26 U.S.C. § 6110(k)(3)). Say so if you surface one.

### Amendment counts do not match amendment years

> "`amendmentsCount` is not the length of `amendmentYears`, and it is not safe to derive one from the other. The two differ on 181,757 regulation sections and 15,648 court-rule sections."

`amendmentYears` is served newest-first and de-duplicated. `lastAmendedYear` is re-derived as its max.
`effectiveDate` and `originalEnactmentDate` are deliberately separate: merging them reports an amended section as effective from its original year.
`reviewDate` is a future review deadline, not an amendment date.

### Follow the pointer when a section moved

`supersededBy` carries the replacement's `actId`, so you can fetch current text in one hop. `renumberedTo` and `transferredTo` do the same.

---

## 8. Citation resolution

Accepts `42 U.S.C. 1983`, `16 C.F.R. 444.1`, `Del. Code Ann. tit. 13, 1301`.

**Charged whether or not it resolves**, by design: a confident "this does not resolve" is exactly the answer you want when checking a citation an LLM produced. Only server errors refund.

> "When it does not resolve, `resolved` is `false` and `section` is `null`: treat that as 'unverified', not 'current'."

Report it as **"did not resolve"**, never as "does not exist". Only the first is something you observed.

### Ambiguous acronyms resolve to false, correctly

`MCA`, `IC`, `G.S.`, `G.L.`, bare `R.S.`, and court-rule sets like `CR`, `RAP`, `TCR` are genuinely ambiguous. `resolved: false` is the right answer, not a coverage gap.

### `state` is a constraint, not a hint

A citation naming a different jurisdiction returns `resolved: false` rather than being forced into the one you passed.
`8 CCR 1206-2` is Colorado and `22 CCR 76227` is California; the acronym alone cannot say which.
For a batch spanning jurisdictions, omit `state` and let each citation name its own.

### `subsection` has two meanings

1. A pinpoint you cited resolves to the parent section and echoes the pinpoint back.
2. The citation named a unit **smaller than the publisher issues**. Rhode Island cites `250-RICR-120-05-15.2` but publishes the whole Part, so `section` is Part 15 and `subsection` is `15.2`.

> "A non-null `subsection` always means `section` is the containing document, not the exact unit you cited."

Do not quote a whole Part as if it were the cited subsection.

### Batch

Up to 50. Duplicates collapsed before pricing. **Every input gets an entry**, unresolved included, because a caller checking thirty citations needs to know *which* failed.

---

## 9. Section intelligence, and its scope limits

Each returns empty with a `note` and **refunds** when out of scope.
An empty result **with** a `note` is an explanation. **Without** one it is a real answer.

| Tool | Scope | Cost | Notes |
| --- | --- | --- | --- |
| `get_section_neighbors` | Any | 2 | `limit` 1-10 per side, default 3. Never crosses the containing chapter. Natural order: `9` before `10`, `1983` before `1983a` |
| `get_section_cited_by` | **USC and CFR only** | 2 | State codes and FR rules carry no cross-reference index |
| `get_section_definitions` | Strongest on USC; CFR and state best-effort | 4 | Refunded when terms cannot be parsed, but `definitionsSection` is still populated so you can read it yourself |
| `get_section_cross_state` | **State statutes only** | 6 | One provision per state, `limit` 1-25 |

### Definitions matter more than they look

> "Statutes do not use ordinary English. 'Person' routinely includes corporations, 'employee' routinely excludes independent contractors, and the section you are reading almost never says so."

Before interpreting a section that turns on a defined term, spend the 4 credits.

### `cited-by` is evidence, not proof

> "A citation that the source never recorded in machine-readable form cannot appear here, so treat a result as evidence of a citation rather than proof there are no others."

### `cross-state` similarity is not a legal opinion

> "A high score means the provisions cover the same ground, never that they impose the same obligation, and the differences are usually the point."

Fewer states than requested means the rest did not clear the similarity floor, not that they are missing.
Verify each hit's body before saying two states' rules match.

---

## 10. Change history

`get_section_changes`, 1 credit, `limit` up to 200.

- `detectedAt` is when **we saw it**, an upper bound on effect. A weekly source is seen up to a week late. Never present it as the date the law changed. For the publisher's own dates read `amendmentHistory` on the section.
- An empty history means **no captured change**, not never amended. Capture began long after the corpus, is per-source, and events are swept at 24 months. Surface the `coverage` prose rather than rendering an empty list as "unchanged".
- **A removed section still answers.** `section` comes back null on a successful, charged 200, not a 404.
- Absence of a `removed` event is not proof a section stands: a narrow, section-scoped refresh cannot observe a removal.
- Page with `sinceId` forward, `beforeId` backward. `id` is monotonic corpus-wide, immune to clock skew, and cannot drop two changes sharing a timestamp. **Never page on `detectedAt`.**
- `hasMore` means "this page filled to `limit`", not that more rows are known to exist.
- Use `displayCitation` for display; it always has something readable.

A 404 means no such section **and** no change ever captured for that id.

---

## 11. Rejected, ignored, or merely empty

**Rejected with 422** (the request was wrong, and saying so is the point):
any unknown key in the search body; unknown `state`, `source`, `fields` name, `corpusType` or `actStatus`; unpaired `chapter` or `part`; `titleNumber` without a USC/CFR scope; a dead `actStatus` with `excludeRepealed: true`; an impossible calendar date such as `2026-02-30`; `yearFrom > yearTo`; `limit` over 50; `offset` over 70; a batch over 50; a `changedSince` window over 20,000 sections.

The forbidden-extras rule exists because a customer sending a key that was not a filter "was charged full price for results that were never filtered, and nothing in the response said so."
**A 422 naming your key is the API working correctly.** Fix the key; do not retry without it.

Note one inconsistency: the single `resolve` returns **400**, not 422, for a bad scope.

**Silently ignored**: nothing important. Filters that cannot apply are rejected rather than dropped.

**Empty, not an error**: `agency` combined with a non-Federal-Register corpus.

**Branch on status, not on the `detail` string** (the wording is not part of the contract):
401 bad key, 402 out of credits, 403 missing scope or plan entitlement, 404 no such resource, 429 rate limited, 503 backend unavailable and refunded.

### Rate limits

| Plan | Per minute | Per hour | Per day |
| --- | --- | --- | --- |
| Pro | 30 | 500 | 1,000 |
| Business | 150 | 2,500 | 10,000 |

Free and 1-credit discovery endpoints still count. `429` carries `Retry-After`; honour it, with exponential backoff as fallback.

**Do not trust `X-RateLimit-Limit`.** It is computed without the plan multiplier, so a Business key enforced at 150/min has every response advertising 30. Derive your budget from your plan, not the header. `X-RateLimit-Reset` is always the next minute boundary, so it says nothing about an hour or day trip.

Batch endpoints are how you stay under these: 30 citations in one call is one request, not 30.

---

## 12. Caching that affects your answer

**Rankings are cached 300 seconds, shared across workers and customers.** The key covers the query and every ranking filter, but not `limit`, `offset`, `includeBody`, `excerptChars` or `fields`.

Consequences:

- Paging is stable by design: every page slices one ranking, so results never repeat or go missing.
- Page 2 costs no compute but still costs 4 credits.
- A corpus refresh landing mid-window is invisible until the TTL expires. If you need certainty about a just-published change, use the change endpoints, not search.
- Empty rankings are never cached, so a transient degradation cannot pin a thin result as the answer.

**Coverage metadata is cached 600 seconds with stale-while-revalidate, and a failing rebuild pins the last good value indefinitely** on purpose: "a stale row is accurate while a degraded row reports a real corpus as absent."
**Read `measuredAt`.** It can legitimately be hours old. It has been observed serving a 15.9-hour-old value at HTTP 200.

A cold coverage call can take a long time (a cold build is ~52 facet queries). A slow first call is expected, not a fault.

---

## 13. Worked patterns

### Verify a citation an LLM produced

```text
resolve_statute_citation("42 U.S.C. 1983")     2 credits
  resolved: false -> report UNVERIFIED, never "does not exist"
  resolved: true  -> read goodLawStatus on the section
                     good_law     -> current
                     unknown      -> unverified, say "verify against the source"
                     not_good_law -> lead with that
```

### "What does the law say about X in California"

```text
list_statutes_coverage                          free   confirm CA holds STATE
search_us_statutes(query=X, state="ca",         4      scope it
                   corpusType="STATE",
                   excludeRepealed=true)
  -> judge by score SEPARATION, not by rows returned
  -> choose from excerpts; do NOT quote them
get_us_statute_section_text(act_id,             6      the actual text
                            format="content" if USC)
get_section_definitions(act_id)                 4      only if a term is doing work
```

About 14 credits for a grounded answer.

### Point-in-time

```text
resolve_statute_citation(cite)                  2
get_section_changes(act_id)                     1      what was observed
get_us_statute_section_text(act_id, asOf=date)  6
  -> report text + the asOf block + coverage prose
  -> if isBounded, say what we can attest to, not what the law was
```

### Check 30 citations in a brief

```text
resolve_statute_citations_batch([...30...])     60     ONE request
  -> every input gets an entry; unresolved ones are named
get_sections_batch(resolved act_ids)            2 each returned
  -> read notFoundDetail.reason on any miss
```

Same price as one at a time, 2 requests instead of 60, and it keeps you inside the rate limit.

---

## 14. Reporting discipline

- **Quote the body, never the excerpt.** The excerpt can start mid-section and drop a subsection marker.
- **On USC, quote from `content`, not `plain`.** The median USC body is 54% commentary.
- Judge a result set by score separation. Rows always come back, even for nonsense.
- Give the citation exactly as reported, including subdivision. Use `citationShort` when a year would mislead.
- Cite `externalUrl` or `sourceUrl`. Never `source`, which is storage.
- Lead with `goodLawStatus` when it is not `good_law`. A correct quotation of dead law is still a wrong answer.
- Report `unknown` as unverified, and tell the user to check the official source.
- Surface the `coverage` prose on anything point-in-time or change-related. It is written to be shown verbatim.
- Distinguish "did not resolve" from "does not exist".
- If nothing was found, say nothing was found. Never fill the gap from memory.

---

## 15. Deprecated

| Deprecated | Use instead |
| --- | --- |
| `/states`, `/laws` | `list_statutes_coverage` (a superset) |
| `/codes` | `list_statute_divisions` |
| bare `/api/v1/statutes/*` | `/api/v1/us/statutes/*` |
| `total` on a search response | `count` |

The bare path still works and emits `Deprecation: true` with a successor link. There is no `Sunset` header and no committed removal date; existing keys keep working.
