---
name: development-workflow
description: 'MASTER PROCESS for a generic, publishable, PRD-driven multi-agent idea-to-ship workflow. Coordinates resumable task state, optional web-grounded research, complexity classification, planning, plan review, implementation, verification, blind code review, honesty audit, and hand-off. Stack-specific package locations and commands come from PROJECT.md, ARCHITECTURE.md, and the active stack profile rather than this skill. USE FOR: any non-trivial feature, fix, refactor, public interface change, data-model change, UI screen/route, dependency change, or continuation of an existing task. DO NOT USE FOR: pure Q&A, read-only investigation, or isolated typo fixes. OUTPUT: enforces `(research?) → (0a resume | 0b PRD intake | 0c research intake) → 1 understand → 1.5 classify → 2 plan → 3 plan-review → 4 implement ↔ 5 code-review → 5.5 honesty-audit → 6 hand-off`, persisted under `.dev/tasks/<slug>/`.'
---

# SKILL: development-workflow

This is the agent framework's master development process. It is **stack-agnostic** and **autonomous-capable**: task state is persisted to `.dev/tasks/<slug>/` so work survives session boundaries, rate-limit interruptions, and agent handoffs.

Project-specific facts live outside this skill:

1. [`PROJECT.md`](../../../PROJECT.md) — authoritative for package locations, dependency policy, and verification gates.
2. [`ARCHITECTURE.md`](../../../ARCHITECTURE.md) — authoritative for system shape: services, data model, public interfaces, repo layout, and conventions.
3. The active stack profile declared by `PROJECT.md` — platform-specific rules and example gates only.

## Skills this dispatches to

- [`deep-research`](../deep-research/SKILL.md) — optional web-grounded research and brainstorming before PRD writing.
- [`task-state`](../task-state/SKILL.md) — state I/O, PRD intake/sync, status, and journal.
- [`complexity-classify`](../complexity-classify/SKILL.md) — Step 1.5; writes `state.complexity`.
- [`plan-task`](../plan-task/SKILL.md) — Step 2; writes the 11-section implementation plan.
- [`review-changes`](../review-changes/SKILL.md) — Steps 3 and 5; plan review and code review.
- [`honesty-audit`](../honesty-audit/SKILL.md) — Step 5.5; final evidence gate.

## Phase-gated agents (recommended)

The workflow is best driven through custom agents under [`.github/agents/`](../../agents/) so phase boundaries are enforced by tool availability:

- [`research`](../../agents/research.agent.md) — optional front-end for raw ideas; writes only `docs/research/*.md`.
- [`prd`](../../agents/prd.agent.md) — turns a brief or research dossier into a PRD.
- [`dev`](../../agents/dev.agent.md) — entry point for Steps 0–3 and orchestration.
- [`dev-impl`](../../agents/dev-impl.agent.md) — edit-capable implementer for Step 4.
- [`dev-review`](../../agents/dev-review.agent.md) — read-only reviewer/auditor for Steps 5 and 5.5.

This skill is the source of truth for the *steps and exit conditions*. Agent files define who holds which tools.

## RARV loop

Steps 4–5 run as **R**eason → **A**ct → **R**eflect → **V**erify:

1. Reason from `PROJECT.md`, `ARCHITECTURE.md`, active profile, `state.json`, approved `plan.md`, and the diff.
2. Act by making only the planned edits.
3. Reflect by appending `journal.md`, updating `state.json`, and regenerating `STATUS.md`.
4. Verify by running the gates declared in `state.verification_gates`.

Failures return to implementation; successes advance to audit.

---

## Step 0 — Bootstrap, Resume, or Research (every session)

No non-trivial task proceeds past Step 0 without durable intent on disk: either existing task state, an imported PRD, or a research dossier that is handed to the PRD agent.

### Step 0a — Resume (state exists)

1. List `.dev/tasks/`. If non-empty, read each `state.json`.
2. If the prompt mentions a slug, says "continue/resume", or names a feature matching an existing slug, load that state.
3. Run PRD sync via [`task-state`](../task-state/SKILL.md) §8.2.
4. Re-establish context from `prd.md`, `state.json`, `plan.md`, and the tail of `journal.md`. Do not re-ask questions already answered on disk.
5. Resume from `state.phase`:

| Phase on disk | Resume at |
|---|---|
| `prd-intake` | finish Step 0b |
| `understand` | Step 1 |
| `planning` / `plan-review` | Step 2 / Step 3 |
| `plan-approved` / `implementing` / `review-loop` | Step 4 |
| `code-review` | Step 5 |
| `audit` | Step 5.5 |
| `done` | confirm with the user before reopening; usually start a new task |

### Step 0b — PRD intake (new task with a PRD or substantive brief)

If no matching state exists and the user supplied a PRD path, URL, or substantive brief:

1. Generate a slug using [`task-state`](../task-state/SKILL.md) §1.
2. Import the PRD into `.dev/tasks/<slug>/prd.md` with source metadata and hash.
3. Set `state.phase = "prd-intake"` during copy, then `state.phase = "understand"` once the internal PRD exists.
4. Append a `prd-intake` journal entry and regenerate `STATUS.md`.
5. Continue to Step 1.

