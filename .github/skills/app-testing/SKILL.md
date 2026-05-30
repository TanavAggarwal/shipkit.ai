---
name: app-testing
description: 'Stack-agnostic local dev/test harness for autonomous self-test with Playwright. Boots one or more project servers using `scripts/with_server.py`, drives the web-accessible app with Python Playwright scripts, captures browser behavior, and tears servers down. Commands, ports, package paths, dependency setup, and required gates come from PROJECT.md and the active stack profile. USE FOR: e2e smoke tests, UI debugging, browser console capture, screenshots, and verification of locally running web apps or web builds of multi-platform apps. DO NOT USE FOR: pure unit tests or non-web native device automation unless the active profile adds such tooling. OUTPUT: Playwright scripts plus optional with_server.py orchestration.'
license: Complete terms in LICENSE.txt
---

# Local Dev/Test Harness

Use this skill to run autonomous browser-based verification against a locally served application. It is stack-agnostic: the server command, cwd, port, dependency setup, and required smoke tests are declared in [`PROJECT.md`](../../../PROJECT.md) and refined by the active stack profile.

**Helper scripts available:**

- `scripts/with_server.py` — starts one or more servers, waits for ports, runs a command, and tears servers down.

Always run helper scripts with `--help` first. Treat bundled scripts as black boxes unless `--help` is insufficient for the task.

---

## Autonomous setup

Before writing or running Playwright automation:

1. Read `PROJECT.md` `dependency_policy`.
2. Run the declared install command in the declared package cwd when dependencies are missing.
3. Install browser automation tooling using the project-approved command. If `PROJECT.md` provides `browser_install_cmd`, use it. Otherwise use the active stack profile's example.
4. Dev/test tooling may be installed autonomously when `dependency_policy.may_add_dev_deps` permits it.
5. New **runtime** dependencies must be planned first, surfaced in `plan.md`, and verified through the expected lockfile/config diff.

Labelled generic setup example:

```bash
pip install playwright && playwright install chromium
```

If the project uses the Node Playwright runner, prefer the command declared in `PROJECT.md` or the active profile.

---

## Decision tree

```text
User task → Is it static HTML?
    ├─ Yes → Read the HTML file directly to identify selectors.
    │         ├─ Enough → Write Playwright script using selectors.
    │         └─ Not enough → Treat as dynamic.
    │
    └─ No, dynamic app → Is the server already running?
        ├─ No → Run: python scripts/with_server.py --help
        │        Then use the helper with PROJECT.md server commands.
        │
        └─ Yes → Reconnaissance then action:
            1. Navigate and wait for readiness.
            2. Capture screenshot, console, or rendered content.
            3. Identify semantic selectors.
            4. Execute actions and assertions.
```

---

## Using `with_server.py`

Run help first:

```bash
python .github/skills/app-testing/scripts/with_server.py --help
```

Labelled generic web example:

```bash
python .github/skills/app-testing/scripts/with_server.py \
  --server "npm run dev --prefix frontend" --port 3000 \
  -- python frontend/__tests__/e2e/smoke.py
```

Labelled Expo + Go example (only for projects whose stack profile and PROJECT.md declare these commands):

```bash
python .github/skills/app-testing/scripts/with_server.py \
  --server "cd backend && go run ./cmd/api" --port 8080 \
  --server "cd app && npx expo start --web --port 8081" --port 8081 \
  -- python app/__tests__/e2e/smoke.py
```

To create an automation script, include only Playwright logic; server lifecycle is handled by the helper:

```python
from playwright.sync_api import sync_playwright, expect

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://localhost:3000")
    page.wait_for_load_state("networkidle")
    expect(page.get_by_role("heading")).to_be_visible()
    browser.close()
```

---

## Playwright MCP (optional, for exploration)

The framework also ships a [Playwright MCP](../../../docs/mcp.md) server in [`.mcp.json`](../../../.mcp.json) for **agent-driven** browser control with persistent state and live page introspection.

Use MCP and this skill for different jobs:

- **This skill (Python + `with_server.py`)** is the default for **verification gates**: deterministic, committed, CI-runnable smoke tests that back `PROJECT.md`. Prefer it for anything a gate depends on — it is repeatable and token-efficient.
- **Playwright MCP** is for **exploration and self-healing loops**: debugging a flaky UI, discovering selectors, or reasoning iteratively over page structure during development.

Rule: explore with MCP, then **codify** the result as a deterministic script in this skill so a gate can run it unattended. Never leave a verification gate depending on an interactive MCP session. See [`docs/mcp.md`](../../../docs/mcp.md) for registration and security notes.

---

## Reconnaissance-then-action pattern

1. Navigate to the page and wait for the dynamic app to settle.
2. Inspect user-visible structure, not implementation-only internals:

```python
page.screenshot(path="artifacts/app-testing/inspect.png", full_page=True)
content = page.content()
buttons = page.get_by_role("button").all()
```

3. Prefer semantic locators: role, label, placeholder, text, test id when the project standard allows it.
4. Execute the flow and assert user-visible outcomes.
5. On failure, capture screenshot, console logs, network hints, and the last server log lines when available.

---

## Best practices

- Use `sync_playwright()` for small Python scripts unless the project standard says otherwise.
- Always close browsers.
- Wait for meaningful readiness: URL, role/text visibility, health endpoint, or `networkidle` when appropriate.
- Keep smoke tests short: setup → core action → assertion → cleanup.
- Use deterministic data and mocks for services outside the test boundary.
- Store artifacts in a project-local ignored directory such as `artifacts/app-testing/` or the path declared in `PROJECT.md`.
- Treat screenshots, traces, console logs, and reports as sensitive; do not publish them blindly.
- Prefer repeatable tests for gates and exploratory scripts for debugging.

---

## Common pitfalls

- Inspecting the DOM before the app has rendered.
- Using brittle CSS selectors when accessible roles or labels exist.
- Forgetting to install the browser binary before running scripts.
- Starting servers manually and leaving orphaned processes instead of using the helper.
- Adding runtime dependencies during test setup without a plan entry.

---

## Reference files

- `scripts/with_server.py` — server lifecycle helper.
- `examples/` — patterns for element discovery, static HTML automation, and console logging.
- [`docs/research/04-autonomous-dev-test-harness.md`](../../../docs/research/04-autonomous-dev-test-harness.md) — research notes behind this harness design.
