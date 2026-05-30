# Multi-agent Framework — Copilot Project Instructions

**Always read this file at the start of every session.** It defines the rules every coding agent must follow in repositories using this template. For project-specific commands and package locations, read [`PROJECT.md`](../PROJECT.md). For system shape, read [`ARCHITECTURE.md`](../ARCHITECTURE.md). For platform rules, read the active stack profile declared in `PROJECT.md`.

---

## 0. Using this template

This repository is a reusable multi-agent "idea → research → PRD → plan → build → review → ship" starter. The framework is intentionally generic: your project's specifics live in committed project files, not in agent prompts.

### Precedence rules

1. [`PROJECT.md`](../PROJECT.md) is **authoritative for commands and package locations**: package paths, dependency policy, and verification gates.
2. [`ARCHITECTURE.md`](../ARCHITECTURE.md) is **authoritative for system shape**: services, data model, API surface, repo layout, and conventions.
3. Stack profiles under [`docs/stack-profiles/`](../docs/stack-profiles/) provide **platform-specific rules and example gates only**. Follow the active profile declared in `PROJECT.md`.
4. **`PROJECT.md` is the only file required for first use.** `ARCHITECTURE.md` ships as a starter stub and can grow from [`ARCHITECTURE.template.md`](../ARCHITECTURE.template.md).

---

## 1. Project overview

This is an LLM-driven codebase: most changes may be written by AI agents. Therefore every change must be **explicit, verifiable, well-tested, and documented**. Implicit assumptions are not allowed.

Agents must derive project facts from `PROJECT.md`, `ARCHITECTURE.md`, and the active stack profile instead of hardcoding a stack, package layout, or command set.

---

## 2. Stack-profile rules

Every code change must comply with the active stack profile declared in `PROJECT.md`.

- Web apps follow [`docs/stack-profiles/web-spa.md`](../docs/stack-profiles/web-spa.md) unless `PROJECT.md` says otherwise.
- Cross-platform apps follow the profile selected by `PROJECT.md`; if it targets web + mobile from one codebase, the strict cross-platform checklist lives in [`docs/stack-profiles/expo-cross-platform.md`](../docs/stack-profiles/expo-cross-platform.md).
- Service-only projects follow [`docs/stack-profiles/backend-service.md`](../docs/stack-profiles/backend-service.md) unless a custom profile is declared.

If the profile and `PROJECT.md` disagree about commands or package paths, `PROJECT.md` wins. If they disagree about platform safety rules, stop and surface the conflict in the plan.

---

## 3. Quality bar

### Documentation

- Any change to system shape, service boundaries, data model, or public API must update [`ARCHITECTURE.md`](../ARCHITECTURE.md).
- Any new environment variable must update the relevant `.env.example` file named in `PROJECT.md` or the affected package.
- Any new user-facing feature should include the feature documentation required by the active stack profile or project conventions.
- Documentation changes must be committed in the same change as the code they describe.

### Testing

- Verification gates come from `PROJECT.md` first, then task-specific gates added by the plan.
- Unit tests cover changed pure logic, components, handlers, workers, adapters, and edge cases.
- Integration tests are required for changed persistence, service boundaries, public APIs, external adapters, or migrations.
- UI changes at standard or complex scope require the E2E smoke behavior required by the active stack profile, usually via [`app-testing`](skills/app-testing/SKILL.md).
- Tests must be deterministic: no real production network calls, no uncontrolled timers, and no hidden dependency on global machine state.

### Type safety & style

- Run the typecheck, lint, formatting, build, and test gates declared in `PROJECT.md`.
- Do not add broad escape hatches such as untyped values, ignored errors, or lint suppressions without an inline reason and a plan entry.
- Prefer ecosystem-native refactoring and package tools over manual edits when they reduce risk.

### Security

- Validate all user input at trust boundaries.
- Never log secrets, credentials, tokens, private keys, or full sensitive personal data.
- Use parameterized queries or safe data-access APIs; never build queries by string concatenation.
- File uploads must validate type, size, and authorization before storage.
- Treat test traces, screenshots, server logs, and browser storage captures as potentially sensitive.

---

## 4. Development workflow

Every non-trivial change follows the autonomous-capable workflow defined in [`development-workflow`](skills/development-workflow/SKILL.md), which dispatches to:

