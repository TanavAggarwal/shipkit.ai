---
name: task-state
description: 'State and resumability layer for the generic multi-agent development workflow. Manages `.dev/tasks/<slug>/` with an internal PRD copy, machine-readable state, persisted plan, human status, and append-only journal so work can resume across sessions and agent handoffs. Stack-specific package paths and commands are recorded from PROJECT.md, not hardcoded here. USE FOR: PRD intake/sync, phase transitions, complexity and verification persistence, review evidence, journal entries, and marking a task done only after verification and audit pass. DO NOT USE FOR: user-level memory, cross-task patterns, or application runtime state. OUTPUT: deterministic task workspace: `prd.md`, `state.json`, `plan.md`, `STATUS.md`, and `journal.md`.'
---

# SKILL: task-state

Persistent per-task workspace for the agent framework. This is what makes the workflow resumable rather than chat-scoped.

This skill defines:

1. Where task state lives.
2. File schemas.
3. Atomic-write rules.
4. How workflow skills read and update state.
5. PRD intake and sync.

---

## 1. Location

Every non-trivial task gets a local, git-ignored directory:

```text
.dev/tasks/<slug>/
├── prd.md          # internal copy of the supplied PRD or brief; source of truth for intent
├── state.json      # machine-readable status and gates
├── plan.md         # artifact produced by plan-task
├── STATUS.md       # one-screen human status, regenerated every step
└── journal.md      # append-only Reflect log
```

`.dev/` is local developer state and should not be committed.

The internal `prd.md` is the source of truth for the duration of the task. The user may provide a file, URL, or inline brief; copy it in and reference the copy from then on.

### Slug rules

- 2–6 words, kebab-case.
- Prefix with a stable generic scope: `feat-`, `fix-`, `infra-`, `docs-`, or `refactor-`.
- Examples: `feat-add-invite-flow`, `fix-token-refresh-race`, `infra-update-ci-gates`, `docs-api-contract`, `refactor-config-loader`.
- Generate at the end of Step 0/1 and reuse an existing slug when the prompt is clearly a continuation.

---

## 2. File schemas

### 2.1 `state.json`

```jsonc
{
  "id": "feat-add-invite-flow",
  "title": "Add team invitation flow",
  "prd": {
    "source_path": "docs/prds/invites.md",
    "source_hash": "sha256:7c1f...",
    "imported_at": "2026-05-10T13:00:00Z"
  },
  "complexity": "standard",
  "phase": "plan-approved",
  "attempt": 1,
  "plan_path": ".dev/tasks/feat-add-invite-flow/plan.md",
  "verification_gates": [
    { "name": "lint", "cmd": "<lint cmd from PROJECT.md>", "cwd": "<package cwd>", "expect": "exit 0", "source": "baseline" },
    { "name": "unit", "cmd": "<unit test cmd from PROJECT.md>", "cwd": "<package cwd>", "expect": "exit 0", "source": "baseline" }
  ],
  "last_verification": {
    "ran_at": "2026-05-10T14:32:11Z",
    "results": [
      { "name": "lint", "cmd": "<lint cmd from PROJECT.md>", "cwd": "<package cwd>", "exit": 0, "tail": "", "source": "baseline" }
    ],
    "all_passed": true,
    "baseline_passed": true
  },
  "review_personas": {
    "stack_profile": "APPROVED",
    "security": "APPROVED",
    "test_quality": "APPROVED"
  },
  "honesty_audit": "PASS",
  "blockers": [],
  "created_at": "2026-05-10T13:00:00Z",
  "updated_at": "2026-05-10T14:32:11Z"
}
```

#### Phase enum

`prd-intake → understand → planning → plan-review → plan-approved → implementing → code-review → review-loop → audit → done`

- `prd-intake` is set while the PRD is being copied.
- `review-loop` means a Step 4 ↔ Step 5 iteration is in progress.
- `done` is only set when verification passed, honesty audit passed when required, and `blockers` is empty.

### 2.2 `prd.md`

Internal copy of the user-supplied PRD, research-grounded PRD, or inline brief. All subsequent skills read this file, not the original source.

If the user supplied only inline text, create a PRD with a synthesized heading and the literal prompt body.

### 2.3 `plan.md`

The exact markdown output of [`plan-task`](../plan-task/SKILL.md). Written at the end of Step 2 and updated only during replan.

### 2.4 `STATUS.md`

Regenerated after every state mutation:

```markdown
# <slug> — <title>

**Phase:** <phase>     **Complexity:** <complexity>     **Attempt:** <n>

## What's done
- <bullet>

## What's next
- <bullet>

## Last verification (<ran_at>)
- `<cmd>` → <exit 0/1/skipped>

## Personas
- stack-profile/platform: <APPROVED|CHANGES_REQUESTED|NA>
- security:               <APPROVED|CHANGES_REQUESTED>
- test-quality:           <APPROVED|CHANGES_REQUESTED>

## Blockers
- <bullet, or "(none)">
```

