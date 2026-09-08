---
name: setup
description: Guide the user through connecting the Vaquill AI US legal-data MCP server. Use when the plugin is first installed, when a Vaquill AI tool returns 401, 402 or 403, or when the user asks how to connect or sign in to Vaquill AI.
---

# Connecting Vaquill AI

The plugin ships one MCP server, `vaquill-us`, covering United States primary law:
the US Code, all 50 state codes plus DC and Puerto Rico, federal and state regulations,
court rules, constitutions, agency guidance and agency adjudications.

**It authenticates with OAuth. There is no API key to create, paste, or store.**

## Step 1: sign in

On first use, Claude prompts to connect `vaquill-us`.
Run `/mcp`, select `vaquill-us`, and follow the sign-in.

The browser opens Vaquill, the user signs in with their normal Vaquill account, and a
consent screen names what Claude is asking for. Approve it once. The decision is
remembered, so reconnecting later does not ask again.

That is the whole setup. No environment variable, no shell restart, no config file.

**This works in Cowork.** A Cowork session runs in an Anthropic cloud container with no
user-settable environment, which is why the previous API-key setup could not work there
at all. OAuth is held by Claude rather than by the machine, so the cloud surfaces behave
the same as the desktop ones.

## Step 2: one live call, because a green connection proves nothing

**A green connection does not mean the user is authenticated.** Measured 2026-09-07: the
server completes the MCP handshake and lists all 25 tools with **no credential at all**.
Authentication is enforced at the API call behind each tool, not at the protocol layer.

So `claude mcp list` showing `Connected`, and a full tool list in `/mcp`, are both
consistent with a completely broken credential. **The only real smoke test is one live
call**, and the cheapest is free:

> "Use Vaquill to list statutes coverage for California."

`list_statutes_coverage` costs 0 credits. An error there means the connection never
carried an identity; a normal response means setup is done.

Test each surface separately if the user works across several. Claude Code, claude.ai,
Claude Desktop and Cowork take different code paths, and Cowork syncs its tooling at
session start, so it needs a fresh session rather than a reload.

## What happens to the user's API key

They do not create one. On first sign-in Vaquill provisions a key for the connector
automatically and holds it server-side. Claude never sees it.

- **It does not count against the plan's key cap.** Connector keys are tracked separately
  from hand-created ones, so connecting cannot consume one of the 2 (or 5, on Business)
  active-key slots, and cannot block the user from creating one.
- **It does not appear in the dashboard key list**, which shows only keys the user made.
  Disconnecting in Claude stops Claude using it but does not itself revoke it.

Billing is unchanged: the same credit balance as any other key. Check the balance in the
console before a long session. There is no free tier published; do not promise free usage.

## If the user already has an API key

They do not need it for this plugin, and it is not read. A `vq_key_` is still the right
credential for a non-Claude client (Cursor, VS Code, Windsurf) or for calling the REST
API directly, both documented at <https://www.vaquill.ai/docs>.

Never write a key into `.mcp.json`, a committed `.env`, or any file under version
control, and never print a full key back into the transcript.

## Attribution

**On the Pro plan, attribution is required**: a visible "Legal data by Vaquill AI" credit,
linked to the site, wherever the data is surfaced.
Business waives it.

If the user is building something customer-facing on Pro, say this once at setup. It is a
licence term, not a suggestion.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| Connected, but every tool call fails | The connection carries no identity. Re-run `/mcp` and complete the sign-in; do not stop at "Connected" |
| Asked to approve twice | Approve the second one. The decision is remembered from then on |
| 402 | Out of credits. Top up in the console |
| 429 | Rate limited. Honour `Retry-After`. Note `X-RateLimit-Limit` under-reports on Business keys |
| 503 | Backend unavailable. The call was refunded. Retry later |
| Tools missing entirely | The server failed to connect. Check `/mcp` before assuming auth is wrong |
| Tools missing in Cowork | Cowork syncs at session start. Start a new session rather than reloading |

An empty result set is not an authentication problem, and since an outage returns 503 with
a refund, an empty 200 is a real answer.
See the `us-statute-research` skill for why an empty result is more often a malformed
`act_id` than a coverage gap.
