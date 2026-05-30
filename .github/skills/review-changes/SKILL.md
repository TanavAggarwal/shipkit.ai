---
name: review-changes
description: 'Steps 3 and 5 of the generic multi-agent workflow. Reviews either a plan or implemented code against PROJECT.md, ARCHITECTURE.md, the active stack profile, security expectations, test strategy, documentation, and scope discipline. In code-review mode it runs the verification gates stored in task state and applies three blind personas: stack-profile/platform, security, and test-quality. USE FOR: plan approval before implementation and implementation review before honesty audit. DO NOT USE FOR: writing plans, editing code, or replacing honesty-audit. OUTPUT: explicit APPROVED or CHANGES REQUESTED verdict, gate evidence, persona findings, and persisted updates to task state.'
---

# SKILL: review-changes

Use in two modes:

- **Plan-review mode** — Step 3, before code exists.
- **Code-review mode** — Step 5, after implementation, looping with Step 4 until clean.

Both modes review against [`PROJECT.md`](../../../PROJECT.md), [`ARCHITECTURE.md`](../../../ARCHITECTURE.md), the active stack profile, and the approved task plan.

---

## Mode 1 — Plan Review (Step 3)

Review `.dev/tasks/<slug>/plan.md`. No product code should have been written yet.

### Plan-review checklist

#### Completeness
- [ ] All 11 sections from [`plan-task`](../plan-task/SKILL.md) are present and non-empty.
- [ ] Every file in §5 has a stated purpose.
- [ ] Every data model or public interface change has matching tests and docs.
- [ ] §11 includes executable gates from `PROJECT.md` plus task-specific gates, each with cwd and pass condition.
- [ ] No `TBD` or unanswered open questions remain.

#### Project configuration
- [ ] Package paths and commands come from `PROJECT.md`.
- [ ] The active stack profile is named and applied only where relevant.
- [ ] New runtime dependencies follow `PROJECT.md` dependency policy and are surfaced in the plan.
- [ ] Dev/test tooling additions are justified and lockfile behavior is stated when applicable.

#### Platform/profile
- [ ] Profile-specific forbidden APIs, accessibility, routing, storage, environment, and responsive requirements are addressed.
- [ ] Cross-platform checklist is applied when the active profile targets multiple platforms; otherwise it is marked `N/A` with a reason.
- [ ] Required e2e/manual/device/browser evidence is planned.

#### Data and backend/service boundaries
- [ ] Data changes include migration, compatibility, and rollback notes.
- [ ] Public interfaces define inputs, outputs, errors, auth/permissions, and compatibility.
- [ ] External I/O is testable via mocks, fakes, local services, or documented harnesses.

#### Testing
- [ ] Unit, integration, e2e/smoke, static, security, and coverage gates are listed as required by `PROJECT.md` and profile.
- [ ] Tests are deterministic: no real production network calls, uncontrolled timers, or unseeded data.
- [ ] Edge cases cover empty/invalid input, auth/permission failure, unavailable dependencies, concurrency where relevant, and rollback paths.

#### Security
- [ ] User input is validated at trust boundaries.
- [ ] No secrets, tokens, credentials, or full PII are logged.
- [ ] Database access uses parameterized queries or safe ORM/query-builder patterns; no string-built queries from user input.
- [ ] No shell command construction from user input.
- [ ] File uploads or generated files validate type, size, path, and storage location.

#### Documentation and scope
- [ ] `ARCHITECTURE.md`, feature/package docs, API docs, and config examples are updated when affected.
- [ ] New code has a clear module/directory home and single-responsibility decomposition; no unrelated concerns are merged into one file and no god/utility dumping ground is introduced.
- [ ] No unrelated refactors or speculative features are included.
- [ ] Unrelated bugs are listed for follow-up, not fixed in this task.

### Verdict

- **APPROVED** — set `state.phase = "plan-approved"`, append a journal entry, regenerate `STATUS.md`, and proceed to Step 4.
- **CHANGES REQUESTED** — list every issue with plan section and concrete fix. The plan author updates and reruns review.

---

## Mode 2 — Code Review (Step 5)

Review actual code changes against the approved plan. Run real verification commands; do not rely on visual inspection alone.

### Step 2A — Run verification gates

Execute every command in `state.verification_gates`:

1. Baseline gates (`source = "baseline"`) first.
2. Task-specific gates (`source = "task"`) second.

For each gate capture:

- name
- exact command
- cwd
- exit code, or approved skip reason
- last approximately 40 lines of output
- pass/fail against `expect`
- source

Persist:

```jsonc
{
  "ran_at": "<iso-now>",
  "results": [
    { "name": "lint", "cmd": "<cmd from PROJECT.md>", "cwd": "<cwd>", "exit": 0, "tail": "...", "source": "baseline" }
  ],
  "all_passed": true,
  "baseline_passed": true
}
```

Failure rules:

