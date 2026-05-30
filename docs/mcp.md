# MCP servers

This template ships a portable [Model Context Protocol](https://modelcontextprotocol.io) configuration in [`.mcp.json`](../.mcp.json) at the repo root, using the standard `mcpServers` schema understood by Copilot CLI, VS Code, Cursor, Claude Code, and other MCP clients.

## Bundled servers

| Server | Command | Purpose |
|---|---|---|
| `playwright` | `npx @playwright/mcp@latest` | Browser automation through Playwright's accessibility tree — navigate, click, fill, snapshot, capture network/console — for **agent-driven** browser control. |

## Registering the servers

The committed `.mcp.json` is picked up automatically by clients that read repo-root MCP config (VS Code, Cursor, Claude Code). For **GitHub Copilot CLI**, register it explicitly:

- Run `/mcp` in an interactive session and add the `playwright` server, **or**
- Copy the `mcpServers` block from [`.mcp.json`](../.mcp.json) into `~/.copilot/mcp-config.json`.

Requirements: Node.js 18+. The first run downloads the Playwright browser; in a fresh environment also run `npx playwright install --with-deps chromium`.

> ⚠️ **Allowlists.** Enterprises can restrict which MCP servers are permitted. If `playwright` does not appear after registration, check your organization's MCP allowlist.
>
> ⚠️ **Security.** Playwright MCP can drive a real browser. Treat any "run arbitrary code in page" tool as remote-code-execution-equivalent, point it only at local or trusted URLs, and never hand it production credentials.

## Playwright MCP vs the `app-testing` skill

The framework gives you **two** browser paths. They are complementary, not redundant:

| Use… | When |
|---|---|
| [`app-testing`](../.github/skills/app-testing/SKILL.md) (Python Playwright + `with_server.py`) | **Default for verification gates.** Deterministic, repeatable, committed E2E smoke tests that run in CI and back `PROJECT.md` gates. Token-efficient: no large tool schemas in context. |
| **Playwright MCP** (this server) | **Exploratory / self-healing loops.** Persistent browser state, live introspection of page structure, and iterative agent reasoning while debugging a UI or discovering selectors before codifying them into a committed test. |

Rule of thumb: explore and debug with **MCP**, then **codify** what you learned as a deterministic `app-testing` script that a verification gate can run unattended. Promote findings out of the interactive MCP loop into committed tests; do not leave verification depending on an interactive session.

## Adding more servers

Add an entry under `mcpServers` in [`.mcp.json`](../.mcp.json) and re-register. Servers that pair well with this framework:

- **Filesystem / Git** — scoped file and repository access.
- **Database** (Postgres, SQLite) — for `backend-service` profile work.
- **Fetch / web** — supplementary sources for the [`deep-research`](../.github/skills/deep-research/SKILL.md) agent.

Keep `.mcp.json` minimal and document any non-obvious server here so a fresh clone knows why it exists.