### 2.5 `journal.md`

Append-only Reflect log:

```markdown
## 2026-05-10T14:32:11Z — review-changes (code-review) — attempt 1
**Verdict:** CHANGES_REQUESTED
- blocker: missing validation for empty invite email
- blocker: e2e gate failed before reaching assertion
**Next:** Step 4 — fix and rerun.
```

---

## 3. Atomic-write rules

Agents can be interrupted mid-write. Always:

1. Write to a sibling temporary file such as `state.json.tmp`.
2. Rename over the target (`Move-Item -Force` on Windows; `mv` on POSIX).
3. Append to `journal.md`; never rewrite the middle of it.
4. Update `state.updated_at` in the same write as any other state change.

If `state.json` is missing or unparseable, do not guess. Halt and ask for human recovery.

---

## 4. How other skills use this

| Skill | Reads | Writes |
|---|---|---|
| [`development-workflow`](../development-workflow/SKILL.md) | `state.json`, `prd.md`, `plan.md`, `journal.md` | phase transitions |
| [`complexity-classify`](../complexity-classify/SKILL.md) | `prd.md`, `state.json` | `state.complexity`, journal |
| [`plan-task`](../plan-task/SKILL.md) | `PROJECT.md`, `ARCHITECTURE.md`, `state.json`, `prd.md` | `plan.md`, `state.verification_gates`, `state.phase` |
| [`review-changes`](../review-changes/SKILL.md) | `state.json`, `plan.md`, diff | `state.review_personas`, `state.last_verification`, `state.phase`, journal |
| [`honesty-audit`](../honesty-audit/SKILL.md) | `state.last_verification`, `journal.md`, diff | `state.honesty_audit`, `state.phase`, blockers |

After every write, regenerate `STATUS.md`.

---

## 5. Resume protocol

At the start of every development session:

1. List `.dev/tasks/`.
2. If the prompt mentions a slug, "continue", "resume", or a feature matching an existing slug, read that task's `state.json` first.
3. Re-establish context from `state.json`, `prd.md`, `plan.md`, and the tail of `journal.md`.
4. Resume at `state.phase` according to [`development-workflow`](../development-workflow/SKILL.md).

If no matching state exists, bootstrap a new task from PRD intake.

---

## 6. Bootstrap

```text
1. Generate slug.
2. Create .dev/tasks/<slug>/.
3. Import PRD or inline brief.
4. Write initial state.json:
   phase=understand, complexity=PENDING, attempt=0, blockers=[],
   prd={ source_path, source_hash, imported_at },
   verification_gates=[], last_verification=null,
   created_at=now, updated_at=now,
   title=<one-line summary>.
5. Write journal.md with an intake entry.
6. Regenerate STATUS.md.
7. Return to development-workflow Step 1.
```

---

## 7. Cleanup

When `state.phase == "done"` and the change has merged, a human may delete `.dev/tasks/<slug>/`. Agents must not auto-delete task workspaces.

---

## 8. PRD intake and sync

### 8.1 Intake

| Input | Action |
|---|---|
| Path to markdown/text | Read it, copy verbatim to `.dev/tasks/<slug>/prd.md`, record `source_path`, `source_hash`. |
| URL to a document or raw markdown | Fetch contents, save verbatim, record URL and hash. |
| Inline brief | Wrap with `# <auto-title>` and save literal body; `source_path = null`, `source_hash = null`. |
| Research dossier | Treat the dossier as grounding context for the PRD agent; import the resulting PRD, not the dossier, as task intent. |
| Another task's PRD | Copy it; do not link across task directories. |

The internal copy is literal. Do not summarize or reformat at intake.

Append:

```markdown
## <iso-now> — prd-intake
**Source:** <source_path or "inline">
**Hash:** <source_hash or "n/a">
**Title:** <derived title>
```

### 8.2 Sync on resume

If `state.prd.source_path` is set:

1. Re-read the source.
2. Compute the current hash.
3. If unchanged, continue.
4. If changed, overwrite internal `prd.md`, update hash and `imported_at`, append a `prd-sync` journal entry, and add a blocker if the task is past `plan-approved` so the plan is revalidated.

If `source_path` is null, no upstream sync is possible.

### 8.3 User-driven amendments

If the user changes intent mid-task, either sync from the original source or edit internal `prd.md`, set `source_hash = null`, and append a `prd-amend` journal entry.

After any PRD change:

- If phase is `plan-approved`, `implementing`, `review-loop`, or `code-review`, add blocker `PRD changed mid-task; re-validate plan` and return to Step 3 before implementation continues.
- If phase is earlier, continue normally.
