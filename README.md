# Credible

The Credible plugin connects Claude to your [Credible](https://credibledata.com) workspaces so you can ask questions about your data in natural language, get grounded answers from your team's published Malloy semantic models, and run real queries against your warehouse — without leaving the conversation.

The plugin bundles:

- A Streamable HTTP **MCP server** at `https://mcp.credibledata.com/global` exposing the core analysis tools (`list_workspaces`, `get_context`, `execute_query`, `search_malloy_docs`).
- A set of **Skills** that teach Claude how to use those tools well — phrase detection, Malloy query patterns, chart selection, analysis pitfalls, Malloy docs lookup, and the canonical analysis workflow.

## What it does

Once installed, you can ask questions like *"how many active subscribers did we have last month?"* or *"break down revenue by region for Q1"*. The plugin gives Claude the tools and skills to:

1. Discover which Credible workspaces you have access to (`list_workspaces`).
2. Search the relevant semantic model for the entities (dimensions, measures, sources) that answer the question (`get_context`).
3. Compose a valid Malloy query against that model and run it against your warehouse (`execute_query`).
4. Return rows, summary stats, and chart-ready output to the chat.

You stay in control: every query is shown before it runs, results are streamed back into the chat, and the underlying Credible permissions (org / workspace / package) are enforced server-side.

## Requirements

- An active [Credible](https://credibledata.com) account with access to at least one published workspace.
- A Claude surface that supports plugins:
  - **Claude Cowork** (Desktop)
  - **Claude Code** (Desktop UI or CLI)
  - **Claude Desktop** (chat — via "Upload plugin")
- Outbound HTTPS access to `mcp.credibledata.com`.

The first time you use the plugin, Claude will prompt you to sign in to Credible via OAuth. To revoke access, uninstall the plugin in your Claude client (Cowork → *Plugins*, Claude Desktop → *Settings → Plugins*, or `claude plugin uninstall credible@<marketplace>`).

## Install

### From the community plugin marketplace (recommended)

In **Claude Code**:

```bash
/plugin marketplace add anthropics/claude-plugins-community
/plugin install credible@claude-community
```

In **Cowork**: open *Organization settings → Plugins → Browse marketplace* and pick **Credible**.

### Manually from this repo

```bash
# Claude Code (local clone install)
git clone https://github.com/credibledata/credible-plugin.git ~/credible-plugins/credible
claude --plugin-dir ~/credible-plugins/credible

# Claude Code CLI marketplace add (no clone)
/plugin marketplace add credibledata/credible-plugin
/plugin install credible@credible
```

### From a release zip

Grab `credible.zip` from the [latest release](https://github.com/credibledata/credible-plugin/releases/latest) and upload it via:

- **Cowork** → *Organization settings → Plugins → Upload plugin*
- **Claude Desktop** → *Settings → Plugins → Upload plugin*

## Usage examples

> Try these prompts after installing. They assume you have at least one published workspace; Claude will help you pick one if you have several.

### 1. Sanity check — explore what's available

> *"What Credible workspaces do I have access to, and what kind of data is in each?"*

Claude will call `list_workspaces` and summarize the orgs / workspaces it sees, then offer to dive into one. Good first-run prompt to confirm the OAuth flow worked end-to-end.

### 2. A natural-language data question

> *"What was our total revenue for each region in Q1 of last year?"*

Claude will use the plugin's tools and skills to identify the right `revenue` measure and `region` dimension in the workspace's semantic model (`get_context`), compose a valid Malloy query with the appropriate time filter, run it (`execute_query`), and return both the table and a renderable chart spec. The `credible-analysis` and `malloy-charts` skills guide it through each step.

### 3. A more advanced analytical question

> *"Compute year-over-year growth in monthly active users, broken down by plan tier, for the last four quarters."*

Year-over-year comparisons aren't a one-liner in Malloy. Claude will use `get_context` to identify the right active-user measure and the `plan_tier` dimension, look up the time-shifting pattern via `search_malloy_docs` if it's unsure, then compose and run a single query via `execute_query`. The `analysis-pitfalls` skill flags caveats — for example, the ambiguity in "monthly" (calendar month vs. trailing 30 days) and partial-period under-counts.

## Tools

| Tool | What it does | Read-only |
|---|---|---|
| `list_workspaces` | Lists every Credible workspace the caller can access, grouped by org. | Yes |
| `get_context` | Searches a workspace's semantic model for sources, dimensions, measures, views, and dimensional values matching the user's question. | Yes |
| `execute_query` | Executes a Malloy query (or a predefined view) against the workspace and returns rows + render metadata. | Yes (queries cannot mutate data) |
| `search_malloy_docs` | Searches the Malloy documentation for syntax, language features, and code examples — used by Claude to look up unfamiliar patterns before writing a query. | Yes |

All tools are annotated with `readOnlyHint: true` — the plugin cannot write back to your warehouse or your Credible workspace.

## Privacy

See [PRIVACY.md](./PRIVACY.md) for the full policy. The plugin only shares the OAuth identity, the questions you ask, and the resulting query results with Credible's servers; it does not transmit credentials or warehouse data to any third party other than the LLM provider already mediating the chat. Credible's privacy policy is also published at <https://credibledata.com/privacy>.

## Support

- **Docs**: <https://docs.credibledata.com>
- **Email**: support@credibledata.com
- **Issues** (this plugin): <https://github.com/credibledata/credible-plugin/issues>

## License

See [LICENSE](./LICENSE).
