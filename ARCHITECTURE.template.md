# Architecture Template

Use this guide to grow `ARCHITECTURE.md` from the starter stub into the source of truth for system shape. Keep commands and package paths in `PROJECT.md`; keep platform rules in the active stack profile.

## Overview

Describe the product, primary users, major capabilities, and the boundaries of this repository.

Example:

- Product purpose:
- Primary users:
- Main workflows:
- Out of scope:

## Services

List each runtime component and its responsibility.

| Service/package | Responsibility | Runtime | Owner | Notes |
|---|---|---|---|---|
| `frontend` | Browser UI | Node/browser | TBD | Example only; replace with real package. |
| `api` | Public API | Server runtime | TBD | Example only. |

Include background jobs, CLIs, scheduled tasks, queues, and third-party integrations when present.

## Data model

Document persistent entities, ownership, lifecycle, privacy sensitivity, and migration rules.

| Entity | Stored in | Key fields | Sensitivity | Notes |
|---|---|---|---|---|
| User | TBD | id, email, role | Sensitive | Example only. |

Add diagrams or links to schema files when useful.

## API surface

Document public contracts: HTTP endpoints, RPC methods, events, webhooks, CLI commands, component APIs, or file formats.

| Interface | Method/event | Request/input | Response/output | Auth/permissions |
|---|---|---|---|---|
| `/api/example` | GET | query params | JSON | TBD |

For each new or changed public interface, include validation, error shape, idempotency, and compatibility notes.

## Repo layout

Explain the real directory structure so agents know where to look before editing.

```text
.
├── PROJECT.md                 # commands, packages, active profile
├── ARCHITECTURE.md            # system shape
├── docs/
│   └── stack-profiles/        # platform rules
└── <packages>/                # replace with actual packages
```

## Tech stack

Summarize major frameworks, languages, package managers, test tools, deployment targets, and observability tools. Keep exact commands in `PROJECT.md`.

| Area | Choice | Why | Notes |
|---|---|---|---|
| Frontend | TBD | TBD | Link to profile if relevant. |
| Backend | TBD | TBD |  |
| Testing | TBD | TBD |  |
| Deployment | TBD | TBD |  |

## Conventions

Record project-specific conventions that are not universal platform rules:

- Naming and directory conventions.
- Error handling and logging policy.
- Authentication and authorization patterns.
- Configuration and environment variable policy.
- Feature flag, migration, and rollback rules.
- Documentation requirements.

## Decision log

Add dated decisions when architecture changes materially.

| Date | Decision | Context | Consequences |
|---|---|---|---|
| YYYY-MM-DD | TBD | TBD | TBD |
