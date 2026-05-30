---
project: my-app
description: One-line description.
target_platforms: [web]            # web | web,ios,android | backend-only
stack_profile: docs/stack-profiles/web-spa.md
packages:
  frontend: frontend               # or null
  backend: null                    # or path
dependency_policy:
  install_cmd: "npm ci"            # how agents install deps; cwd = package dir
  browser_install_cmd: "npx playwright install --with-deps chromium"
  may_add_runtime_deps: false      # if false, new runtime deps require a plan entry + user note
  may_add_dev_deps: true           # agents may add dev/test tooling (e.g. Playwright)
  lockfile: package-lock.json      # new deps must show up as a reviewable lockfile diff
verification_gates:
  - name: typecheck
    cwd: frontend
    command: "npm run typecheck"
    pass: "exit 0"
  - name: lint
    cwd: frontend
    command: "npm run lint"
    pass: "exit 0"
  - name: unit
    cwd: frontend
    command: "npm test -- --ci"
    pass: "exit 0"
  - name: coverage
    cwd: frontend
    command: "npm test -- --coverage --ci"
    pass: "lines >= 80%"        # full unit-test coverage on logic; raise as the project matures
  - name: e2e
    cwd: .
    command: "python .github/skills/app-testing/scripts/with_server.py --server \"npm run dev --prefix frontend\" --port 3000 -- python frontend/__tests__/e2e/smoke.py"
    pass: "exit 0"
---

# Project overview

This file is the first file to customize after cloning the template. It is authoritative for package locations, dependency installation, and verification commands. The default assumes a web single-page application in `frontend` using the [`web-spa`](docs/stack-profiles/web-spa.md) stack profile.

Replace `project`, `description`, package paths, and verification gates with your real project details before asking agents to implement production work.

# Target platforms

Default: `web`.

Use `target_platforms` to tell agents where the software must run:

- `[web]` for a browser application.
- `[web, ios, android]` for a shared cross-platform app.
- `[backend-only]` for an API, worker, CLI, or service without UI.

The selected `stack_profile` defines platform rules; this file defines commands and package paths.

# Stack & package locations

- `packages.frontend`: path to the frontend package, or `null` if none.
- `packages.backend`: path to the backend/service package, or `null` if none.
- `stack_profile`: markdown checklist under `docs/stack-profiles/` or your custom profile path.

Agents must read the active stack profile before planning UI, platform, storage, routing, accessibility, or E2E work.

# Precedence rules

1. `PROJECT.md` is **authoritative for commands and package locations**.
2. `ARCHITECTURE.md` is **authoritative for system shape**: services, data model, API surface, repo layout, and conventions.
3. Stack profiles provide **platform-specific rules and example gates only**.
4. **`PROJECT.md` is the only file required for first use.** `ARCHITECTURE.md` ships as a stub and can grow over time.

# Autonomous dependency policy

The YAML block is the source of truth.

- Agents run `dependency_policy.install_cmd` in each relevant package before verification when dependencies are missing.
- New runtime dependencies are not allowed by default. If needed, they must be named in `plan.md`, justified with alternatives considered, surfaced to the user, and persisted in the lockfile diff.
- Dev/test tooling may be added autonomously when `may_add_dev_deps: true`, especially local E2E tooling needed to run declared gates.
- Browser automation setup uses `browser_install_cmd` unless the active profile or package scripts declare a different command.
- Any dependency change must leave a reviewable manifest and lockfile diff.

# Local dev/test harness

Default web harness:

1. Install dependencies in `frontend` with `npm ci`.
2. Install the browser runtime with `npx playwright install --with-deps chromium`.
3. Start the app with `npm run dev --prefix frontend`.
4. Run the E2E gate through `.github/skills/app-testing/scripts/with_server.py` so the server lifecycle is explicit and cleaned up by PID.

If your project uses different commands, update the YAML block rather than editing agent prompts.

# Project-specific quality deltas

Add rules here that are specific to your project and not already covered by the active stack profile. Examples:

- Required browser support or accessibility conformance target.
- Coverage thresholds (the default `coverage` gate enforces ≥ 80% lines on logic; raise it per package as the project matures).
- API latency budgets.
- Data retention, privacy, or compliance constraints.
- Release, migration, or rollback requirements.
