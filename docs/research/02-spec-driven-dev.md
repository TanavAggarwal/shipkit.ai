# Spec-driven development, skills, and file-defined agents: dense research notes

Focus: how leading systems turn ideas into durable specs/plans/tasks, and how reusable skills/agents are packaged as files for a generic starter template.

## The recurring pattern

`idea -> clarify -> spec/requirements -> design/plan -> tasks -> implementation -> verification -> review/audit -> ship`

The best systems make intermediate artifacts executable or at least enforceable. They do not let the model jump from vague idea to code without a written contract.

## Systems studied

### GitHub Spec Kit
- Source: [github/spec-kit](https://github.com/github/spec-kit).
- Purpose: open-source toolkit for Spec-Driven Development; explicitly says specs become executable and shift focus from vibe coding to predictable outcomes.
- Workflow slash commands:
  1. `/speckit.constitution` — project principles/governance.
  2. `/speckit.specify` — what/why: requirements and user stories, not tech stack.
  3. `/speckit.plan` — technical implementation plan and architecture choices.
  4. `/speckit.tasks` — actionable task list.
  5. `/speckit.implement` — execute tasks.
  6. Optional `/speckit.clarify`, `/speckit.analyze`, `/speckit.checklist`, `/speckit.taskstoissues`.
- Customization model: templates, project-local overrides, presets, extensions; supports 30+ coding-agent integrations; can install as slash commands or skills.
- Reusable idea: treat workflow commands as prompt files/templates, not product-specific code. Add `constitution` early to constrain every later artifact.

### Amazon Kiro spec mode
- Sources: [Kiro Specs](https://kiro.dev/docs/specs/), [Feature Specs](https://kiro.dev/docs/specs/feature-specs/), [Requirements-first](https://kiro.dev/docs/specs/feature-specs/requirements-first/), [Quick Plan](https://kiro.dev/docs/specs/quick-plan/), [Analyze Requirements](https://kiro.dev/docs/specs/analyze-requirements/).
- Purpose: IDE workflow where specs are the unit of work.
- Core files under `.kiro/specs/<feature>/`:
  - `requirements.md` — user stories, acceptance criteria, often EARS-style `WHEN ... THE SYSTEM SHALL ...` statements.
  - `design.md` — architecture, components, data models, sequence diagrams, error handling, testing strategy.
  - `tasks.md` — concrete implementation steps, dependencies, status.
- Workflows: requirements-first, design-first, bugfix specs, quick-plan one-shot generation.
- Adjacent systems: steering files provide persistent project context; hooks automate checks/actions on events.
- Reusable idea: one feature = one directory with requirements/design/tasks. Make dependencies explicit so non-conflicting tasks can run in waves.

### Anthropic Claude Agent Skills / Claude Code Skills
- Sources: [Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code/skills), [Agent Skills standard](https://agentskills.io).
- Purpose: reusable capabilities loaded only when relevant; avoid putting every procedure in global instructions.
- File shape:
  - Directory: `.claude/skills/<skill-name>/SKILL.md` (project) or `~/.claude/skills/<skill-name>/SKILL.md` (personal).
  - Optional supporting files: templates, examples, scripts, detailed references.
  - YAML frontmatter can define `name`, `description`, `when_to_use`, `argument-hint`, `allowed-tools`, `disallowed-tools`, `model`, `effort`, invocation controls, etc.
- Progressive disclosure: keep `SKILL.md` concise; reference supporting files only when needed. Once loaded, skill content stays in context, so every line has token cost.
- Skill types:
  - Reference content: conventions/domain knowledge applied inline.
  - Task content: step-by-step workflows, often manually invoked and sometimes run in fork/subagent context.
- Reusable idea: package each lifecycle step as a skill with short trigger metadata + precise output contract + optional scripts.

### Claude Code subagents
- Source: [Claude Code Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents).
- Purpose: specialized assistants with their own context, prompt, tools, permissions, model, and optional skills; preserve main context and enforce constraints.
- File shape:
  - Project: `.claude/agents/<agent>.md`.
  - User: `~/.claude/agents/<agent>.md`.
  - Markdown file with YAML frontmatter plus body prompt.
- Example fields: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `hooks`, `maxTurns`, `skills`, `isolation`, `background`.
- Built-ins demonstrate phase boundaries: Explore/Plan are read-only; general-purpose can act.
- Reusable idea: define `researcher`, `specifier`, `planner`, `implementer`, `reviewer`, `auditor`, `shipper` as files. Tool access is part of the role definition.

### Claude Code hooks
- Source: [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks).
- Purpose: shell commands, HTTP endpoints, or LLM prompts triggered at lifecycle events.
- Events include `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PostToolBatch`, `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `FileChanged`, `Stop`, etc.
- Hooks can block decisions, e.g. deny destructive shell commands before execution.
- Reusable idea: enforce framework invariants outside prompts: block implementation before approved plan; require tests before done; prevent reviewer edits.

### BMAD-METHOD
- Sources: [BMAD-METHOD repo](https://github.com/bmad-code-org/BMAD-METHOD), [BMAD docs index](https://docs.bmad-method.org/llms.txt).
- Purpose: AI-driven agile development with scale-adaptive workflows, specialized agents, and structured phases from brainstorming to deployment.
- Key ideas: 12+ domain agents (PM, Architect, Developer, UX, etc.), workflows for analysis/planning/architecture/implementation, Party Mode for multi-agent discussion, web bundles for upfront planning in chat tools, installable modules.
- Reusable idea: complexity-adaptive planning. Small bug fixes should not pay enterprise-planning token cost; large systems need product/architecture/test roles.

### Cline rules, skills, Plan/Act, memory bank
- Sources: [Cline rules](https://docs.cline.bot/features/cline-rules), [Cline docs index](https://docs.cline.bot/llms.txt).
- Purpose: IDE/CLI agent with persistent instructions, plan/act split, checkpoints, skills, subagents, hooks, MCP, GitHub workflows.
- Rules are markdown files in `.clinerules/`; Cline also detects `.cursorrules`, `.windsurfrules`, and `AGENTS.md` for cross-tool compatibility.
- Conditional rules use YAML frontmatter paths to load only when relevant.
- Reusable idea: support `AGENTS.md` and conditional rule files as a cross-tool bridge; plan mode should be read-only until approved.

### Roo Code custom modes
- Source: [Roo custom modes](https://docs.roocode.com/features/custom-modes).
- Purpose: define specialized modes with role, tools, file permissions, custom instructions, and `whenToUse` metadata.
- File/config concepts: `slug`, `name`, `description`, `roleDefinition`, `groups` (toolsets/file access), `whenToUse`, `customInstructions`; modes import/export as YAML.
- Reusable idea: publish agents/modes as portable YAML/markdown with explicit permissions. A reviewer mode can be read-only by construction.

### Cursor rules/agents
- Source: [Cursor docs](https://docs.cursor.com/) metadata describes Agent, Rules, MCP, Skills, CLI.
- Purpose: agentic IDE where rules provide persistent project instructions and agent mode performs multi-file edits/commands.
- Reusable idea: many developers already understand `.cursor/rules`/project rules; provide adapter files generated from the canonical framework instructions.

### Replit Agent skills and Plan mode
- Sources: [Replit Agent](https://docs.replit.com/replitai/agent), [Replit docs index](https://docs.replit.com/llms.txt).
- Purpose: hosted idea-to-app agent; Plan mode brainstorms and creates ordered tasks before code; Agent tests work and creates checkpoints.
- Reusable idea: approval gates should be UX-visible: user sees task list, approves, then build begins. Checkpoints are part of the workflow, not an afterthought.

### OpenHands skills/file agents
- Sources: [OpenHands docs](https://docs.openhands.dev/), [OpenHands llms index](https://docs.openhands.dev/llms.txt).
- Purpose: composable software-agent SDK with skills, file-based agents, hooks, event logs, sandboxes, persistence, sub-agent delegation, critic/iterative refinement.
- Reusable idea: include SDK-oriented adapters for people who want to run the framework outside a specific IDE. File-based agents keep no-code customization possible.

## Spec -> plan -> build -> verify as an artifact schema

Minimum generic feature directory:

```text
.agent/tasks/<slug>/
  brief.md          # imported user idea/PRD; source of truth for intent
  spec.md           # user value, requirements, acceptance criteria, non-goals
  plan.md           # architecture/approach, files, risks, verification gates
  tasks.md          # ordered checklist with dependencies/status
  verify.md         # commands run, outputs summarized, manual checks, gaps
  review.md         # reviewer findings + disposition
  state.json        # machine-readable phase/status/resume metadata
```

For publishability, keep `.agent/tasks/` configurable; many projects may prefer `.dev/tasks/`, `.ai/tasks/`, or `.kiro/specs/`.

## File-defined skills/agents: recommended generic format

### Skill file

```md
---
name: plan-task
description: Turn an approved spec into an implementation plan with verification gates.
when_to_use: Use after spec.md is accepted and before any code edits.
allowed-tools: Read Grep Glob
---
Inputs: task directory containing spec.md and state.json.
Outputs: plan.md with sections: summary, assumptions, design, files, risks, verification gates, rollback.
Rules: do not edit product code. If requirements are ambiguous, record assumptions or create clarify questions.
```

### Agent file

```md
---
name: dev-review
description: Read-only reviewer that verifies changes against plan and tests.
tools: Read, Grep, Glob, Bash
model: sonnet
permissionMode: read-only
---
Review only. Run declared verification gates. Report blockers with file paths, evidence, and exact remediation. Do not edit files.
```

### Hook policy examples

- `PreToolUse(Edit|Write)`: deny unless `state.phase == implement` and actor is implementer.
- `TaskCompleted`: require `verify.md` updated for implementation tasks.
- `Stop`: warn if there are modified files but no review/audit artifact.
- `PostToolUse(Bash)`: capture test/build command summary into verification evidence.

## Lessons for our framework

1. **Adopt a canonical artifact chain.** Use `brief/spec/plan/tasks/verify/review/state` as the portable core; generate tool-specific adapters from it.
2. **Make specs testable.** Requirements should include acceptance criteria and edge cases; prefer EARS-style statements where helpful.
3. **Put governance first.** A `constitution` or `project-principles.md` should define quality/security/testing expectations before specs are written.
4. **Use progressive disclosure.** Keep top-level agent instructions tiny; move procedures into skills loaded only for that phase.
5. **Define agents as files with permissions.** Role, trigger, tools, model, inputs, outputs, and stop criteria belong in frontmatter/body; no hidden orchestration magic.
6. **Plan mode must be non-mutating.** Research/spec/planning agents are read-only; code edits start only after plan approval.
7. **Verification gates belong in the plan.** Every task plan must name commands/checks and expected evidence; implementation is incomplete without `verify.md`.
8. **Review is a separate actor.** Reviewer/auditor cannot edit. Findings go back to implementer; approval requires evidence, not vibes.
9. **Support hooks but do not require one vendor.** Provide optional Claude/Cline/Roo/Cursor/GitHub adapters plus generic shell scripts/CI checks.
10. **Scale ceremony by complexity.** Simple bug: brief + tiny plan + tests. Complex product: full spec, design, tasks, independent review, audit, release notes.
11. **Make resume deterministic.** `state.json` should identify phase, approved artifacts, pending tasks, verification gates, and next allowed agents.
12. **Design for publication.** Avoid project-specific names, stacks, and policies in the core; ship examples as presets/extensions.
