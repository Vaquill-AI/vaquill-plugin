# Vaquill AI plugin marketplace

Claude Code plugins for [Vaquill AI](https://www.vaquill.ai), US primary law as agent tooling.

**Works everywhere Claude runs, including Cowork.** The bundled MCP server authenticates with
OAuth, so there is no key to set and nothing to read from the environment. Until 2026-09-08 it read
a `vq_key_` from `VAQUILL_API_KEY`, which could not work in Cowork at all: that container has no
user-settable environment, so the template was sent literally and every tool call 401d while the
connection still showed as healthy.

```text
/plugin marketplace add Vaquill-AI/vaquill-plugin
/plugin install primary-law@vaquill
```

## Plugins

| Plugin | Description |
| --- | --- |
| [`primary-law`](plugins/primary-law) | Search and cite US primary law from official sources, resolve citations to exact sections, and read a section as it stood on a past date. |

## Layout

```text
.claude-plugin/marketplace.json      the catalogue
plugins/primary-law/
  .claude-plugin/plugin.json         manifest
  .mcp.json                          the US MCP server
  SETUP.md                           OAuth sign-in, walked through on install
  skills/                            research judgment the tool schemas cannot carry
  commands/                          user-invoked workflows
```

The corpus is United States only.
When a second country ships, it is one more entry in `.mcp.json` and one more section in `SETUP.md`, not a second plugin: a customer working across two countries should install once.
The jurisdiction split lives in the MCP servers, which publish disjoint tool sets.

## Developing

```bash
claude plugin validate .
/plugin marketplace add ./vaquill-plugin    # local, for testing before pushing
```

Bump `version` in `plugins/primary-law/.claude-plugin/plugin.json` on every release.
Users only receive updates when that field changes.

MIT.
