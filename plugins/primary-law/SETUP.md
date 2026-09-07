---
name: setup
description: Guide the user through connecting the Vaquill AI US legal-data MCP server. Use when the plugin is first installed, when a Vaquill AI tool returns 401, 402 or 403, or when the user asks how to get or set a Vaquill AI API key.
---

# Connecting Vaquill AI

The plugin ships one MCP server, `vaquill-us`, covering United States primary law:
the US Code, all 50 state codes plus DC and Puerto Rico, federal and state regulations,
court rules, constitutions, agency guidance and agency adjudications.

It authenticates with a `vq_key_` API key read from the `VAQUILL_API_KEY` environment variable.

## Step 1: get a key

Keys are created at <https://app.vaquill.ai/settings> under **API Keys**.
Plans and current pricing are at <https://www.vaquill.ai/legal-api>.

**Copy the key when it is shown. It is not shown again.**

Two things to tell the user before they start:

- **Active key caps are low.** Business allows 5 keys, every other plan allows 2. The console's own Playground mints a key when it cannot reuse one, so one integration key plus Playground use already hits the cap of 2.
- **Rotation differs by plan.** Business can create the new key before revoking the old one, so there is no downtime. Other plans must revoke first and accept a gap.

Check the credit balance in the console before a long session. There is no free tier published; do not promise the user free usage.

## Step 2: set the environment variable

```bash
export VAQUILL_API_KEY="vq_key_..."
```

Put it in the shell profile rather than typing it per session, then restart Claude Code so the plugin's MCP server picks it up.

Do not write the key into `.mcp.json`, into a committed `.env`, or into any file under version control.
The variable indirection in `.mcp.json` exists precisely so the key never lands in the repo.
Never print a full key back into the transcript.

If the variable does not expand (this can happen when Claude Code is launched from the desktop app rather than a shell), the fallback is the path-token form of the server URL, `https://mcp.vaquill.ai/s/vq_key_...`. **Treat that URL as a secret**: it carries the credential in the path, so it must not be shared, logged or committed.

## Step 3: confirm the connection

Run `/mcp` and check that `vaquill-us` is connected before running a research task.
A server that failed to start looks identical to a corpus with no results, and the second reading wastes the user's time.

## Attribution

**On the Pro plan, attribution is required**: a visible "Legal data by Vaquill AI" credit, linked to the site, wherever the data is surfaced.
Business waives it.

If the user is building something customer-facing on Pro, say this once at setup. It is a licence term, not a suggestion.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| 401 on every tool | Key missing, malformed, or the `Bearer ` prefix was dropped. The key alone is rejected |
| 401 right after revoking a key | Key validation is cached about 60 seconds. A revoked key can work briefly |
| 402 | Out of credits. Top up in the console |
| 429 | Rate limited. Honour `Retry-After`. Note `X-RateLimit-Limit` under-reports on Business keys |
| 503 | Backend unavailable. The call was refunded. Retry later |
| Tools missing entirely | The server failed to connect. Check `/mcp` before assuming the key is wrong |

An empty result set is not an authentication problem, and since an outage returns 503 with a refund, an empty 200 is a real answer.
See the `us-statute-research` skill for why an empty result is more often a malformed `act_id` than a coverage gap.
