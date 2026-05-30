---
name: complexity-classify
description: 'Step 1.5 of the generic multi-agent workflow. Classifies a task as `trivial`, `simple`, `standard`, or `complex` using stack-neutral heuristics so planning, checkpointing, review, and audit depth scale to risk. USE FOR: deciding whether to invoke plan-task, how intensely Step 4 checkpoints, whether blind multi-persona review and honesty audit are mandatory, and writing `state.complexity`. DO NOT USE FOR: estimating calendar time or judging product value. OUTPUT: one label, a short rationale in `journal.md`, and updated task state.'
---

# SKILL: complexity-classify

Right-size the workflow. Invoke at the end of Step 1, before [`plan-task`](../plan-task/SKILL.md).

---

## Heuristics (deterministic)

Walk the table **top-down**. The first matching row wins. If uncertain, choose the higher complexity.

| Label | Trigger (any one) |
|---|---|
| **complex** | New service or top-level package; OR 10 or more files touched; OR data-model migration with backfill; OR new runtime dependency; OR cross-cutting change spanning two or more major areas; OR high-risk security/privacy impact; OR multi-step rollout. |
| **standard** | New public API/route/CLI command; OR data model/schema change; OR new background job/workflow; OR new UI screen/route with its own state; OR cross-platform UI change under a multi-platform profile; OR refactor across three or more files; OR integration with an external system. |
| **simple** | Single-package behavior change with at least one new or updated test; OR bugfix requiring two files; OR utility/hook/helper addition with tests; OR docs plus small code change. |
| **trivial** | Typo/comment/dead-import removal; OR rename with no public contract change; OR markdown-only docs change; OR test-only maintenance that does not change product behavior. |

Ambiguity must err toward more process. User phrasing such as "quick" or "small" does not override the table.

---

## Downstream meaning

| Label | Step 2 plan | Step 3 review | Step 4 checkpointing | Step 5 review | Step 5.5 audit |
|---|---|---|---|---|---|
| trivial | skip | skip | direct edit | single stack-profile/platform pass | skip |
| simple | full plan | full review | per logical phase | 3 blind personas | required |
| standard | full plan | full review | per logical phase | 3 blind personas | required |
| complex | full plan | full review | per file or subtask | 3 blind personas | required |

There is no kill-switch for review/audit on non-trivial work.

---

## Procedure

1. Read the agreed problem statement from `prd.md` and current `state.json`.
2. Check [`PROJECT.md`](../../../PROJECT.md) for package layout, target platforms, and dependency policy when relevant.
3. Walk the heuristic table and pick the label.
4. Update `state.json` with `complexity` and `updated_at`.
5. Append:

```markdown
## <iso-now> — complexity-classify
**Label:** <label>
**Why:** <one-sentence rationale naming the matched heuristic>
```

6. Regenerate `STATUS.md`.
7. Return control to [`development-workflow`](../development-workflow/SKILL.md).

---

## Edge cases

- **User disagrees:** record the deterministic heuristic and proceed with the higher-safety label.
- **Mid-task escalation:** if implementation reveals broader scope, raise complexity, append a journal entry, and return to planning. Do not silently keep a lower label.
- **Mid-task de-escalation:** do not lower complexity. Extra review is acceptable; missing review is not.
