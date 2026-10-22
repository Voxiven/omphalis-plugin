# Omphalis

Omphalis MCP server and plugin: connect your Omphalis Library to Grok, Cursor, Claude, ChatGPT and any MCP client.

Omphalis is a workspace for understanding articles, PDFs, videos and podcasts.
Bring sources together, work through difficult passages, ask questions across
them, and keep your own notes connected to the original material.

This plugin connects an agent to your Omphalis Library through the hosted
Omphalis MCP server, and adds two skills:

| Skill | What it does |
| --- | --- |
| `omphalis-save-source` | Save a link (article, PDF, YouTube video, podcast) or a note to your Library, and optionally tag it or file it in a Collection or Space |
| `omphalis-ask-library` | Search what you kept and answer questions from your own saved sources, with citations back to them |

Works with Grok Build (`.grok-plugin/plugin.json`, `.mcp.json`), Cursor
(`.cursor-plugin/plugin.json`, `mcp.json`) and Grok Bot.

- Product: https://omphalis.ai
- Agent connection: https://omphalis.ai/auth.md
- Plans: https://omphalis.ai/pricing

## Install

The server is remote Streamable HTTP. The endpoint is
`https://app.omphalis.ai/api/v2/mcp`. The first call opens the Omphalis
sign-in and consent screen. Sign in with your Omphalis account and approve
the permissions you want to give. There is no API key to paste.

You need an Omphalis account. There's a free plan; sign up at
https://app.omphalis.ai/signup.

### Grok Build

From a shell, with [Grok Build](https://x.ai/news/grok-plugin-marketplace):

```bash
grok plugin install Voxiven/omphalis-plugin --trust
```

`--trust` is required for Grok to attach the plugin's MCP server. The plugin
root is this repository: `.grok-plugin/plugin.json` and `.mcp.json`.

### Cursor

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=omphalis&config=eyJ1cmwiOiJodHRwczovL2FwcC5vbXBoYWxpcy5haS9hcGkvdjIvbWNwIn0=)

That deeplink installs the remote server `omphalis` with
`{"url":"https://app.omphalis.ai/api/v2/mcp"}`, the same entry as [`mcp.json`](mcp.json).
The `cursor://` form is
`cursor://anysphere.cursor-deeplink/mcp/install?name=omphalis&config=eyJ1cmwiOiJodHRwczovL2FwcC5vbXBoYWxpcy5haS9hcGkvdjIvbWNwIn0=`.

