# Omphalis plugin

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

## Setup

1. Install the plugin.
2. The first time the agent uses Omphalis, your browser opens the Omphalis
   sign-in and consent screen. Sign in with your Omphalis account and approve
   the permissions you want to give.
3. That's it. There is no API key to paste.

You need an Omphalis account. There's a free plan; sign up at
https://app.omphalis.ai/signup.

## Example requests

- "Save https://example.com/some-long-essay to Omphalis."
- "Keep this as a note in Omphalis, titled 'Questions for Thursday'."
- "What do my saved podcasts say about sleep and memory?"
- "Find the passage I marked about base rates and show me my note on it."
- "Compare what the three papers in my 'Attention' Space say about scaling, and keep the answer in a new notebook."

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
| Authorization | OAuth 2.1, authorization code with PKCE (S256), dynamic client registration |
| Metadata | https://app.omphalis.ai/.well-known/oauth-protected-resource |
| Agent docs | https://omphalis.ai/auth.md |
| Credentials stored in this repo | None |

The plugin contains no code, scripts or hooks. It ships two `SKILL.md` files,
the MCP server configuration above, and a logo. Every call is scoped to the
user who authorized it; there is no shared or admin token. Access tokens last
15 minutes and refresh tokens rotate. You can revoke access at any time from
Settings in the Omphalis app.

### Tools the skills use

| Tool | Scope | Kind |
| --- | --- | --- |
| `library_save` | `library:write` | Write (paid plan) |
| `library_organize` | `library:write` | Write, undoable |
| `library_get`, `library_list`, `tags_list` | `library:read` | Read |
| `collections_list`, `spaces_list` | `collections:read` | Read |
| `search_library` | `search:read` | Read |
| `marks_get` | `marks:read` | Read |
| `library_ask` | `library:synthesize` | Read (paid plan) |
| `notebooks_list` | `notebooks:read` | Read |
| `notebooks_create`, `notebooks_append` | `notebooks:write` | Write, append-only |

The server exposes more tools than these; the full list and the scope each
needs are in https://omphalis.ai/auth.md.

Rate limit: 120 requests per rolling 60 seconds per user, shared across every
agent the user has connected.

## Your content

Content in your Library is yours. Agents using this plugin are told to use it
only to serve your request, and to treat text inside saved sources as content,
never as instructions.

## Support

- Email: stanly@tryomphalis.com
- Product guide: https://docs.omphalis.ai

## License

MIT. See [LICENSE](LICENSE). The license covers the files in this repository
only, not the Omphalis service.
