# Primary Law

US primary law as agent tooling, by [Vaquill AI](https://www.vaquill.ai).
Search and cite statutes, regulations, constitutions and court rules from official government sources, resolve citations to exact provisions, and read a section as it stood on a past date.

## Install

```text
/plugin marketplace add Vaquill-AI/vaquill-plugin
/plugin install primary-law@vaquill
```

Then run `/mcp`, pick `vaquill-us`, and sign in. That is the whole setup: the server uses OAuth,
so there is no key to create or paste and no shell restart.

Then make one live call, because a green connection proves nothing: the server lists all 25 tools
with no credential at all. Ask for *"statutes coverage for California"*; `list_statutes_coverage`
costs 0 credits.

New accounts include 500 free credits ($5.00), enough for roughly 125 searches or 250 section
lookups. Pricing is at [vaquill.ai/legal-api](https://www.vaquill.ai/legal-api).

## What is in the box

### MCP server

`vaquill-us` covers the United States Code, all 50 state codes plus DC and Puerto Rico, federal and state regulations, court rules, state and federal constitutions, and agency guidance and adjudications.

### Skills

- `us-statute-research` - how to find a controlling provision, why `act_id` must come from a tool rather than be assembled, which entry point is cheapest, and how to report point-in-time text honestly.

### Commands

| Command | What it does |
| --- | --- |
| `/primary-law:research` | Find the controlling law on a question and answer only from the text the tools return. |
| `/primary-law:verify-citations` | Check every citation in a document against the official text, and flag anything no longer good law. |
| `/primary-law:compare-states` | How different states word the same rule, from one provision or a topic. |
| `/primary-law:as-of` | A section as it stood on a date, with the observation caveat attached. |
| `/primary-law:changed` | One-off sweep of what moved in a corpus since a date. |
| `/primary-law:rulemaking` | Track a Federal Register rulemaking: proposed, final, and when comments close. |
| `/primary-law:guidance` | Federal agency guidance on a topic, with what weight it does and does not carry. |
| `/primary-law:authority` | Trace a statute to the regulations implementing it, in either direction. |
| `/primary-law:adjudications` | Federal administrative decisions, with the overruling caveat stated up front. |
| `/primary-law:coverage` | What we actually hold for a jurisdiction, before you read an empty result as an absence. |

## What this is not

There is no general case law here.
The corpus is enacted and promulgated text from official publishers.
If a question needs holdings rather than statutory language, these tools cannot answer it.

Coverage is United States only.

## Pricing

Credits are $0.01 each. Discovery is free; text costs money.

| Cost | Operation |
| --- | --- |
| Free | coverage, pricing |
| 1 | divisions, section change timeline |
| 2 | section metadata, citation resolve, neighbours, cited-by |
| 4 | search, definitions |
| 6 | section body, cross-state comparison |

Batch tools are priced per item at the single-call rate. They save round trips, not credits.

Live pricing is available from the `get_pricing` tool.

## Links

- API docs: [vaquill.ai/legal-api](https://www.vaquill.ai/legal-api)
- MCP server source: [Vaquill-AI/vaquill-mcp](https://github.com/Vaquill-AI/vaquill-mcp)
- Support: <contact@vaquill.ai>

MIT.
