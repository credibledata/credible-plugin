# Connecting OpenCode to Credible

Credible connects OpenCode to your published Malloy semantic models so you can
ask data questions, discover the correct measures and dimensions, and execute
grounded warehouse queries from your coding session.

This integration includes a remote MCP configuration and the Credible analysis
skills. The skills provide the retrieval, Malloy, charting, and verification
workflow; the MCP tool descriptions provide parameter contracts.

## Requirements

- [OpenCode](https://opencode.ai) with outbound HTTPS access to
  `mcp.credibledata.com`.
- An active [Credible](https://credibledata.com) account with access to at
  least one published workspace.

## Install

Clone the Credible integration repository, then enter its OpenCode subtree:

```bash
git clone https://github.com/credibledata/credible-plugin.git ~/credible-plugins/credible
cd ~/credible-plugins/credible/opencode
```

Replace only the bundled skills in OpenCode's global skill directory. This
avoids leaving a stale file behind when a Credible skill changes or is removed:

```bash
mkdir -p ~/.config/opencode/skills
for skill in ./skills/*/; do
  name=${skill%/}
  name=${name##*/}
  rm -rf ~/.config/opencode/skills/"$name"
  cp -R "$skill" ~/.config/opencode/skills/"$name"
done
```

Merge the `mcp.credible` entry from `./opencode.jsonc` into
`~/.config/opencode/opencode.jsonc`. If you do not already have an OpenCode
configuration, use this complete file:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "credible": {
      "type": "remote",
      "url": "https://mcp.credibledata.com/global/",
      "enabled": true
    }
  }
}
```

Restart OpenCode after changing its configuration or installed skills.

## Authenticate And Verify

Authenticate with Credible:

```bash
opencode mcp auth credible
```

OpenCode opens a browser for OAuth and stores the resulting tokens securely.
After authentication, Credible's MCP tools and skills are available in every
OpenCode project. Confirm the connection or diagnose an OAuth failure with:

```bash
opencode mcp list
opencode mcp debug credible
```

## What It Provides

OpenCode registers the following tool names with the `credible_` prefix:

| Tool | What it does |
|---|---|
| `credible_list_workspaces` | Lists Credible workspaces available to you. |
| `credible_get_context` | Searches a workspace semantic model for sources, measures, dimensions, views, and values. |
| `credible_execute_query` | Executes a read-only Malloy query and returns rows plus render metadata. |
| `credible_search_malloy_docs` | Searches Malloy documentation for syntax and examples. |

The bundled `credible-analysis` skill is the entry point for data questions. It
loads the focused Malloy and analysis skills as needed and requires context
retrieval before constructing a query.

Example prompts:

- "What Credible workspaces can I access? Use Credible."
- "What was total revenue by region last quarter? Use Credible."
- "Find the documented Malloy pattern for year-over-year growth. Use Credible."

## Update

Pull the latest integration files, replace the refreshed Credible skills, then
restart OpenCode:

```bash
cd ~/credible-plugins/credible
git pull --ff-only
cd opencode
for skill in ./skills/*/; do
  name=${skill%/}
  name=${name##*/}
  rm -rf ~/.config/opencode/skills/"$name"
  cp -R "$skill" ~/.config/opencode/skills/"$name"
done
```

## Workspace Scope

The global endpoint lets the agent call `credible_list_workspaces` and select a
workspace per request. To lock an installation to one workspace, replace the
URL in the `credible` MCP entry with:

```text
https://<org>.mcp.credibledata.com/mcp/workspace/<workspace>/
```
