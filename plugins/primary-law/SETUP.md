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

## Cowork and other cloud sessions: not supported yet

**Stop here if the user is in Cowork, or any Claude session running in an Anthropic cloud container rather than on their own machine.** Everything below is a terminal instruction with no cloud equivalent.

Measured 2026-09-07 in a real Cowork session:

- The plugin installs, and the skill and all ten commands load correctly.
- The MCP server never connects, because `VAQUILL_API_KEY` is unset in the container and `${VAQUILL_API_KEY}` has nothing to expand to.
- A cloud container never sees the user's own machine, so `~/.zshenv` is irrelevant there.

This is not a gap in this plugin. `user_config`, the mechanism Anthropic designed for passing credentials to a bundled MCP server, does not reach the server process in Cowork either: [anthropics/claude-code#39125](https://github.com/anthropics/claude-code/issues/39125), labelled a bug and closed as `not planned`. The only workaround in that thread is per-session and lost when the ephemeral VM ends.

So there is currently **no durable way for a cloud session to hold this credential**. Say that plainly rather than walking a Cowork user through the steps below, which will end in a 401 with no path forward. Cowork support needs OAuth on `mcp.vaquill.ai`, which is server work rather than a change to this plugin.

## Step 2: put the key in `~/.zshenv`, not `~/.zshrc`

This is the step that goes wrong, so walk the user through it exactly.

**Claude Code only.** `${VAQUILL_API_KEY}` expansion reads the *process* environment, so it needs a shell you control. Verified 2026-09-07: it works end to end in Claude Code once the variable actually reaches the process, and it **cannot work in Cowork**, which has no user-settable environment and no shell. There the template is sent literally, the connection still reports healthy, and every tool call returns 401. If the user is in Cowork, say the plugin does not support it yet rather than walking them through anything below.

```bash
echo 'export VAQUILL_API_KEY="vq_key_YOUR_KEY"' >> ~/.zshenv
chmod 600 ~/.zshenv
```

**`~/.zshrc` is not enough.** It is sourced only for *interactive* shells. Claude Code and the processes it spawns are not always interactive, and `~/.zshenv` is sourced for every zsh invocation. On bash, the equivalent is `~/.bash_profile` plus `~/.bashrc`.

A bare `export` typed into a terminal is also not enough. It dies with that shell.

Verify it reaches a **new** shell, which is what actually matters:

```bash
zsh -c 'echo ${VAQUILL_API_KEY:+PRESENT}'    # must print PRESENT
```

Use the `:+` form. `${VAR:-...}` prints the key itself and will leak it into the transcript.

Do not write the key into `.mcp.json`, into a committed `.env`, or into any file under version control.
The variable indirection in `.mcp.json` exists precisely so the key never lands in the repo.
Never print a full key back into the transcript.

## Step 3: fully quit and relaunch Claude Code

Environment is inherited **once, at process start**. So none of these pick up a new variable:

- `/reload`
- MCP servers reconnecting
- Starting a new session in the same window
- Reinstalling the plugin

Quit Claude Code itself (`/exit`, or Ctrl+D), then run `claude` again from a shell that has the variable.

Prove it took:

```bash
pgrep -f claude    # must be a DIFFERENT number than before
```

If the pid is unchanged, the variable is not there and every tool call will return 401 no matter what `/mcp` says.

## Step 4: one live call, because a green connection proves nothing

**A green connection does not mean the key works.** Measured 2026-09-07: the server completes the MCP handshake and lists all 25 tools with no `Authorization` header at all, with an unexpanded `${VAQUILL_API_KEY}` template, and with an invalid key. Authentication is enforced at the API call behind each tool, not at the protocol layer.

So `claude mcp list` showing `Connected`, and a full tool list in `/mcp`, are both consistent with a completely broken credential. **The only real smoke test is one live call**, and the cheapest is free:

> "Use Vaquill to list statutes coverage for California."

`list_statutes_coverage` costs 0 credits. A 401 there means the key never reached the server; a normal response means setup is done.

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
