# Stack profile: web-spa

Default profile for browser-first applications such as Vite/React, Next.js-style single-page surfaces, dashboards, admin tools, and web SaaS products.

Use this as a checklist. Commands shown here are examples; the authoritative commands live in [`../../PROJECT.md`](../../PROJECT.md).

## Forbidden APIs / patterns

- No unguarded direct access to browser globals during render or module initialization when server rendering, prerendering, tests, or static analysis may execute the code. Guard `window`, `document`, `navigator`, `location`, `matchMedia`, and storage APIs behind runtime checks or framework-safe hooks.
- No secrets in client code, build-time public env vars, browser storage, screenshots, traces, logs, or source maps.
- No production network calls in tests. Mock API boundaries with test doubles, MSW, route interception, or local fixtures.
- No string-built HTML injection. Any rich HTML rendering requires sanitization and a plan-level security note.
- No broad `any`, blanket lint disables, ignored promise rejections, or unchecked async errors.
- No inaccessible click-only elements. Interactive UI must use semantic controls or equivalent ARIA and keyboard behavior.
- No fixed-width layouts that break below 320 px or require horizontal scrolling for primary flows.

## Required test types

- **Type/static checks:** run the typecheck and lint gates from `PROJECT.md`.
- **Unit tests:** changed utilities, hooks, state machines, validators, API clients, and data transforms.
- **Component tests:** changed reusable components and screens/routes with user-visible behavior. Prefer Testing Library queries by role, label, and text.
- **Integration tests:** API client boundaries, routing loaders/actions, auth/session flows, cache invalidation, persistence adapters, and error handling.
- **E2E smoke:** required for new or materially changed user flows at standard or complex scope.
- **Accessibility checks:** at minimum semantic labels, keyboard navigation, focus management, contrast-sensitive states, and error messages.

## Required E2E behavior

Use Playwright or the project's declared browser runner. New user-facing flows should include one short smoke that:

1. Boots the local app through the harness declared in `PROJECT.md` or [`app-testing`](../../.github/skills/app-testing/SKILL.md).
2. Navigates through the real route, not an implementation-only test hook.
3. Uses semantic locators (`getByRole`, labels, visible text) instead of brittle selectors.
4. Covers the happy path plus one important visible assertion: saved state, route change, validation message, or accessible UI state.
5. Mocks or seeds API data deterministically.
6. Captures traces/screenshots only on failure and treats artifacts as sensitive.

Default path convention: `frontend/__tests__/e2e/<slug>.py` for the bundled Python harness, or the package's existing Playwright test directory if one already exists. The plan decides the exact path.

## Routing, accessibility, and responsive checklist

- Routes are deep-linkable, refresh-safe, and have loading/error/empty states.
- Navigation uses the framework router, not ad hoc URL mutation.
- Page titles, landmarks, headings, labels, and focus order are intentional.
- All controls are keyboard reachable and show visible focus.
- Forms associate labels, descriptions, errors, and disabled/loading states.
- Layout works at 320 px, common tablet widths, and desktop widths up to 1920 px.
- Prefer fluid containers (`width: 100%`, `max-width`, CSS grid/flex) over fixed content widths.
- Avoid hover-only interactions; provide click/tap/keyboard equivalents.
- Respect reduced motion and color contrast needs.

## Browser storage and environment rules

- Store sensitive data only in server-managed sessions or secure, project-approved mechanisms. Do not put secrets or long-lived tokens in local storage.
- Namespaced browser storage keys must have a documented lifecycle and migration/clear strategy.
- Public client env vars must use the framework's explicit public prefix and must not contain secrets.
- Missing env vars fail fast with clear messages in development and tests.
- API base URLs and feature flags are read through a typed config module, not scattered literals.

## API mocking and data rules

- Tests mock network at the boundary closest to the user-visible behavior.
- Shared fixtures are minimal, named, and deterministic.
- Error states cover at least one server/client failure for changed critical flows.
- Do not couple tests to internal component structure when a user-visible assertion is possible.

## Default verification gates (examples)

Put the real commands in `PROJECT.md`. Common gates:

| Gate | Example command | Required when |
|---|---|---|
| install | `npm ci` | Dependencies are absent or changed. |
| typecheck | `npm run typecheck` | Always for code changes. |
| lint | `npm run lint` | Always for code changes. |
| unit/component | `npm test -- --ci` | Logic, hooks, components, routes. |
| build | `npm run build` | Build config, routing, env, bundle-sensitive changes. |
| e2e | `npm run e2e` or bundled `with_server.py` command | New or changed user-facing flows. |
| accessibility | project a11y script or Playwright assertions | New layouts, forms, dialogs, navigation. |

## When this profile is N/A

Use a different profile when:

- The project has no browser UI.
- The UI must share code with native mobile targets.
- The app is primarily a static content site with a separate publishing/accessibility profile.
- The project has a custom framework profile that supersedes these rules.
