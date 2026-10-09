# DeployAngel plugin

Lets a coding agent check its own releases in production. After a deploy, the
agent waits for [DeployAngel](https://www.deployangel.com)'s verdict (cleared,
failed, or not cleared yet) and, when a release fails, reads the findings and
new exceptions to work out why.

It contains:

- **The `deployangel` MCP server**, with `wait_for_verification`,
  `get_verification`, `get_exercise_plan`, `list_deployments`,
  `get_exception`, and `list_late_regressions`. None of them can change
  production.
- **The `verify-deploy` skill**: when to wait for a verdict, and what to do
  with each outcome.

## Requirements

- An app that runs the DeployAngel agent: the `deployangel` gem (Rails) or
  the `deployangel` Python package (Django, FastAPI, Flask). The MCP server is
  that package's own `deployangel mcp` command, run with `bundle exec` in a
  Rails app, with `uv run` or `poetry run` in a Python app that uses them, and
  from your `PATH` otherwise.
- `DEPLOYANGEL_API_TOKEN` in the environment your agent runs in: a "CLI &
  coding agents" token from the app's Settings in DeployAngel. It's read-only.
  Keep it out of the repository, for example in an `.envrc` with direnv.
- macOS or Linux (the server starts through `sh`).

## Configuration

The plugin has no settings of its own. Set `DEPLOYANGEL_URL` only if you run
against a DeployAngel server other than `https://api.deployangel.com`.

To set up a single project without the plugin, run
`bundle exec deployangel install agents` (or `deployangel install agents`).
It writes the same MCP server into the project's Claude Code, Cursor, and Codex
config, and adds the instructions to `AGENTS.md`.