From this point onward, all skills read the internal `prd.md`, never the original path.

### Step 0c — Optional research / brainstorm front-end (raw idea)

If the user has a raw idea, market question, or "help me think this through" request instead of a PRD:

1. Invoke [`deep-research`](../deep-research/SKILL.md) or hand off to [`research`](../../agents/research.agent.md).
2. Produce a cited dossier at `docs/research/<slug>.md` covering prior art, options, trade-offs, scaling considerations, risks, recommendation, and sources.
3. Hand the dossier path to [`prd`](../../agents/prd.agent.md) as grounding context.
4. Once the PRD exists, return to Step 0b and import it into `.dev/tasks/<slug>/prd.md`.

Research is optional. Do not force it for clear bugfixes, maintenance tasks, or supplied PRDs.

---

## Step 1 — Understand

Read `.dev/tasks/<slug>/prd.md`, [`PROJECT.md`](../../../PROJECT.md), and [`ARCHITECTURE.md`](../../../ARCHITECTURE.md). Ask concise clarifying questions only when acceptance criteria, scope, data shape, target platforms, or constraints remain ambiguous.

If clarifications materially change intent, append them to `prd.md` under `## Clarifications` and set `state.prd.source_hash = null` if the upstream source was not edited.

**Exit condition:** you can state what is being built, what is out of scope, and what "done" means. `state.phase` is `understand`.

---

## Step 1.5 — Classify

Invoke [`complexity-classify`](../complexity-classify/SKILL.md). It applies deterministic stack-neutral heuristics, writes `state.complexity`, appends a journal entry, and regenerates `STATUS.md`.

- `trivial` skips Steps 2–3 and goes directly to Step 4 with single-pass review.
- All other labels continue to Step 2.

---

## Step 2 — Analyze & Plan

Invoke [`plan-task`](../plan-task/SKILL.md).

Read [`PROJECT.md`](../../../PROJECT.md), [`ARCHITECTURE.md`](../../../ARCHITECTURE.md), the active stack profile, relevant source files, current tests, and relevant docs. Produce the 11-section plan, persist it to `.dev/tasks/<slug>/plan.md`, and copy §11 gates into `state.verification_gates`.

**Exit condition:** plan exists on disk; `state.phase = "plan-review"`.

---

## Step 3 — Review the Plan

Invoke [`review-changes`](../review-changes/SKILL.md) in **plan-review mode**.

The reviewer validates the plan against `PROJECT.md`, `ARCHITECTURE.md`, the active stack profile, and [`.github/copilot-instructions.md`](../../copilot-instructions.md). Revise until verdict is `APPROVED`.

**Exit condition:** `state.phase = "plan-approved"`. No code has been written.

---

## Step 4 — Implement (Reason → Act)

Write code according to the approved plan.

- Touch only files listed in the plan, except task-state files.
- Use the package locations and commands declared in `PROJECT.md`.
- Follow the active stack profile for platform, accessibility, dependency, and testing rules.
- For `complex` tasks, checkpoint state after each file edit. For `simple`/`standard`, checkpoint after each logical phase.
- If reality invalidates the plan, stop, return to Step 2, replan, and obtain Step 3 approval again.

**Exit condition:** planned edits are complete and `state.phase = "code-review"`.

---

## Step 5 — Review the Implementation (Reflect → Verify)

Invoke [`review-changes`](../review-changes/SKILL.md) in **code-review mode**.

It runs every gate in `state.verification_gates`, captures evidence, and performs blind persona review: stack-profile/platform, security, and test-quality.

- **CHANGES REQUESTED** → set `state.phase = "review-loop"`, increment `state.attempt`, return to Step 4.
- **APPROVED** → set `state.phase = "audit"`, proceed to Step 5.5.

---

## Step 5.5 — Honesty audit

For every task whose complexity is not `trivial`, invoke [`honesty-audit`](../honesty-audit/SKILL.md).

It does not rerun tests. It verifies that claims match captured command output, planned files/docs are present, findings were closed honestly, and no unplanned scope or silent rot was introduced.

- **PASS** → set `state.phase = "done"`.
- **FAIL** → set `state.phase = "review-loop"`, append blockers, return to Step 4.

`trivial` tasks skip this step.

---

## Step 6 — Hand-off

When `state.phase = "done"`:

1. Summarize the slug, changed files, verification results, and deferred follow-ups.
2. Do not delete `.dev/tasks/<slug>/`; a human may remove it after merge.

---

## When to skip steps

| Task type | Steps required |
|---|---|
| New feature, public API/route, data-model migration, new UI screen, new service/package | 0 → 1 → 1.5 → 2 → 3 → 4 → 5 → 5.5 → 6 |
| Refactor across multiple areas or files | All |
| Cross-platform UI change under a profile that supports multiple platforms | All |
| Single-line behavior fix | 0 → 1 → 1.5 → 2 → 3 → 4 → 5 → 5.5 → 6 unless classified `trivial` |
| Typo/comment-only edit | 0 → 1 → 1.5 (`trivial`) → 4 → 5 → 6 |
| Pure Q&A/read-only investigation | No workflow required |

When in doubt, run the workflow. The cost of a small plan is lower than the cost of an unverified change.