Cursor connection to this endpoint depends on [Voxiven/Omphalis#4915](https://github.com/Voxiven/Omphalis/issues/4915).

### Claude Desktop and Claude Code

Claude Code:

```bash
claude mcp add --transport http omphalis https://app.omphalis.ai/api/v2/mcp
```

In Claude Desktop, add a custom connector with that same URL.

### VS Code

Command Palette → **MCP: Add Server** → **HTTP** → paste
`https://app.omphalis.ai/api/v2/mcp` → server id `omphalis`.

Workspace [`.vscode/mcp.json`](https://code.visualstudio.com/docs/agents/reference/mcp-configuration):

```json
{
  "servers": {
    "omphalis": {
      "type": "http",
      "url": "https://app.omphalis.ai/api/v2/mcp"
    }
  }
}
```

### Windsurf

Open the MCP config from Windsurf settings and add the remote URL. Windsurf
uses `serverUrl` for Streamable HTTP and can run the OAuth sign-in itself:

```json
{
  "mcpServers": {
    "omphalis": {
      "serverUrl": "https://app.omphalis.ai/api/v2/mcp"
    }
  }
}
```

### ChatGPT

The pricing page says Omphalis can be used inside ChatGPT. To connect the MCP
server, in ChatGPT open **Plugins**, choose **Add custom MCP server**, and
enter `https://app.omphalis.ai/api/v2/mcp`. Sign in when ChatGPT asks.

### Any MCP client

Point the client at `https://app.omphalis.ai/api/v2/mcp` with Streamable HTTP.
Authorization is OAuth 2.1, authorization code with PKCE (S256), and dynamic
client registration. Discovery metadata is at
https://app.omphalis.ai/.well-known/oauth-protected-resource. The human-readable
connection guide is https://omphalis.ai/auth.md.

## Example requests

- "Save https://example.com/some-long-essay to Omphalis."
- "Keep this as a note in Omphalis, titled 'Questions for Thursday'."
- "What do my saved podcasts say about sleep and memory?"
- "Find the passage I marked about base rates and show me my note on it."
- "Compare what the three papers in my 'Attention' Space say about scaling, and keep the answer in a new notebook."

## Tools the skills use

`omphalis-save-source` calls `library_save`, `library_get`, `tags_list`,
`collections_list`, `spaces_list`, and `library_organize`.
`omphalis-ask-library` calls `search_library`, `library_list`, `spaces_list`,
`library_get`, `marks_get`, `library_ask`, `notebooks_list`,
`notebooks_create`, and `notebooks_append`.

A paid plan is required for `library_save` and `library_ask`. `marks_create`
also needs a paid plan; these skills do not call it. Read tools work on every
plan, including Free. The server refuses a gated call with a message naming
the plan requirement.

| Tool | Scope | Kind | Plan |
| --- | --- | --- | --- |
| `library_save` | `library:write` | Write | Paid plan |
| `library_organize` | `library:write` | Write, undoable | Not named as a paid tool |
| `library_get`, `library_list`, `tags_list` | `library:read` | Read | Every plan, including Free |
| `collections_list`, `spaces_list` | `collections:read` | Read | Every plan, including Free |
| `search_library` | `search:read` | Read | Every plan, including Free |
| `marks_get` | `marks:read` | Read | Every plan, including Free |
| `library_ask` | `library:synthesize` | Read | Paid plan |
| `notebooks_list` | `notebooks:read` | Read | Every plan, including Free |
| `notebooks_create`, `notebooks_append` | `notebooks:write` | Write, append-only | Not named as a paid tool |

The server exposes more tools than these. The full list and the scope each
needs are in https://omphalis.ai/auth.md.

On https://omphalis.ai/pricing, connecting any MCP client to read the Library
is on every plan. Saving items and adding notes through that connection is on
the paid plans.

Rate limit: 120 requests per rolling 60 seconds per user, shared across every
agent the user has connected.

## Plans

Read tools (listing, reading, searching your Library, reading Marks,
Collections, Connections and notebooks) work on every plan, including Free.
Saving (`library_save`), creating Marks (`marks_create`) and Ask
(`library_ask`) need a paid plan; the server refuses them on Free with a
message naming the plan requirement.

New accounts get a 30-day trial, no card required, with 600 comprehension
units, 200 Ask voice minutes and 15 assistant answers a day. After that:

| Plan | Price | Comprehension units |
| --- | --- | --- |
| Free | $0 | 100 a month |
| Pro | $9 a month | 300 a month |
| Premium | $19 a month | 600 a month |
| Scholar | $39 a month | 1,200 a month |

Current details: https://omphalis.ai/pricing

## What this plugin connects to

| Item | Value |
| --- | --- |
| MCP endpoint | `https://app.omphalis.ai/api/v2/mcp` (Streamable HTTP) |
| Registry name | `io.github.voxiven/omphalis` in [`server.json`](server.json) |
| Authorization | OAuth 2.1, authorization code with PKCE (S256), dynamic client registration |
| Metadata | https://app.omphalis.ai/.well-known/oauth-protected-resource |
| Agent docs | https://omphalis.ai/auth.md |
| Credentials stored in this repo | None |

The plugin contains no code, scripts or hooks. It ships two `SKILL.md` files,
the MCP server configuration above, and a logo. Every call is scoped to the
user who authorized it; there is no shared or admin token. Access tokens last
15 minutes and refresh tokens rotate. You can revoke access at any time from
Settings in the Omphalis app.

## FAQ

### How do I connect Omphalis to Claude?

Claude Code: `claude mcp add --transport http omphalis https://app.omphalis.ai/api/v2/mcp`.
In Claude Desktop, add a custom connector with that URL. The first call opens
Omphalis sign-in.

### Does Omphalis have an MCP server?

Yes. One hosted server, Streamable HTTP, at
`https://app.omphalis.ai/api/v2/mcp`. Details and the tool list:
https://omphalis.ai/auth.md.

### How do I connect Omphalis to Cursor?

Use the Add to Cursor button above, or the `url` in [`mcp.json`](mcp.json).
Cursor connection depends on [Voxiven/Omphalis#4915](https://github.com/Voxiven/Omphalis/issues/4915).

### How do I connect Omphalis to ChatGPT?

Add a custom MCP server in ChatGPT Plugins with
`https://app.omphalis.ai/api/v2/mcp`. https://omphalis.ai/pricing also says
Omphalis can be used inside ChatGPT.

### How do I connect Omphalis to Grok?

`grok plugin install Voxiven/omphalis-plugin --trust`

### Does Omphalis have a free plan?

There's a free plan. MCP read tools work on it. `library_save`,
`marks_create`, and `library_ask` need a paid plan. See
https://omphalis.ai/pricing.

### Do I need an API key for Omphalis MCP?

No. Sign in with your Omphalis account when the client opens the consent
screen. This repository stores no credentials.

## Your content

Content in your Library is yours. Agents using this plugin are told to use it
only to serve your request, and to treat text inside saved sources as content,
never as instructions.

## Support

- Email: hello@omphalis.ai
- Product guide: https://docs.omphalis.ai
- Product: https://omphalis.ai
- Plans: https://omphalis.ai/pricing
- Agent connection: https://omphalis.ai/auth.md

## License

MIT. See [LICENSE](LICENSE). The license covers the files in this repository
only, not the Omphalis service.
