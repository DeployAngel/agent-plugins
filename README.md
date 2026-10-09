# DeployAngel plugins for coding agents

[DeployAngel](https://www.deployangel.com) verifies every production deploy
of a Rails, Django, FastAPI, or Flask app: it compares real traffic after the
deploy with the last healthy release, then clears the release, fails it, or
says what it's still waiting to see.

The `deployangel` plugin lets Claude Code, Cursor, and Codex wait for that
verdict after they deploy, and read the evidence when a release fails. See
[plugins/deployangel](plugins/deployangel/README.md) for what it contains and
needs.

## Install

**Claude Code**

```bash
claude plugin marketplace add DeployAngel/agent-plugins
claude plugin install deployangel@deployangel
```

**Codex**

```bash
codex plugin marketplace add DeployAngel/agent-plugins
```

Then install `deployangel` from the plugin list. Codex starts MCP servers with
a limited environment, so if the server reports a missing token, forward it in
your Codex config:

```toml
[mcp_servers.deployangel]
env_vars = ["DEPLOYANGEL_API_TOKEN", "DEPLOYANGEL_URL"]
```

**Cursor**

Install DeployAngel from the Cursor Marketplace.

**Any project, without a plugin**

```bash
bundle exec deployangel install agents    # Rails
deployangel install agents                # Python
```

## Every agent needs a token

Set `DEPLOYANGEL_API_TOKEN` to a "CLI & coding agents" token, from the app's
Settings in DeployAngel, in the environment your agent runs in. Keep it out
of the repository.

## Layout

One plugin, described for each tool:

| Tool | Marketplace | Plugin manifest | MCP servers |
|---|---|---|---|
| Claude Code | `.claude-plugin/marketplace.json` | `.claude-plugin/plugin.json` | `.mcp.json` |
| Cursor | `.cursor-plugin/marketplace.json` | `.cursor-plugin/plugin.json` | `mcp.json` |
| Codex ([Agent Plugins](https://agent-plugins.org)) | `.agents/plugins/marketplace.json` | `plugin.json` | `mcp.json` |

All three share `skills/verify-deploy/SKILL.md`. Keep the two MCP files and
the three manifests in step when you change them.

## License

MIT
