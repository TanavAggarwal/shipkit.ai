# Autonomous dev/test harness patterns

Purpose: patterns for agents that can safely install dependencies, boot local servers, exercise apps with browser automation, review failures, patch code, and repeat until verification gates pass.

## Source landscape

| System/tool | Pattern to reuse | Source |
|---|---|---|
| OpenHands | Local GUI/SDK for coding agents; Docker/WSL requirements; `openhands serve --mount-cwd`; Docker app runs at localhost:3000; supports search config and code-focused SDK with planning, context compression, security analysis, agent-computer interfaces. | [Local setup](https://docs.openhands.dev/openhands/usage/run-openhands/local-setup), [SDK](https://docs.openhands.dev/sdk), [GitHub](https://github.com/All-Hands-AI/OpenHands) |
| SWE-agent | CLI initializes `SWEEnv`; deployment starts local/remote Docker; shell session executes commands; ACI tools installed; history compressed by `HistoryProcessor`; model output parsed and executed. | [Architecture](https://swe-agent.com/latest/background/architecture/) |
| mini-SWE-agent | Very small loop: bash-only, linear history, independent `subprocess.run` actions; easy sandbox swap (`docker exec`), local/docker/podman/singularity/bubblewrap environments. | [mini-SWE-agent](https://mini-swe-agent.com/latest/) |
| Playwright Test | E2E runner with assertions, isolation, parallelization, reports, traces, CI support; `webServer` can boot local app before tests. | [Intro](https://playwright.dev/docs/intro), [Config](https://playwright.dev/docs/test-configuration), [CI](https://playwright.dev/docs/ci-intro) |
| Playwright MCP | LLM browser control via accessibility snapshots, no vision model; navigate/click/fill/screenshots/network/storage; warns `browser_run_code_unsafe` is RCE-equivalent. | [Docs](https://playwright.dev/docs/getting-started-mcp), [GitHub](https://github.com/microsoft/playwright-mcp) |
| Browser Use | Python browser agent; can self-host; persistent browser CLI; custom tools; cloud option for scaling/stealth. | [GitHub](https://github.com/browser-use/browser-use) |

## Harness architecture

```text
Task/PRD
  -> Planner: derives verification gates + test strategy
  -> Implementer: edits code in workspace
  -> Runtime manager: installs deps, starts services, captures PIDs/logs/ports
  -> Test runner: unit/integration/E2E/static checks
  -> Browser driver: Playwright scripts or MCP/CLI snapshots
  -> Reviewer: reads diffs + test artifacts + logs; emits findings only
  -> Fix loop: implementer patches, reruns targeted gates, then full gates
  -> Audit: confirms claims match files/tests; writes handoff
```

## Minimum local runtime contract

- **Workspace mount**: project directory available read/write to implementer; reviewer read-only where possible.
- **Dependency policy**: allow ecosystem package managers (`npm`, `pnpm`, `pip`, `go`, `cargo`) but record every added dependency and why.
- **Server lifecycle**: start with explicit command, port, health URL, log file, timeout, and shutdown method by PID.
- **Environment file policy**: use `.env.example` and local `.env` placeholders; never invent real secrets.
- **Artifact capture**: stdout/stderr, screenshots, traces, videos, coverage, HTML reports, server logs.
- **Network policy**: package registries and public docs allowed; production customer endpoints denied unless explicit.
- **Isolation**: prefer containers/virtualenv/node_modules per repo; do not depend on global mutable state.
- **Repeatability**: every gate must be runnable by a human in CI or local shell.

## Verification gates

| Gate | Examples | Agent behavior |
|---|---|---|
| Static | typecheck, lint, formatting, vet | Run early; fix cheap errors before E2E. |
| Unit | pure functions, hooks, handlers | Add/adjust for changed logic; deterministic mocks. |
| Integration | DB/API/service boundaries | Start real local deps or test containers; seed data. |
| E2E | browser/user flows with Playwright | Boot app with `webServer`, run smoke path, capture trace on retry. |
| Security | dependency audit, secret scan, input validation review | Block high-risk changes; never log secrets. |
| Cross-platform | web + mobile emulator/simulator where available | At least static/platform-specific checks when devices unavailable. |
| Honesty | compare claimed work vs diffs/tests | Fail if tests were not run or results are overstated. |

## Playwright harness defaults

- Use `@playwright/test` for web E2E; it includes runner, assertions, isolation, parallelization, reports, and traces ([Playwright intro](https://playwright.dev/docs/intro)).
- Configure `webServer` so tests boot the local app and wait for readiness ([config](https://playwright.dev/docs/test-configuration)):

```ts
webServer: {
  command: 'npm run dev',
  url: 'http://localhost:3000',
  reuseExistingServer: !process.env.CI,
  timeout: 120_000,
}
```

- Use `baseURL`, role/text locators, web-first assertions, and traces on first retry.
- Store artifacts under test output directories ignored by git.
- In CI, install browsers with `npx playwright install --with-deps`, run tests, upload `playwright-report/` ([CI guide](https://playwright.dev/docs/ci-intro)).
- Treat Playwright reports/traces as sensitive; they may contain credentials, tokens, app data, or source snippets ([CI guide](https://playwright.dev/docs/ci-intro#properly-handling-secrets)).

## Browser automation for agents

| Mode | Best for | Trade-off |
|---|---|---|
| Playwright tests | Repeatable regression gates | Requires writing tests. |
| Playwright scripts | Debugging a specific flow | Less standardized unless saved. |
| Playwright MCP | Exploratory UI testing, accessibility-tree reasoning, persistent browser state | Verbose context; unsafe code tool is RCE-equivalent. |
| Playwright CLI/skills | Token-efficient coding-agent browser actions | Less rich than persistent MCP. |
| Browser Use | General web task automation and custom browser agents | More agentic; less deterministic than test runner. |

## Self-correction loop

1. Run baseline gates before edits when feasible; record pre-existing failures.
2. Implement smallest coherent change.
3. Run targeted static/unit tests.
4. If app/UI changed, start server and run Playwright smoke.
5. On failure, classify: code bug, test bug, env/dependency, flaky timing, missing seed data.
6. Fix one class at a time; rerun the narrowest failing gate.
7. After narrow gates pass, run full declared gates.
8. Reviewer reads diff and artifacts blind; emits only material findings.
9. Implementer fixes findings; loop until approved.
10. Honesty audit verifies final summary against actual commands and outputs.

## Dependency installation safety

- Prefer project-native commands: `npm install`, `pnpm add`, `npx expo install`, `pip install -r`, `go mod tidy`.
- Pin or lock dependencies via existing lockfile conventions.
- Avoid global installs unless tool explicitly requires it; prefer `npx`, `uvx`, local venv, or project devDependency.
- Do not run install scripts from unknown repos without need; inspect package popularity/maintenance for risky dependencies.
- Record why a new dependency is needed and alternatives considered.
- Re-run lockfile-aware install after manifest edits to persist exact versions.

## Sandboxing patterns

| Pattern | Use | Notes |
|---|---|---|
| Local process | Fast iteration | Least isolation; good for trusted repo. |
| Virtualenv/node_modules | Language isolation | Cheap; still shares OS. |
| Docker dev container | Default for publishable framework | Reproducible, resource limits, service composition. |
| Docker-in-Docker / socket mount | Agents that launch sandboxes | Powerful but high risk; OpenHands Docker command mounts Docker socket ([local setup](https://docs.openhands.dev/openhands/usage/run-openhands/local-setup)). |
| Remote sandbox | Untrusted code or scale | More setup; easier quotas and cleanup. |
| Read-only review agent | Code review/audit phase | Enforces separation of implement/review. |

## Server lifecycle checklist

- Pick deterministic port or discover free port and write it to state.
- Start server in background with captured PID.
- Wait for health endpoint or HTTP 200, not fixed sleeps.
- Stream/store logs for later review.
- On failure, show last N log lines and exit code.
- Stop by PID at end; avoid name-based process killing.
- For multi-service stacks, use compose profiles or a wrapper script with teardown.

## Test design for autonomous agents

- Tests should encode user-visible behavior, not implementation details.
- Prefer semantic locators (`getByRole`, labels, text) over brittle CSS.
- Seed deterministic data; mock network calls not under test.
- Use fake timers where time matters.
- Make smoke tests short: login/setup -> core action -> assertion -> cleanup.
- Add screenshots/traces only on failure to save time and storage.
- Separate “required gates” from “nice-to-run gates” for slow suites.

## Lessons for our framework

1. Provide a built-in **runtime manager** abstraction: `install`, `start`, `healthcheck`, `test`, `stop`, `collect_artifacts`.
2. Ship templates with **Playwright configured with `webServer`**, traces, HTML report, and CI workflow.
3. Include both **deterministic E2E tests** and an **exploratory browser-agent mode**; use tests for gates, MCP/Browser Use for debugging.
4. Make phase gates physical: implementer can edit; reviewer/auditor should be read-only to avoid self-approval.
5. Store command history and artifacts so honesty audits can verify “what actually ran”.
6. Prefer mini-SWE-agent’s simple linear trajectory for debuggability, but keep optional richer tools for large repos.
7. Use containers for untrusted execution, but document the Docker socket risk and offer safer local-process mode for trusted repos.
8. Define verification gates in the plan before coding; agents should not invent “done” after implementation.
9. Treat Playwright reports/traces/logs as sensitive artifacts and avoid publishing them blindly.
10. Publish starter apps with one-command local dev (`npm run dev`), one-command tests (`npm test`, `npm run e2e`), and CI parity.
