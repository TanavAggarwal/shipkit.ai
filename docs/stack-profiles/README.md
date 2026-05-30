# Stack profiles

Stack profiles are checklist-style rule packs for platform behavior, testing expectations, and example verification gates. They keep the agent framework generic while still giving implementers concrete guardrails.

`PROJECT.md` selects the active profile with:

```yaml
stack_profile: docs/stack-profiles/web-spa.md
```

## Precedence

1. `PROJECT.md` is authoritative for package locations and commands.
2. `ARCHITECTURE.md` is authoritative for system shape.
3. Stack profiles provide platform-specific rules and example gates only.

If a profile's example command conflicts with `PROJECT.md`, use `PROJECT.md`.

## Decision table

| Building... | Use profile | Notes |
|---|---|---|
| A browser app, dashboard, SaaS UI, marketing app, or web-only product | [`web-spa.md`](web-spa.md) | Default. Works for Vite/React or Next.js-style apps. |
| One app that must run on web + iOS + Android | [`expo-cross-platform.md`](expo-cross-platform.md) | Strict shared-code and platform-parity checklist. |
| An API, worker, CLI, integration service, or backend-only package | [`backend-service.md`](backend-service.md) | No UI rules; focuses on service tests and contracts. |

## How to add a custom profile

1. Copy the closest existing profile.
2. Keep the checklist headings: Forbidden APIs, Required test types, Required E2E behavior, Platform/accessibility checklist, Default verification gates, When N/A.
3. Mark every command as an example. Put real commands in `PROJECT.md`.
4. Add any stack-specific dependency, routing, storage, environment, and security rules.
5. Update `PROJECT.md` to point `stack_profile` at the new file.
6. Add task-specific gates in plans when a change exceeds the default profile.

Profiles should be opinionated enough that a reviewer can block unsafe code without re-litigating the stack on every task.
