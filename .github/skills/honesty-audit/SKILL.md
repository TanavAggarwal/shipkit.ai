---
name: honesty-audit
description: 'Step 5.5 of the generic multi-agent workflow. Final anti-sycophancy gate after review-changes approves code. Verifies that claimed verification actually ran, findings were honestly closed, planned files/docs are present, no unplanned scope or dependencies slipped in, and no silent rot remains. Stack-specific commands are taken from task state and PROJECT.md, not hardcoded. USE FOR: every simple, standard, or complex task before marking done. DO NOT USE FOR: replacing code review or judging style. OUTPUT: `state.honesty_audit = PASS | FAIL`, journal entry, and either `phase = done` or return to review-loop with blockers.'
---

# SKILL: honesty-audit

Final gate before a task can be marked `done`. It catches the failure mode where an agent says "this should pass" instead of proving "this did pass".

Invoke after [`review-changes`](../review-changes/SKILL.md) returns `APPROVED` in code-review mode for any task whose `state.complexity != "trivial"`.

---

## Evidence checklist

Every item is yes/no. Any **no** means `FAIL`.

### Verification gates ran

- [ ] Every command in `state.verification_gates` has a matching entry in `state.last_verification.results`.
- [ ] Every executed result has a real process exit code, not an assumption or prose claim.
- [ ] Every approved skip has the same skip reason documented in the approved plan.
- [ ] Every non-zero exit was fixed or recorded as a current blocker.
- [ ] `state.last_verification.ran_at` belongs to the current attempt, not a stale prior run.

### Review findings closed

- [ ] Every blocker in prior `journal.md` review entries was fixed with file evidence or explicitly retracted with reasoning.
- [ ] No blocker was dismissed as "minor", "out of scope", or "follow-up" unless `state.blockers` records the deferral and owner/tracking note.

### Plan adherence

- [ ] Every file in the approved plan's file list appears in the diff or is explicitly marked not needed in an approved replan.
- [ ] Every product file in the diff is listed in the approved plan.
- [ ] No new runtime dependency was added unless the approved plan listed it and `PROJECT.md` dependency policy allows the path taken.
- [ ] Dev/test tooling additions are consistent with `PROJECT.md` and show expected lockfile/config changes.

### No silent rot

- [ ] No unowned `TODO`, `FIXME`, or `XXX` was introduced.
- [ ] No commented-out code was introduced.
- [ ] No stray debug prints or ad-hoc diagnostics remain in production paths.
- [ ] No language-specific lint/type/error suppression appears without inline justification.
- [ ] No secrets, credentials, tokens, or local-only paths were committed.

### Documentation matches reality

- [ ] Promised updates to `ARCHITECTURE.md`, package/feature docs, public interface docs, or config examples are in the diff.
- [ ] Documentation claims match actual behavior and verified commands.
- [ ] Any known gaps are documented as blockers or explicit follow-ups.

### Honesty about coverage and quality

- [ ] Coverage, performance, accessibility, security, or compatibility claims are backed by captured tool output or documented manual evidence.
- [ ] If a threshold was missed, `state.blockers` reflects it.
- [ ] The final user-facing summary does not overstate what was tested.

---

## Procedure

1. Read `state.json`, `plan.md`, full `journal.md`, and `git diff --name-only` against the merge base.
2. Walk the checklist top-down, collecting all gaps.
3. Write a journal entry:

```markdown
## <iso-now> — honesty-audit
**Verdict:** PASS | FAIL
**Gaps:**
- <category> — <specific evidence missing> — <required fix>
```

4. Update `state.json`:
   - PASS: `honesty_audit = "PASS"`, `phase = "done"`.
   - FAIL: `honesty_audit = "FAIL"`, `phase = "review-loop"`, `attempt += 1`, append gaps to `blockers`.
5. Regenerate `STATUS.md`.
6. Return control to [`development-workflow`](../development-workflow/SKILL.md).

---

## What this skill does not do

- It does not rerun verification commands; code-review mode owns execution.
- It does not edit code or docs.
- It does not add stylistic preferences. It audits evidence, scope, and truthfulness.

---

## Failure mode this catches

> The task state says all gates passed, but `last_verification` contains no exit code or output for the project's test command.

That is a FAIL. The workflow reopens until real command evidence is captured.