0. Optional [`research`](agents/research.agent.md) front-end — turn a raw idea into a cited dossier in `docs/research/<slug>.md` using [`deep-research`](skills/deep-research/SKILL.md).
1. [`task-state`](skills/task-state/SKILL.md) — persist state to `.dev/tasks/<slug>/` so tasks survive session boundaries.
2. [`complexity-classify`](skills/complexity-classify/SKILL.md) — route trivial / simple / standard / complex.
3. [`plan-task`](skills/plan-task/SKILL.md) — produce the 11-section plan with verification gates pulled from `PROJECT.md`.
4. [`review-changes`](skills/review-changes/SKILL.md) — plan-review and code-review with three blind personas plus verification gates.
5. [`honesty-audit`](skills/honesty-audit/SKILL.md) — anti-sycophancy gate before a task may be marked DONE.

The loop is: `research (optional) → prd → Step 0 (resume │ PRD intake) → 1 → 1.5 → 2 → 3 → 4 ↔ 5 (until APPROVED) → 5.5 → 6 (hand-off)`. Every non-trivial task starts from **either** existing state on disk **or** a freshly imported PRD at `.dev/tasks/<slug>/prd.md`.

### Phase-gated agents

The workflow is packaged as custom agents under [`agents/`](agents/) so phase boundaries are enforced by the **available tool set**, not just instructions:

| Agent | Tools | Phase |
|---|---|---|
| [`research`](agents/research.agent.md) | search + fetch + edit restricted to research docs | Pre-PRD: source-grounded brainstorming and research dossier. Hands off to `prd`. |
| [`prd`](agents/prd.agent.md) | edit + search + fetch (no app-code edits) | PRD authoring via [`doc-coauthoring`](skills/doc-coauthoring/SKILL.md). Hands off to `dev`. |
| [`dev`](agents/dev.agent.md) | full | Steps 0–3: orchestration, PRD intake, classification, planning, plan-review. Hands off to `dev-impl`. |
| [`dev-impl`](agents/dev-impl.agent.md) | edit + search + terminal | Step 4: implementation only. Hands off to `dev-review`. |
| [`dev-review`](agents/dev-review.agent.md) | **no edit** — read + terminal only | Steps 5 + 5.5: gates, blind personas, honesty audit. Hands findings back to `dev-impl`. |

The absence of edit tools in `dev-review` is the gate: the reviewer physically cannot fix what they review. The typical chain is `research → prd → dev → dev-impl ↔ dev-review`. If you already have a PRD, start with `dev`; if you only have a product idea, start with `research` or `prd` depending on whether web-grounded research is needed.

---

## 5. Operating principles for agents

- **Resume or intake before starting.** At the start of every session, list `.dev/tasks/`. If the prompt looks like a continuation, load that task's `state.json` first via [`task-state`](skills/task-state/SKILL.md) and run PRD sync. Otherwise, import a PRD from the user's path, URL, one-line brief, or `docs/prds/<slug>.md` into `.dev/tasks/<slug>/prd.md`.
- **Be explicit.** State assumptions and promote uncertainty into PRD open questions or plan risks.
- **Read before writing.** Read `PROJECT.md`, `ARCHITECTURE.md`, the active stack profile, and every file you edit before making a non-trivial change.
- **Small, reviewable changes.** Prefer narrow diffs and clear checkpoints over sprawling edits.
- **No silent scope creep.** If you spot unrelated bugs, list them as follow-ups instead of fixing them in the same change.
- **No dead code, no commented-out code, no ownerless TODOs.**
- **Update docs in the same change as code.** System shape → `ARCHITECTURE.md`; command/package/profile changes → `PROJECT.md`; feature docs → the location required by the active stack profile.

---

## 6. Memory protocol

The agent framework has three memory tiers. Use the right one:

| Tier | Path | Holds | Lifetime |
|---|---|---|---|
| **Episodic** (per task) | `.dev/tasks/<slug>/` (git-ignored) | `prd.md`, `state.json`, `plan.md`, `STATUS.md`, `journal.md` for the in-flight task | Until the developer deletes it after merge |
| **Procedural** (per repo) | `/memories/repo/` | Verified commands, build invocations, working test setups, repo-specific gotchas | Survives across tasks in this repo |
| **Semantic** (cross-repo) | `/memories/` (user) | Cross-project patterns with no repo-specific secrets or proprietary details | Survives across all repos |

Rules:

- Per-task state is owned by [`task-state`](skills/task-state/SKILL.md). Do not write task state into `/memories/`.
- Procedural notes go in `/memories/repo/` only after a command/setup has been verified to work in this repo.
- Semantic notes in `/memories/` must not contain repo-specific confidential information or secrets.
- When a memory turns out to be wrong, update or delete it in the same change that uncovers the mistake.