- Failed baseline gate → automatic `CHANGES REQUESTED`.
- Failed task gate → automatic `CHANGES REQUESTED`.
- Approved skips count as passing only when the skip reason appears in the approved plan.
- Personas still run after command failures to surface additional issues.

### Step 2B — Three blind personas

Run three independent reviews against the diff. Do not show one persona another persona's output until all are complete.

All three personas run for `simple`, `standard`, and `complex` tasks. `trivial` tasks get one stack-profile/platform pass.

#### Persona 1 — Stack-profile / platform reviewer

Scope: active stack profile declared in [`PROJECT.md`](../../../PROJECT.md), plus affected platform targets.

Checklist:
- [ ] Diff follows the active profile's forbidden APIs and required abstractions.
- [ ] Accessibility, responsive behavior, routing/navigation, storage, environment access, and asset handling follow the profile.
- [ ] Cross-platform rules are enforced only when the profile requires them; otherwise verdict can be `NA`.
- [ ] Platform-specific files/adapters are thin and documented.
- [ ] Required browser/device/manual evidence from the plan is present.
- [ ] User-facing UI changes include appropriate e2e/smoke coverage per `PROJECT.md`.

Verdict: `APPROVED | CHANGES_REQUESTED | NA`.

#### Persona 2 — Security reviewer

Scope: OWASP awareness, trust boundaries, secrets/PII, auth/permissions, data access, dependency risk.

Checklist:
- [ ] Every new public interface has appropriate authn/authz or a documented reason none is needed.
- [ ] All user-controlled input is validated for type, size, range, format, and business invariants.
- [ ] No secrets, tokens, credentials, password hashes, or full PII are logged or exposed.
- [ ] Database queries are parameterized or use safe data-access abstractions; no string-built queries from user input.
- [ ] Shell/process invocation does not concatenate user input into commands.
- [ ] File paths, uploads, generated artifacts, and downloads are constrained and validated.
- [ ] New dependencies are planned, locked, and justified.
- [ ] No new attack surface lacks a mitigation.

Verdict: `APPROVED | CHANGES_REQUESTED`.

#### Persona 3 — Test-quality reviewer

Scope: plan adherence, deterministic tests, gate evidence, docs, and scope discipline.

Checklist:
- [ ] Every planned file was created or modified as described.
- [ ] No unplanned product file was modified.
- [ ] No new dependency was added outside the approved plan.
- [ ] Unit/integration/e2e/static/security gates required by `PROJECT.md` ran or have approved skips.
- [ ] Tests assert user-visible behavior or public contracts, not brittle implementation details.
- [ ] Tests are deterministic: no real production network, no uncontrolled time, no order dependence.
- [ ] Every new or changed unit of non-trivial logic has a unit test; line coverage meets the `PROJECT.md` threshold (default ≥ 80% on logic) or has an approved exception, supported by captured tool output.
- [ ] Code is modular: each changed file has a single clear responsibility, layers (UI/logic/data) stay separated, and no god file or circular import is introduced.
- [ ] Coverage thresholds or quality targets in the plan are supported by captured tool output.
- [ ] No dead code, commented-out code, copy-paste duplication, placeholder names, stray debug prints in production paths, or unowned TODOs.
- [ ] No language-specific lint/type suppression appears without inline justification.
- [ ] Documentation and config examples promised in the plan are in the diff.
- [ ] Scope did not expand into unrelated cleanup or speculative features.

Verdict: `APPROVED | CHANGES_REQUESTED`.

### Step 2C — Merge and final verdict

Write:

```jsonc
{ "stack_profile": "APPROVED", "security": "APPROVED", "test_quality": "CHANGES_REQUESTED" }
```

Final verdict:

- **APPROVED** iff every non-`NA` persona approves, `baseline_passed == true`, and `all_passed == true`. Set `state.phase = "audit"` and hand off to [`honesty-audit`](../honesty-audit/SKILL.md).
- **CHANGES REQUESTED** otherwise. Set `state.phase = "review-loop"`, increment `state.attempt`, append findings to `journal.md`, regenerate `STATUS.md`, and return to Step 4.

---

## Output format

### Plan-review mode

```markdown
### Review (plan-review) — <task title>

**Verdict:** <APPROVED | CHANGES REQUESTED>

**Findings:**
1. <blocker|major|minor> — <plan section> — <issue> — <fix>

**Notes:** <anything the user should know>
```

### Code-review mode

```markdown
### Review (code-review) — <task title> — attempt <n>

**Verdict:** <APPROVED | CHANGES REQUESTED>

**Verification gates:**
- `<cmd>` (cwd `<dir>`) → exit <code> — <pass|fail vs expect=...>

**Persona verdicts:**
- stack-profile/platform: <APPROVED | CHANGES_REQUESTED | NA>
- security: <APPROVED | CHANGES_REQUESTED>
- test-quality: <APPROVED | CHANGES_REQUESTED>

**Findings:**
1. <blocker|major|minor> — [persona] — `<file:line>` — <issue> — <fix>
```

Be strict on blockers and never approve with a failed gate or unresolved material finding.
