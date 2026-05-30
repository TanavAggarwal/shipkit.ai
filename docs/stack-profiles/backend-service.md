# Stack profile: backend-service

Profile for API, worker, CLI, integration, and service-only projects with no browser UI.

Commands shown here are examples; the authoritative commands live in [`../../PROJECT.md`](../../PROJECT.md).

## Forbidden APIs / patterns

- No string-built queries or command invocations with untrusted input.
- No secrets, credentials, tokens, private keys, or full sensitive personal data in logs.
- No unbounded request bodies, file uploads, queues, retries, or background jobs.
- No public interface change without validation, error shape, compatibility, and documentation updates.
- No real external network calls in tests unless the plan explicitly marks an integration gate and isolates credentials.

## Required test types

- Static checks, formatting, linting, and build gates from `PROJECT.md`.
- Unit tests for handlers, services, validators, adapters, schedulers, and pure logic.
- Integration tests for database/persistence, queues, external adapter boundaries, migrations, and public API contracts.
- Contract tests or golden fixtures for public request/response/event/file shapes.
- Security review for auth, authorization, input validation, rate limits, and secrets handling.

## Required E2E behavior

UI E2E is N/A. For end-to-end service changes, use the project's integration or smoke command to start local dependencies, seed deterministic data, call the public interface, assert the result, and tear down by PID or container lifecycle.

## Platform/accessibility checklist

- [ ] N/A for UI accessibility.
- [ ] Runtime targets and deployment assumptions are documented in `ARCHITECTURE.md`.
- [ ] Health checks, readiness, logging, metrics, and failure modes are considered for changed services.
- [ ] Migrations and rollback strategy are documented when persistence changes.

## Default verification gates (examples)

| Gate | Example command | Required when |
|---|---|---|
| install | `<package-manager install>` | Dependencies are absent or changed. |
| lint/static | `<lint command>` | Always for code changes. |
| build | `<build command>` | Always for deployable services. |
| unit | `<unit test command>` | Changed logic. |
| integration | `<integration test command>` | Persistence, public API, worker, queue, or adapter changes. |
| contract/smoke | `<service smoke command>` | Public interface or deployment-path changes. |

## When this profile is N/A

Use another profile when the project includes user-facing browser or native mobile UI that needs routing, accessibility, responsive, or browser automation rules.
