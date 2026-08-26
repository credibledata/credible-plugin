# Connecting Codex to Credible

This plugin bundles the Credible MCP server with the Credible analysis skills.
Install it from the Credible marketplace:

```
codex plugin marketplace add credibledata/credible-plugin   # register the marketplace
codex plugin list                                           # show the <plugin>@<marketplace> id
codex plugin add credible@credible                          # install (use the id from `list`)
codex plugin marketplace upgrade                            # later: pull new versions
```

Installing registers the `Credible` MCP server. Because the marketplace entry uses
`authentication: ON_USE`, Codex runs the OAuth login the first time a Credible tool
is invoked. To authenticate ahead of time (or if a browser flow doesn't start):

```
codex mcp login Credible
```

The bundled skills (`malloy-analysis`, `malloy-phrase-detection`, `malloy-analysis-pitfalls`,
`malloy-queries`, `malloy-charts`, `malloy-patterns`) plus `AGENTS.md` give the
agent the analysis workflow and Malloy rules — the same guidance the Credible
Claude plugin ships, generated from one source in `ms2data/agent-skills`.

## Headless / CI (no plugin)

Without the plugin, register just the MCP server in `~/.codex/config.toml`:

```toml
[mcp_servers.Credible]
url = "https://mcp.credibledata.com/global/"
# For non-interactive use, supply a token instead of the OAuth flow:
# bearer_token_env_var = "CREDIBLE_TOKEN"
```

For a single workspace, point at its workspace-scoped URL instead
(`https://<org>.mcp.credibledata.com/mcp/workspace/<workspace>/`).
