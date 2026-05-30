---
name: plan-task
description: 'Step 2 of the generic multi-agent workflow. Produces an 11-section, reviewable implementation plan before code is written, using PROJECT.md for package locations, dependency policy, and verification gates; ARCHITECTURE.md for system shape; and the active stack profile for platform rules. USE FOR: any non-trivial feature, fix, refactor, public interface change, data-model change, UI screen/route, integration, or dependency change. DO NOT USE FOR: trivial edits or pure Q&A. OUTPUT: `.dev/tasks/<slug>/plan.md` plus `state.verification_gates`, ready for review-changes in plan-review mode. NEVER writes product code.'
---

# SKILL: plan-task

**When to use:** Step 2 of [`development-workflow`](../development-workflow/SKILL.md), after intent is understood and before any code edits.

**Goal:** produce a written plan that makes implementation mechanical: files, interfaces, data changes, tests, docs, risks, and executable verification gates.

---

## Required inputs

Before planning, confirm you have:

1. Problem statement from `.dev/tasks/<slug>/prd.md`.
2. Scope boundaries and non-goals.
3. Target platforms from [`PROJECT.md`](../../../PROJECT.md) and the PRD.
4. Acceptance criteria.
5. Package locations, dependency policy, and baseline gates from `PROJECT.md`.
6. System shape from [`ARCHITECTURE.md`](../../../ARCHITECTURE.md).
7. Active stack profile from `PROJECT.md` (for example, [`docs/stack-profiles/web-spa.md`](../../../docs/stack-profiles/web-spa.md)).

If any item is unclear, return to Step 1 and ask. Do not guess.

---

## Procedure

### A. Read before planning

Read:

1. [`PROJECT.md`](../../../PROJECT.md) — authoritative commands, package paths, dependency policy, verification gates.
2. [`ARCHITECTURE.md`](../../../ARCHITECTURE.md) — services, data model, public interfaces, repo layout, conventions.
3. The active stack profile declared by `PROJECT.md`.
4. Relevant source files and tests.
5. Relevant docs such as feature READMEs, API docs, schema/model definitions, environment examples, or design notes.

Use the repository's existing search and code-intelligence tools. Plan from actual code, not assumptions.

### B. Draft the plan

Write a single markdown plan with exactly these 11 sections. Every section is mandatory; use `N/A — <reason>` when not applicable.

```markdown
## Plan: <short title>

### 1. Summary
One paragraph: what changes and why.

### 2. Affected areas
- Packages/modules: <paths from PROJECT.md>
- Runtime surfaces: <frontend/backend/worker/CLI/library/etc.>
- Shared contracts: <public API, data model, events, config, generated types>

### 3. Data model / schema changes
- Models/tables/documents/messages added or modified: <field-level detail>
- Migration/backfill strategy: <how data changes safely, or N/A>
- Compatibility: <breaking changes, versioning, rollback>
- Generated artifacts: <what must be regenerated, per active stack/profile>

### 4. Public interface / API changes
For each public interface change:
- Interface type: <HTTP route, RPC method, CLI command, SDK function, event, UI route, config key>
- Request/input shape and validation rules
- Response/output shape and error cases
- Authn/authz or permission requirements
- Idempotency, rate limits, caching, and compatibility notes

### 5. Files to create or modify
Flat list. Every file gets one line.
- `path/to/file` — <create|modify> — <purpose>

### 6. Platform considerations — active stack profile
- Active profile: <path from PROJECT.md>
- Profile checklist items that apply: <accessibility, responsive, native/mobile/desktop/browser, storage, routing, env, etc.>
- Cross-platform rules: <apply only when the active profile requires them; otherwise N/A>
- Platform-specific files/adapters needed: <list or N/A>
- Manual/device/browser validation required: <what evidence>

### 7. Tests
List tests by category according to PROJECT.md verification gates and active profile.

#### 7.a Unit tests
- Files + cases:
- Determinism/mocking strategy:

#### 7.b Integration tests
- Boundary under test:
- Local dependencies or test containers:
- Files + cases:
- Required when: <data model, public interface, workflow, external integration, or profile-specific rule>

#### 7.c End-to-end / smoke tests
- User-visible flow:
- Harness: <app-testing, project runner, browser/device tool, or N/A>
- Artifact path:
- Evidence captured:

#### 7.d Other gates
- Type/static analysis:
- Security/dependency checks:
- Coverage thresholds, if any:

### 8. Documentation updates
- `ARCHITECTURE.md`: <sections to update or N/A>
- Feature/package README: <path or N/A>
- `.env.example` / config docs: <new vars/keys or N/A>
- API/public contract docs: <path or N/A>
- Research/PRD links: <if useful>

### 9. Security review
- Inputs validated where:
- Secrets/PII touched and protections:
- Authn/authz impact:
- Injection/file upload/shell/network risks:
- Dependency risk and lockfile expectations:
- OWASP categories considered:

### 10. Rollout & risk
- Safe order of changes:
- Rollback plan:
- Migration/deployment notes:
- Known risks and mitigations:
- Open questions: <must be empty before plan approval>

### 11. Verification gates
Pull the baseline list from PROJECT.md `verification_gates`. Add task-specific gates as needed. Each gate must include name, cwd, command, pass condition, and source.

#### 11.a Baseline from PROJECT.md
- `<gate name>` — cwd `<cwd>` — command `<command from PROJECT.md>` — pass: `<pass>` — source: baseline

#### 11.b Task-specific
- `<gate name>` — cwd `<cwd>` — command `<exact command>` — pass: `<condition>` — source: task

If a gate is intentionally skipped, include the approved skip reason. Do not silently drop gates.
```

Concrete command examples belong in [`PROJECT.md`](../../../PROJECT.md) or the active stack profile, not in this skill.

### C. Self-check

Before Step 3, verify:

- [ ] All 11 sections are present.
- [ ] Every file in §5 has a purpose.
- [ ] Any data model or public interface change has tests and docs.
- [ ] §6 references the active stack profile and marks non-applicable rules as `N/A` with reasons.
- [ ] §7 covers unit, integration, e2e/smoke, and other gates as required by `PROJECT.md`.
- [ ] §8 includes all promised docs/config examples.
- [ ] §10 has no unanswered open questions.
- [ ] §11 copies baseline gates from `PROJECT.md` and adds concrete task-specific gates.
- [ ] New runtime dependencies are explicitly planned and surfaced per `PROJECT.md` dependency policy.

### D. Persist

1. Write the plan to `.dev/tasks/<slug>/plan.md` via [`task-state`](../task-state/SKILL.md).
2. Copy §11 gates into `state.verification_gates` as `{ name, cmd, cwd, expect, source }`.
3. Set `state.phase = "plan-review"`.
4. Append a journal entry and regenerate `STATUS.md`.

---

## Output

Return the plan in chat and persist it on disk. Do not write product code in this step.
